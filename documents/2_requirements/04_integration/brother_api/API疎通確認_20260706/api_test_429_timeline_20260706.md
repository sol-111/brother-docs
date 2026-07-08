# 生成API 429（クォータ制限）検証まとめ

- 作成: 株式会社アイビス
- 検証日: 2026-07-06 / 2026-07-08（時刻はすべてJST）
- 対象環境: brother-backend-dev-yokinini
- 対象事象: 生成系APIで `AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota).` によりジョブがFAILEDとなる事象（API疎通確認 確認事項A）
- 関連: `api_test_verification_20260706.md`

---

## 1. 結論（要約）

- dev環境の生成APIは、実測上**約1分に1件**しか生成が成功しない（それを超えるペースの投入は429でFAILEDになる）
- 投入間隔の調整（10秒間隔、完全直列）を試したが、**クライアント側の工夫だけでは回避できない**
- 429で失敗しても、30秒〜1分後の再投入で成功する（一時的なクォータ枯渇であり恒久的な失敗ではない）
- 本番で複数ユーザーが同時利用する前提では成立しないため、**バックエンド側での対応をご相談したい**（下記2）

## 2. 対応のご提案・ご相談

| # | 対応 | 実施者 | 位置づけ |
|---|---|---|---|
| 1 | AIプロバイダ側クォータの引き上げ | ブラザー側 | 根本対応。まずdev環境の現行クォータ値をご教示いただきたい |
| 2 | バックエンドでの429時自動リトライ or キューイング | ブラザー側 | 本命。AIプロバイダから429が返った際、即FAILED確定せず待機・再試行（またはジョブをキューで整列）していただければ、クライアントからは処理時間が延びるだけで失敗が見えなくなる |
| 3 | Bubble側での429限定自動リトライ | アイビス側 | 保険として実装予定。`errorCode: AI_GENERATION_FAILED` かつ `errorMessage` に `:429:` を含むFAILEDのみ、30〜60秒後に自動再投入（最大2〜3回）。※リトライごとに新しいgeneratedImageIdが発行されるため、ポーリング対象を差し替える実装とする |

※2が入るまでの間、複数サイズの一括生成はBubble側で直列化しても約半数が429になるため（下記3-2）、dev環境での画面検証に支障が出ています。1のクォータ引き上げを優先いただけると助かります。

## 3. 実測結果

### 3-1. 発生状況（7/6 疎通確認時）

- 全17ジョブ中**5件**が429でFAILED（長尺横 初回、ハガキ 初回・再投入、長尺縦再生成 1回目・2回目）
- 画像生成対象の全6サイズ（A4 / A3 / 長尺縦 / 長尺横 / ハガキ / 封筒(長形3号)。A4便箋は仕様上画像生成対象外）を約1分以内に連続投入した状態で発生
- いずれも同一リクエストの再投入で最終的に成功

429で最も再投入を要した「長尺縦の再生成」のタイムライン:

| 試行 | 投入時刻 | generatedImageId | 結果 | 結果確認時刻 | 所要 |
|---|---|---|---|---|---|
| 1回目 | 11:09:32 | gen-7fbd32d4-36f8-4a0e-b71f-41d6e833af68 | FAILED（429） | 11:10:50（※） | 投入から約40秒〜1分でFAILED |
| 2回目 | 11:12:25 | gen-445f1a30-f730-40f4-8f1a-0cf7066cf99f | FAILED（429） | 11:13:08 | 約43秒でFAILED |
| 3回目 | 11:14:51 | gen-add6a20f-8e00-4fd9-9869-634d60aacb67 | **COMPLETED** | 11:15:11 | 約20秒で完了 |

- ※1回目の結果確認時刻はポーリングループ終了時のファイル書き込み時刻。ポーリング記録上は投入から約40秒後にFAILEDへ遷移
- **初回投入から成功まで: 11:09:32 → 11:15:11 = 約5分39秒**（手動リトライ2回・リトライ間隔2〜3分を含む）

