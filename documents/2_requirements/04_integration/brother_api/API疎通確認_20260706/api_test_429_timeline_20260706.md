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
- 当日は17ジョブ中4件が429でFAILED。いずれも同一リクエストの再投入で成功しており、恒久的な失敗ではなく短時間の集中投入で枯渇する一時的なクォータ制限とみられる（検証のため数分間に複数ジョブを連続投入していた点は考慮が必要）

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
