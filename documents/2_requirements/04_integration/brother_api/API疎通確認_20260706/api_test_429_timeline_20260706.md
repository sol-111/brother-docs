# 429（クォータ制限）発生時のリトライ所要時間エビデンス

- 対象: 確認事項A（AI側クォータ制限 429）
- 検証日: 2026-07-06（時刻はすべてJST）
- 対象環境: brother-backend-dev-yokinini
- 記録方法: 検証時に保存したAPIレスポンスファイルの書き込み時刻（ローカルファイルのタイムスタンプ）から再構成
- 関連: `api_test_verification_20260706.md`

## 要約

429で失敗した「長尺縦の再生成」は、手動リトライ2回を挟み、**初回投入（11:09:32）から成功（11:15:11確認）まで約5分40秒**を要した。各失敗は投入後30秒〜1分でFAILEDが返却され、正常時の生成は10〜60秒程度で完了する。

## 長尺縦・再生成のタイムライン

親ジョブ: gen-feb6f18b-7d8c-481a-9d81-705dfaa72935（初回生成 長尺縦、sig3 src-f366749e-02cb-429b-bddb-e15ca25f6117）

| 試行 | 投入時刻 | generatedImageId | 結果 | 結果確認時刻 | 所要 |
|---|---|---|---|---|---|
| 1回目 | 11:09:32 | gen-7fbd32d4-36f8-4a0e-b71f-41d6e833af68 | FAILED（429） | 11:10:50（※） | 投入から約40秒〜1分でFAILED |
| 2回目 | 11:12:25 | gen-445f1a30-f730-40f4-8f1a-0cf7066cf99f | FAILED（429） | 11:13:08 | 約43秒でFAILED |
| 3回目 | 11:14:51 | gen-add6a20f-8e00-4fd9-9869-634d60aacb67 | **COMPLETED** | 11:15:11 | 約20秒で完了（初回ポーリング時点で完了済み） |

- ※1回目の結果確認時刻はポーリングループ終了時のファイル書き込み時刻。ポーリング記録上はループ4周目（投入から約40秒後）にFAILEDへ遷移している
- リトライ間隔は約2〜3分（特別な待機はせず手動で再投入）
- **合計: 11:09:32 → 11:15:11 = 約5分39秒**（リトライ待ち時間込み）

## 挙動の補足

- 429の場合も即時にエラーとはならず、POSTは正常受付（`status: RUNNING`）され、**30秒〜1分後のポーリングでFAILEDに遷移**する
- 正常時の生成所要時間はおおむね10〜60秒（多くのジョブがポーリング1〜2回目、10〜20秒でCOMPLETED）
- 当日は**17ジョブ中5件**が429でFAILED（長尺横 初回、ハガキ 初回・再投入、長尺縦再生成 1回目・2回目）。いずれも同一リクエストの再投入で成功しており、恒久的な失敗ではなく短時間の集中投入で枯渇する一時的なクォータ制限とみられる
- 検証では画像生成対象の全6サイズ（A4 / A3 / 長尺縦 / 長尺横 / ハガキ / 封筒(長形3号)。A4便箋は仕様上画像生成対象外）を**約1分以内に連続投入**しており、この同時実行バーストの状態で429が発生している。単発投入では再現していない

## 追記（2026-07-08）: 投入間隔による429回避可否の検証

「投入間隔を空ければ429を回避できるか」を確認するため、urabon1の6サイズ（A4 / A3 / 長尺縦 / 長尺横 / ハガキ / 封筒(長形3号)）を対象に2パターンを実測した（時刻はJST、eventName/eventPeriodのみの最小リクエスト、x-api-key認証）。

### 実験1: 10秒間隔で投入 → 回避できず（6件中3件が429）

| 投入時刻 | サイズ | generatedImageId | 結果 |
|---|---|---|---|
| 18:48:12 | A4 | gen-87f9beb3-474c-4a36-849d-9fa2e751aafe | COMPLETED |
| 18:48:23 | A3 | gen-aa8b5545-2f78-411b-b830-483cbfe1965f | COMPLETED |
| 18:48:33 | 長尺縦 | gen-7173aa71-351c-4326-92b4-8c9b4b870268 | FAILED（429） |
| 18:48:44 | 長尺横 | gen-ff79db22-349c-41a9-8f64-9cb5b1311baf | COMPLETED |
| 18:48:54 | ハガキ | gen-1afe4152-5325-4b19-8703-ec482ab5247e | FAILED（429） |
| 18:49:07 | 封筒(長形3号) | gen-8afd56a3-29c4-4607-b180-718fa1eea785 | FAILED（429） |

### 実験2: 完全直列投入（前ジョブの完了/失敗を確認してから次を投入）→ それでも回避できず（6件中3件が429、成功と失敗が交互）