### 3-2. 投入間隔を空ければ回避できるか（7/8 追加検証）→ 回避できず

urabon1の6サイズを対象に、eventName / eventPeriod のみの最小リクエストで2パターンを実測。

**実験1: 10秒間隔で投入 → 6件中3件が429**

| 投入時刻 | サイズ | generatedImageId | 結果 |
|---|---|---|---|
| 18:48:12 | A4 | gen-87f9beb3-474c-4a36-849d-9fa2e751aafe | COMPLETED |
| 18:48:23 | A3 | gen-aa8b5545-2f78-411b-b830-483cbfe1965f | COMPLETED |
| 18:48:33 | 長尺縦 | gen-7173aa71-351c-4326-92b4-8c9b4b870268 | FAILED（429） |
| 18:48:44 | 長尺横 | gen-ff79db22-349c-41a9-8f64-9cb5b1311baf | COMPLETED |
| 18:48:54 | ハガキ | gen-1afe4152-5325-4b19-8703-ec482ab5247e | FAILED（429） |
| 18:49:07 | 封筒(長形3号) | gen-8afd56a3-29c4-4607-b180-718fa1eea785 | FAILED（429） |

**実験2: 完全直列投入（前ジョブの完了/失敗を確認してから次を投入）→ それでも6件中3件が429**

| 投入時刻 → 結果確認 | サイズ | generatedImageId | 結果 |
|---|---|---|---|
| 18:50:35 → 18:50:55 | A4 | gen-715dfe22-4899-42b9-9738-4a7a5bcef005 | COMPLETED |
| 18:50:55 → 18:51:26 | A3 | gen-88a9b7d5-0646-4517-bb9b-5eb6f745c510 | FAILED（429） |
| 18:51:26 → 18:51:47 | 長尺縦 | gen-56a3a328-9222-4b4e-92b3-2b5824313d42 | COMPLETED |
| 18:51:47 → 18:52:17 | 長尺横 | gen-c14fa89b-a0b7-4ce3-9b67-57763e4bb1b2 | FAILED（429） |
| 18:52:17 → 18:52:38 | ハガキ | gen-fdeafb82-1ed5-41cd-8de0-23982c71b938 | COMPLETED |
| 18:52:38 → 18:53:08 | 封筒(長形3号) | gen-cb788252-315b-4388-ac42-d04a8cba9552 | FAILED（429） |

- 実験2では成功と失敗が交互に発生。成功ジョブの投入時刻は 18:50:35 / 18:51:26 / 18:52:17 と**約50秒間隔**
- → **クォータは毎分1リクエスト程度**と推定。仮にクライアント側で1分/件に間引くと6サイズの生成に6分以上を要し、複数ユーザーの同時利用では成立しない

### 3-3. 挙動の特徴

- 429の場合もPOSTは正常受付（`status: RUNNING`）され、**30秒〜1分後のポーリングでFAILEDに遷移**する（POSTレスポンスでは判別できない）
- 正常時の生成所要時間はおおむね10〜60秒
- FAILEDレスポンスの形式:

```json
{
  "status": "FAILED",
  "errorCode": "AI_GENERATION_FAILED",
  "errorMessage": "AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."
}
```

---

## 4. エビデンス（APIレスポンス全文）

### 4-1. 長尺縦・再生成（7/6）

POST /generated-images/gen-feb6f18b-7d8c-481a-9d81-705dfaa72935/regenerate リクエスト（3回とも同一）:

```json
{
  "eventName": "ブラザー記念夏祭り工場見学会",
  "eventPeriod": "2026年8月23日〜24日",
  "contactName": "ブラザー工業 総務部 山田",
  "contactDetails": "TEL 052-123-4567（平日9:00〜17:00）"
}
```

1回目 POSTレスポンス（11:09:32保存）:

```json
{"generatedImageId":"gen-7fbd32d4-36f8-4a0e-b71f-41d6e833af68","parentGeneratedImageId":"gen-feb6f18b-7d8c-481a-9d81-705dfaa72935","sourceImageId":"src-f366749e-02cb-429b-bddb-e15ca25f6117","status":"RUNNING"}
```

1回目 GET最終レスポンス（11:10:50保存）:

```json
{"generatedImageId":"gen-7fbd32d4-36f8-4a0e-b71f-41d6e833af68","sourceImageId":"src-f366749e-02cb-429b-bddb-e15ca25f6117","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
```

1回目 ポーリング記録（10秒間隔）:

```
[1] 7fbd32d4:RUNNING
[2] 7fbd32d4:RUNNING
[3] 7fbd32d4:RUNNING
[4] 7fbd32d4:FAILED
[5] 7fbd32d4:FAILED
[6] 7fbd32d4:FAILED
```

2回目 POSTレスポンス（11:12:25保存）:

```json
{"generatedImageId":"gen-445f1a30-f730-40f4-8f1a-0cf7066cf99f","parentGeneratedImageId":"gen-feb6f18b-7d8c-481a-9d81-705dfaa72935","sourceImageId":"src-f366749e-02cb-429b-bddb-e15ca25f6117","status":"RUNNING"}
```

2回目 GET最終レスポンス（11:13:08保存）:

```json
{"generatedImageId":"gen-445f1a30-f730-40f4-8f1a-0cf7066cf99f","sourceImageId":"src-f366749e-02cb-429b-bddb-e15ca25f6117","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
```

2回目 ポーリング記録:

```
[1] 445f1a30:RUNNING
[2] 445f1a30:RUNNING
[3] 445f1a30:FAILED
```

3回目 POSTレスポンス（11:14:51保存）:

```json
{"generatedImageId":"gen-add6a20f-8e00-4fd9-9869-634d60aacb67","parentGeneratedImageId":"gen-feb6f18b-7d8c-481a-9d81-705dfaa72935","sourceImageId":"src-f366749e-02cb-429b-bddb-e15ca25f6117","status":"RUNNING"}
```

3回目 GET最終レスポンス（11:15:11保存、成功）:

```json
{"generatedImageId":"gen-add6a20f-8e00-4fd9-9869-634d60aacb67","sourceImageId":"src-f366749e-02cb-429b-bddb-e15ca25f6117","status":"COMPLETED","fileKey":"generated-images/bubble-test-user/gen-add6a20f-8e00-4fd9-9869-634d60aacb67/output.png","thumbnailUrl":"https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-add6a20f-8e00-4fd9-9869-634d60aacb67/thumbnail.png"}
```

3回目 ポーリング記録:

```
[1] add6a20f:COMPLETED
```

### 4-2. 7/6に429でFAILEDした他の3ジョブのGETレスポンス

初回生成 長尺横（urabon1、1回目投入分）:

```json
{"generatedImageId":"gen-1ead186a-f276-4018-a7a0-369fe145390d","sourceImageId":"src-c4144f6c-0a2c-4a94-937d-a9a175f47be1","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
```

初回生成 ハガキ（urabon1、1回目投入分）:

```json
{"generatedImageId":"gen-eb509f44-b6ed-4f91-984f-da9bdffd97c4","sourceImageId":"src-385b23c9-5b69-4bc1-9615-7964f9db4b64","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
```

初回生成 ハガキ（urabon1、2回目投入分。3回目=gen-cbbd64ccで成功）:

```json
{"generatedImageId":"gen-cafa692a-b491-4752-8de7-43afd8e70991","sourceImageId":"src-385b23c9-5b69-4bc1-9615-7964f9db4b64","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
```

### 4-3. 実験1（7/8 10秒間隔）GET最終レスポンス