| 投入時刻 → 結果確認時刻 | サイズ | generatedImageId | 結果 |
|---|---|---|---|
| 18:50:35 → 18:50:55 | A4 | gen-715dfe22-4899-42b9-9738-4a7a5bcef005 | COMPLETED |
| 18:50:55 → 18:51:26 | A3 | gen-88a9b7d5-0646-4517-bb9b-5eb6f745c510 | FAILED（429） |
| 18:51:26 → 18:51:47 | 長尺縦 | gen-56a3a328-9222-4b4e-92b3-2b5824313d42 | COMPLETED |
| 18:51:47 → 18:52:17 | 長尺横 | gen-c14fa89b-a0b7-4ce3-9b67-57763e4bb1b2 | FAILED（429） |
| 18:52:17 → 18:52:38 | ハガキ | gen-fdeafb82-1ed5-41cd-8de0-23982c71b938 | COMPLETED |
| 18:52:38 → 18:53:08 | 封筒(長形3号) | gen-cb788252-315b-4388-ac42-d04a8cba9552 | FAILED（429） |

### 考察

- 成功したジョブの投入間隔はおおむね50秒前後（18:50:35 / 18:51:26 / 18:52:17）で、**dev環境のAIクォータは毎分1リクエスト程度**しか許容していないとみられる
- **クライアント側の投入間隔調整だけでは回避できない**。仮に1分/件に間引くと6サイズの生成に6分以上かかり、複数ユーザーの同時利用では成立しない
- 恒久対応としては以下が必要と考えられる:
  1. AIプロバイダ側クォータの引き上げ（本命）
  2. バックエンドでの429時自動リトライまたはキューイング（クォータ枯渇時に即FAILEDにせず待って再試行）
- Bubble側の緩和策としては、FAILED（`AI_REQUEST_FAILED:429`）検知時の自動再投入導線の実装（実測では失敗の30秒〜1分後の再投入で成功している）

### 実験1・2のGET最終レスポンス全文

実験1（10秒間隔）:

```json
{"generatedImageId":"gen-87f9beb3-474c-4a36-849d-9fa2e751aafe","sourceImageId":"src-272cbd66-267f-4b5c-bd36-20747213c44f","status":"COMPLETED","fileKey":"generated-images/bubble-test-user/gen-87f9beb3-474c-4a36-849d-9fa2e751aafe/output.png","thumbnailUrl":"https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-87f9beb3-474c-4a36-849d-9fa2e751aafe/thumbnail.png"}
{"generatedImageId":"gen-aa8b5545-2f78-411b-b830-483cbfe1965f","sourceImageId":"src-125cddcf-c928-4081-89c1-5238283d28de","status":"COMPLETED","fileKey":"generated-images/bubble-test-user/gen-aa8b5545-2f78-411b-b830-483cbfe1965f/output.png","thumbnailUrl":"https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-aa8b5545-2f78-411b-b830-483cbfe1965f/thumbnail.png"}
{"generatedImageId":"gen-7173aa71-351c-4326-92b4-8c9b4b870268","sourceImageId":"src-915725fa-8f13-466f-8c3d-628def287ec5","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
{"generatedImageId":"gen-ff79db22-349c-41a9-8f64-9cb5b1311baf","sourceImageId":"src-c4144f6c-0a2c-4a94-937d-a9a175f47be1","status":"COMPLETED","fileKey":"generated-images/bubble-test-user/gen-ff79db22-349c-41a9-8f64-9cb5b1311baf/output.png","thumbnailUrl":"https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-ff79db22-349c-41a9-8f64-9cb5b1311baf/thumbnail.png"}
{"generatedImageId":"gen-1afe4152-5325-4b19-8703-ec482ab5247e","sourceImageId":"src-385b23c9-5b69-4bc1-9615-7964f9db4b64","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
{"generatedImageId":"gen-8afd56a3-29c4-4607-b180-718fa1eea785","sourceImageId":"src-cf4d527e-a233-4f51-84c7-10a223e1ce0a","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
```

実験2（完全直列）:

```json
{"generatedImageId":"gen-715dfe22-4899-42b9-9738-4a7a5bcef005","sourceImageId":"src-272cbd66-267f-4b5c-bd36-20747213c44f","status":"COMPLETED","fileKey":"generated-images/bubble-test-user/gen-715dfe22-4899-42b9-9738-4a7a5bcef005/output.png","thumbnailUrl":"https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-715dfe22-4899-42b9-9738-4a7a5bcef005/thumbnail.png"}
{"generatedImageId":"gen-88a9b7d5-0646-4517-bb9b-5eb6f745c510","sourceImageId":"src-125cddcf-c928-4081-89c1-5238283d28de","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
{"generatedImageId":"gen-56a3a328-9222-4b4e-92b3-2b5824313d42","sourceImageId":"src-915725fa-8f13-466f-8c3d-628def287ec5","status":"COMPLETED","fileKey":"generated-images/bubble-test-user/gen-56a3a328-9222-4b4e-92b3-2b5824313d42/output.png","thumbnailUrl":"https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-56a3a328-9222-4b4e-92b3-2b5824313d42/thumbnail.png"}
{"generatedImageId":"gen-c14fa89b-a0b7-4ce3-9b67-57763e4bb1b2","sourceImageId":"src-c4144f6c-0a2c-4a94-937d-a9a175f47be1","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
{"generatedImageId":"gen-fdeafb82-1ed5-41cd-8de0-23982c71b938","sourceImageId":"src-385b23c9-5b69-4bc1-9615-7964f9db4b64","status":"COMPLETED","fileKey":"generated-images/bubble-test-user/gen-fdeafb82-1ed5-41cd-8de0-23982c71b938/output.png","thumbnailUrl":"https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-fdeafb82-1ed5-41cd-8de0-23982c71b938/thumbnail.png"}
{"generatedImageId":"gen-cb788252-315b-4388-ac42-d04a8cba9552","sourceImageId":"src-cf4d527e-a233-4f51-84c7-10a223e1ce0a","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
```

## エビデンス

### 1回目（gen-7fbd32d4）

POST /generated-images/gen-feb6f18b-7d8c-481a-9d81-705dfaa72935/regenerate リクエスト（2回目・3回目も同一）:

```json
{
  "eventName": "ブラザー記念夏祭り工場見学会",
  "eventPeriod": "2026年8月23日〜24日",
  "contactName": "ブラザー工業 総務部 山田",
  "contactDetails": "TEL 052-123-4567（平日9:00〜17:00）"
}
```

POSTレスポンス（11:09:32保存）:

```json
{"generatedImageId":"gen-7fbd32d4-36f8-4a0e-b71f-41d6e833af68","parentGeneratedImageId":"gen-feb6f18b-7d8c-481a-9d81-705dfaa72935","sourceImageId":"src-f366749e-02cb-429b-bddb-e15ca25f6117","status":"RUNNING"}
```

GET /generated-images/{id} 最終レスポンス（11:10:50保存）:

```json
{"generatedImageId":"gen-7fbd32d4-36f8-4a0e-b71f-41d6e833af68","sourceImageId":"src-f366749e-02cb-429b-bddb-e15ca25f6117","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
```

ポーリング記録（10秒間隔、投入直後から）:

```
[1] 7fbd32d4:RUNNING
[2] 7fbd32d4:RUNNING
[3] 7fbd32d4:RUNNING
[4] 7fbd32d4:FAILED
[5] 7fbd32d4:FAILED
[6] 7fbd32d4:FAILED
```

### 2回目（gen-445f1a30）

POSTレスポンス（11:12:25保存）:

```json
{"generatedImageId":"gen-445f1a30-f730-40f4-8f1a-0cf7066cf99f","parentGeneratedImageId":"gen-feb6f18b-7d8c-481a-9d81-705dfaa72935","sourceImageId":"src-f366749e-02cb-429b-bddb-e15ca25f6117","status":"RUNNING"}
```

GET最終レスポンス（11:13:08保存）:

```json
{"generatedImageId":"gen-445f1a30-f730-40f4-8f1a-0cf7066cf99f","sourceImageId":"src-f366749e-02cb-429b-bddb-e15ca25f6117","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
```

ポーリング記録:

```
[1] 445f1a30:RUNNING
[2] 445f1a30:RUNNING
[3] 445f1a30:FAILED
```

### 3回目（gen-add6a20f）成功

POSTレスポンス（11:14:51保存）:

```json
{"generatedImageId":"gen-add6a20f-8e00-4fd9-9869-634d60aacb67","parentGeneratedImageId":"gen-feb6f18b-7d8c-481a-9d81-705dfaa72935","sourceImageId":"src-f366749e-02cb-429b-bddb-e15ca25f6117","status":"RUNNING"}
```

GET最終レスポンス（11:15:11保存）:

```json
{"generatedImageId":"gen-add6a20f-8e00-4fd9-9869-634d60aacb67","sourceImageId":"src-f366749e-02cb-429b-bddb-e15ca25f6117","status":"COMPLETED","fileKey":"generated-images/bubble-test-user/gen-add6a20f-8e00-4fd9-9869-634d60aacb67/output.png","thumbnailUrl":"https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-add6a20f-8e00-4fd9-9869-634d60aacb67/thumbnail.png"}
```

ポーリング記録:

```
[1] add6a20f:COMPLETED
```

### 参考: 当日429でFAILEDした他の3ジョブのGETレスポンス

初回生成 長尺横（urabon1、1回目投入分）:

```json
{"generatedImageId":"gen-1ead186a-f276-4018-a7a0-369fe145390d","sourceImageId":"src-c4144f6c-0a2c-4a94-937d-a9a175f47be1","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
```

初回生成 ハガキ（urabon1、1回目投入分）:

```json
{"generatedImageId":"gen-eb509f44-b6ed-4f91-984f-da9bdffd97c4","sourceImageId":"src-385b23c9-5b69-4bc1-9615-7964f9db4b64","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
```

初回生成 ハガキ（urabon1、2回目投入分。3回目で成功=gen-cbbd64cc）:

```json
{"generatedImageId":"gen-cafa692a-b491-4752-8de7-43afd8e70991","sourceImageId":"src-385b23c9-5b69-4bc1-9615-7964f9db4b64","status":"FAILED","errorCode":"AI_GENERATION_FAILED","errorMessage":"AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."}
```