```json
{"generatedImageId":"gen-87f9beb3-474c-4a36-849d-9fa2e751aafe","sourceImageId":"src-272cbd66-267f-4b5c-bd36-20747213c44f","status":"COMPLETED","fileKey":"generated-images/bubble-test-user/gen-87f9beb3-474c-4a36-849d-9fa2e751aafe/output.png","thumbnailUrl":"https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-87f9beb3-474c-4a36-849d-9fa2e751aafe/thumbnail.png"}
{"generatedImageId":"gen-aa8b5545-2f78-411b-b830-483cbfe1965f","sourceImageId":"src-125cddcf-c928-4081-89c1-5238283d28de","status":"COMPLETED","fileKey":"generated-images/bubble-test-user/gen-aa8b5545-2f78-411b-b830-483cbfe1965f/output.png","thumbnailUrl":"https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-aa8b5545-2f78-411b-b830-483cbfe1965f/thumbnail.png"}
{"generatedImageId":"gen-7173aa71-351c-4326-92b4-8c9b4b870268","sourceImageId":"src-915725fa-8f13-466f-8c3d-628def287ec5","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
{"generatedImageId":"gen-ff79db22-349c-41a9-8f64-9cb5b1311baf","sourceImageId":"src-c4144f6c-0a2c-4a94-937d-a9a175f47be1","status":"COMPLETED","fileKey":"generated-images/bubble-test-user/gen-ff79db22-349c-41a9-8f64-9cb5b1311baf/output.png","thumbnailUrl":"https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-ff79db22-349c-41a9-8f64-9cb5b1311baf/thumbnail.png"}
{"generatedImageId":"gen-1afe4152-5325-4b19-8703-ec482ab5247e","sourceImageId":"src-385b23c9-5b69-4bc1-9615-7964f9db4b64","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
{"generatedImageId":"gen-8afd56a3-29c4-4607-b180-718fa1eea785","sourceImageId":"src-cf4d527e-a233-4f51-84c7-10a223e1ce0a","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
```

### 4-4. 実験2（7/8 完全直列）GET最終レスポンス

```json
{"generatedImageId":"gen-715dfe22-4899-42b9-9738-4a7a5bcef005","sourceImageId":"src-272cbd66-267f-4b5c-bd36-20747213c44f","status":"COMPLETED","fileKey":"generated-images/bubble-test-user/gen-715dfe22-4899-42b9-9738-4a7a5bcef005/output.png","thumbnailUrl":"https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-715dfe22-4899-42b9-9738-4a7a5bcef005/thumbnail.png"}
{"generatedImageId":"gen-88a9b7d5-0646-4517-bb9b-5eb6f745c510","sourceImageId":"src-125cddcf-c928-4081-89c1-5238283d28de","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
{"generatedImageId":"gen-56a3a328-9222-4b4e-92b3-2b5824313d42","sourceImageId":"src-915725fa-8f13-466f-8c3d-628def287ec5","status":"COMPLETED","fileKey":"generated-images/bubble-test-user/gen-56a3a328-9222-4b4e-92b3-2b5824313d42/output.png","thumbnailUrl":"https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-56a3a328-9222-4b4e-92b3-2b5824313d42/thumbnail.png"}
{"generatedImageId":"gen-c14fa89b-a0b7-4ce3-9b67-57763e4bb1b2","sourceImageId":"src-c4144f6c-0a2c-4a94-937d-a9a175f47be1","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
{"generatedImageId":"gen-fdeafb82-1ed5-41cd-8de0-23982c71b938","sourceImageId":"src-385b23c9-5b69-4bc1-9615-7964f9db4b64","status":"COMPLETED","fileKey":"generated-images/bubble-test-user/gen-fdeafb82-1ed5-41cd-8de0-23982c71b938/output.png","thumbnailUrl":"https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-fdeafb82-1ed5-41cd-8de0-23982c71b938/thumbnail.png"}
{"generatedImageId":"gen-cb788252-315b-4388-ac42-d04a8cba9552","sourceImageId":"src-cf4d527e-a233-4f51-84c7-10a223e1ce0a","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
```
