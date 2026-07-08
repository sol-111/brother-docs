# API疎通確認（issue修正版）検証結果

- 実施日時: 2026-07-06
- 対象環境: brother-backend-dev-yokinini
- Base URL: `https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod`
- 経緯: ブラザー工業 柳さまより「前回のissue対応 + contactName/contactDetails追加対応で生成AI系APIを更新したため疎通確認をお願いしたい」旨の連絡（2026-07-05メール）を受け、前回（API疎通確認_20260610）の確認事項7件の再検証を実施
- 前回の確認事項: `../API疎通確認_20260610/api_test_issues_20260610.md`

---

## 判定サマリ

| # | 前回の事象 | 判定 | 備考 |
|---|---|---|---|
| 1 | 再生成時にリクエストにない文言が出力される | ✅ 解消 | 前回と同一ペイロードの再生成で「盂蘭盆会法要」「堀田太郎師」等の混入なし |
| 2 | contactName / contactDetails が未実装 | ✅ 実装済み | 初回生成・再生成とも全サイズで受理され、出力画像にも描画される。※長尺縦はcontactNameのみ描画（サイズ別描画項目マトリクスどおり） |
| 3 | A3の生成結果が長尺縦と同じレイアウト | ✅ 解消 | SourceImage登録不備が原因（Bubble側回答で確定）。不備ソースは旧グループごと削除済み（追記2参照）。urabonグループのA3は正常（896×1195） |
| 4 | dev環境に3サイズしか未登録 | ✅ 解消 | urabon1/2/3の3グループが追加され、各7サイズ登録（A3/A4/A4便箋/ハガキ/封筒(長形3号)/長尺横/長尺縦） |
| 5 | フィールド名が仕様書と異なる | 変更なし（運用対応） | 実装はリリースガイドのフィールド名（eventPeriod等）のまま。サイズ別入力制約のAPI側400は撤廃され、N/A項目を送信しない責務はクライアント側に整理（サイズ別描画項目マトリクス注記どおり） |
| 6 | POSTレスポンスのstatusが `done` | ✅ 修正 | `RUNNING` が返却されるようになった（仕様書準拠に変更）。リリースガイドの記載（`done`）の方が古くなった点に注意 |
| 7 | ポーリング中のstatusが `PROCESSING` | ✅ 変更 | `RUNNING` が返却されるようになった（仕様書準拠に変更）。**PROCESSING前提のポーリング判定は要修正** |

### 新規確認事項（ブラザー側への確認・依頼候補）

| # | 事象 | 重要度 |
|---|---|---|
| A | AI側クォータ制限（429）が頻発。本検証17ジョブ中4件が `AI_REQUEST_FAILED:429` でFAILED。同一リクエストの再投入で成功するため一過性だが、Bubble側のリトライ導線とdev環境クォータの引き上げ要否を要検討 | 高 |
| D | 封筒(長形3号)の出力で eventName が「ブラザー夏祭り 夏祭り盆踊り会」と一部重複して描画された → **Bubble側回答（7/6）: 検討中** | 低 |

---

## 検証1: ソースイメージ登録状況（事象3・4）

### GET /source-image-groups

```bash
curl -s https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/source-image-groups | jq .
```

```json
{
  "items": [
    {
      "sourceImageGroupId": "urabon3",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/urabon3/src-2ac977ea-b2bf-4011-8612-8683ee555792/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageGroupId": "urabon1",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/urabon1/src-272cbd66-267f-4b5c-bd36-20747213c44f/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageGroupId": "sig3",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/sig3/src-f366749e-02cb-429b-bddb-e15ca25f6117/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageGroupId": "sig1",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/sig1/src-5058b17b-eb72-4f0a-933d-b6b75ab43a6c/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageGroupId": "urabon2",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/urabon2/src-cd2a6c4a-9f11-40a7-8722-40276e6b57a1/thumbnail.png",
      "status": "active"
    }
  ]
}
```

前回の2グループ（sig1, sig3）に加え、**urabon1 / urabon2 / urabon3 の3グループが追加**されていた。

### グループ別サイズ登録状況（GET /source-image-groups/{id}/source-images）

| サイズ | sig1 | sig3 | urabon1 | urabon2 | urabon3 |
|---|---|---|---|---|---|
| A3 | ○（2件） | ○（3件）※縦長のまま | ○ src-125cddcf-c928-4081-89c1-5238283d28de | ○ src-947d7e04 | ○ src-b1ec6f12 |
| A4 | ○（5件） | — | ○ src-272cbd66 | ○ src-cd2a6c4a | ○ src-2ac977ea |
| A4便箋 | — | — | ○ src-46cdf9ef | ○ src-a2804936 | ○ src-89e45d86 |
| ハガキ | ○（1件） | — | ○ src-385b23c9-5b69-4bc1-9615-7964f9db4b64 | ○ src-d90ce00f | ○ src-8036502d |
| 封筒(長形3号) | — | — | ○ src-cf4d527e-a233-4f51-84c7-10a223e1ce0a | ○ src-a6d93a6a | ○ src-5fb34a54 |
| 長尺横 | — | — | ○ src-c4144f6c-0a2c-4a94-937d-a9a175f47be1 | ○ src-e8156270 | ○ src-270de017 |
| 長尺縦 | — | ○ src-f366749e | ○ src-915725fa | ○ src-fccbbe53 | ○ src-ac4bfb56 |

- 事象4（長尺横/ハガキ/封筒 長形3号が未登録）は**解消**。
- サイズ名の表記は「封筒(長形3号)」。仕様の6サイズに加えて「**A4便箋**」という新サイズが存在する（仕様書未記載、Bubble側のサイズ出し分け実装に影響する可能性あり）。
- **sig3のA3ソースは未修正**: `src-b9bfb2fc-0b50-4137-81d5-797969548444`（size: A3）のサムネイル実寸は **512×1551px（縦長＝長尺縦相当）** のまま。一方urabon1のA3（src-125cddcf）のサムネイルは1008×1344pxでA3縦の比率。
  - sig3 A3サムネイル: `https://d3p6nztoczy84t.cloudfront.net/source-images/sig3/src-b9bfb2fc-0b50-4137-81d5-797969548444/thumbnail.png`
  - urabon1 A3サムネイル: `https://d3p6nztoczy84t.cloudfront.net/source-images/urabon1/src-125cddcf-c928-4081-89c1-5238283d28de/thumbnail.png`

---

## 検証2: 生成ジョブ一覧

| No | 種別 | サイズ（グループ） | sourceImageId | generatedImageId | 結果 |
|---|---|---|---|---|---|
| 1 | 初回 | A4（sig1） | src-bubble-1780448083182 | gen-44ac1366-e147-454c-b741-7f864ffcc7b3 | COMPLETED |
| 2 | 初回 | A3（sig3） | src-b9bfb2fc-0b50-4137-81d5-797969548444 | gen-4d9757c1-d9fa-4e14-ad9b-059a4c95ec1f | COMPLETED（レイアウトNG） |
| 3 | 初回 | 長尺縦（sig3） | src-f366749e-02cb-429b-bddb-e15ca25f6117 | gen-feb6f18b-7d8c-481a-9d81-705dfaa72935 | COMPLETED（文言NG） |
| 4 | 初回 | 長尺横（urabon1） | src-c4144f6c-0a2c-4a94-937d-a9a175f47be1 | gen-1ead186a-f276-4018-a7a0-369fe145390d | FAILED（429） |
| 5 | 初回 | ハガキ（urabon1） | src-385b23c9-5b69-4bc1-9615-7964f9db4b64 | gen-eb509f44-b6ed-4f91-984f-da9bdffd97c4 | FAILED（429） |
| 6 | 初回 | 封筒 長形3号（urabon1） | src-cf4d527e-a233-4f51-84c7-10a223e1ce0a | gen-b1a822cb-55a5-4c83-a69b-904a07cfd0c6 | COMPLETED |
| 7 | 初回 | 長尺縦・制約外フィールド送信（sig3） | src-f366749e-02cb-429b-bddb-e15ca25f6117 | gen-5ee17a58-98de-4e7d-8821-4c2d919f1342 | COMPLETED（400にならず） |
| 8 | 再生成 | A4（親: No.1） | src-bubble-1780448083182 | gen-e652baf8-2268-41e7-809b-f7134f0a40cd | COMPLETED |
| 9 | 再生成 | A3（sig3、親: No.2） | src-b9bfb2fc-0b50-4137-81d5-797969548444 | gen-7013514e-543e-4e4a-9f4a-7cc2d7c7805b | COMPLETED（レイアウトNG） |
| 10 | 再生成 | 長尺縦（親: No.3）1回目 | src-f366749e-02cb-429b-bddb-e15ca25f6117 | gen-7fbd32d4-36f8-4a0e-b71f-41d6e833af68 | FAILED（429） |
| 11 | 初回 | 長尺横（urabon1）再投入 | src-c4144f6c-0a2c-4a94-937d-a9a175f47be1 | gen-59c52e75-8efc-4d9f-a7c3-c253a3299594 | COMPLETED |
| 12 | 初回 | ハガキ（urabon1）再投入 | src-385b23c9-5b69-4bc1-9615-7964f9db4b64 | gen-cafa692a-b491-4752-8de7-43afd8e70991 | FAILED（429） |
| 13 | 初回 | A3（urabon1） | src-125cddcf-c928-4081-89c1-5238283d28de | gen-ac8ff65d-cbd0-48e3-b2c0-45e2f9f770c1 | COMPLETED |
| 14 | 初回 | ハガキ（urabon1）再々投入 | src-385b23c9-5b69-4bc1-9615-7964f9db4b64 | gen-cbbd64cc-6ec4-4fa2-9509-b6540526e702 | COMPLETED |
| 15 | 再生成 | 長尺縦（親: No.3）2回目 | src-f366749e-02cb-429b-bddb-e15ca25f6117 | gen-445f1a30-f730-40f4-8f1a-0cf7066cf99f | FAILED（429） |
| 16 | 再生成 | 長尺縦（親: No.3）3回目 | src-f366749e-02cb-429b-bddb-e15ca25f6117 | gen-add6a20f-8e00-4fd9-9869-634d60aacb67 | COMPLETED |
| 17 | 再生成 | A3（urabon1、親: No.13） | src-125cddcf-c928-4081-89c1-5238283d28de | gen-fa32c8b2-5f08-4177-995a-5feb6f619d21 | COMPLETED |

リクエスト文言（初回生成）:

```json
{
  "eventName": "ブラザー夏祭り盆踊り会",
  "eventPeriod": "2026年8月23日",
  "eventDetails": "境内にて夏祭りを開催いたします",
  "eventProgram": "10:00 開会 / 12:00 盆踊り / 15:00 閉会",
  "notes": "雨天時は中止となります",
  "contactName": "ブラザー工業 総務部 山田",
  "contactDetails": "TEL 052-123-4567（平日9:00〜17:00）"
}
```

※長尺縦・長尺横・ハガキ・封筒は eventName / eventPeriod / contactName / contactDetails のみ送信。

リクエスト文言（再生成、前回issueの再現ペイロード + contact）:

```json
{
  "eventName": "ブラザー記念夏祭り工場見学会",
  "eventPeriod": "2026年8月23日〜24日",
  "eventDetails": "工場敷地内にて夏祭りと工場見学会を開催",
  "eventProgram": "10:00 工場見学 / 12:00 昼食 / 14:00 盆踊り / 16:00 閉会",
  "notes": "雨天決行・荒天中止",
  "contactName": "ブラザー工業 総務部 山田",
  "contactDetails": "TEL 052-123-4567（平日9:00〜17:00）"
}
```

※長尺縦の再生成は eventName / eventPeriod / contactName / contactDetails のみ送信。

---

## 検証3: 事象別詳細

### 事象1: 再生成時のリクエストにない文言 → ✅ 解消

前回問題の再現ケース（長尺縦を親に eventName「ブラザー記念夏祭り工場見学会」で再生成）を実行。

- 再生成 長尺縦（gen-add6a20f）: 出力は「ブラザー 記念夏祭り工場見学会 / 2026年8月23日〜24日 / ブラザー工業 総務部 山田」のみ。前回混入していた「盂蘭盆会法要」等は**なし**。
- 再生成 A4（gen-e652baf8）: リクエスト文言＋連絡先のみ。前回混入していた「兄弟合主幹 堀田 太郎 師…」は**なし**。
- 再生成 A3 urabon1（gen-fa32c8b2）: リクエスト文言＋連絡先のみで完全に正常。

※例外: 旧sig3のA3ソース（src-b9bfb2fc）を親にした再生成（gen-7013514e）では、初回生成時と同様にテンプレ由来の文言（「法学会」「令和八年8○月○○日」「○○寺」等）が混在。これは再生成固有の問題ではなく、ソース画像未修正（事象3）に起因するデータ問題と判断。

### 事象2: contactName / contactDetails → ✅ 実装済み

- POST /generated-images および POST /generated-images/{id}/regenerate の両方で `contactName` / `contactDetails` を送信し、全サイズで 400 にならず受理された。
- 出力画像にも描画される（A4/A3(urabon1)/ハガキ/封筒/長尺横: contactName+contactDetails 描画。長尺縦: contactName のみ描画。いずれもサイズ別描画項目マトリクスどおり）。

### 事象3: A3が長尺縦レイアウト → ✅ 解消（不備ソースは削除済み・追記2参照）

| ソース | 生成画像実寸 | 判定 |
|---|---|---|
| sig3 src-b9bfb2fc（A3）初回 gen-4d9757c1 | **592×1794px（縦長）** | NG（前回と同じ） |
| sig3 src-b9bfb2fc（A3）再生成 gen-7013514e | **592×1794px（縦長）** | NG |
| urabon1 src-125cddcf（A3）初回 gen-ac8ff65d | 896×1195px（A3縦相当） | OK |
| urabon1 src-125cddcf（A3）再生成 gen-fa32c8b2 | 896×1195px（A3縦相当） | OK |

sig3のA3はソース画像そのものが縦長（サムネイル512×1551px）のままのため、生成結果も縦長になる（検証時点の記録。その後Bubble側回答で登録不備と確定し、旧グループごと削除済み）。

### 事象4: サイズ登録 → ✅ 解消

検証1の通り、urabon1/2/3の各グループに7サイズが登録済み。長尺横（gen-59c52e75、1794×592px）・ハガキ（gen-cbbd64cc、832×1280px）・封筒 長形3号（gen-b1a822cb、736×1440px）の生成も成功。

### 事象5: フィールド名 → 変更なし + サイズ別入力制約の撤廃（N/A項目はクライアント側で送信しない責務に整理）

- 実装はリリースガイドのフィールド名（eventName / eventPeriod / eventDetails / eventProgram / notes）のまま。従来方針どおりリリースガイド準拠で実装する。
- 前回は長尺縦に eventDetails / eventProgram / notes を送ると `400 INVALID_REQUEST`（"size=長尺縦 does not allow fields …"）だったが、今回同じリクエスト（gen-5ee17a58）は**受理され生成が完走した**。サイズ別入力制約が撤廃されたのか、意図した変更かをブラザー側に確認する。

```bash
# 前回400だったリクエスト（今回はRUNNINGで受理された）
curl -s -X POST $BASE/generated-images -H "Content-Type: application/json" -d '{
  "sourceImageId": "src-f366749e-02cb-429b-bddb-e15ca25f6117",
  "eventName": "テスト",
  "eventPeriod": "2026年8月23日",
  "eventDetails": "テスト詳細",
  "eventProgram": "テストプログラム",
  "notes": "テスト備考"
}'
```

```json
{
  "generatedImageId": "gen-5ee17a58-98de-4e7d-8821-4c2d919f1342",
  "sourceImageId": "src-f366749e-02cb-429b-bddb-e15ca25f6117",
  "status": "RUNNING"
}
```

### 事象6: POSTレスポンスstatus → ✅ 修正（done → RUNNING）

全17ジョブのPOSTレスポンスが `"status": "RUNNING"` を返却。仕様書（PENDING/RUNNING）準拠に変更された。リリースガイド記載の `done` はもはや返却されないため、**Bubble側で `done` を受付完了と判定する実装にしている場合は修正が必要**。

例（初回生成 A4）:

```json
{
  "generatedImageId": "gen-44ac1366-e147-454c-b741-7f864ffcc7b3",
  "sourceImageId": "src-bubble-1780448083182",
  "status": "RUNNING"
}
```

例（再生成 A4）:

```json
{
  "generatedImageId": "gen-e652baf8-2268-41e7-809b-f7134f0a40cd",
  "parentGeneratedImageId": "gen-44ac1366-e147-454c-b741-7f864ffcc7b3",
  "sourceImageId": "src-bubble-1780448083182",
  "status": "RUNNING"
}
```

### 事象7: ポーリングstatus → ✅ 変更（PROCESSING → RUNNING）

GET /generated-images/{id} の処理中statusは `RUNNING` を返却（仕様書準拠に変更）。`PROCESSING` は観測されなかった。**PROCESSING をポーリング継続条件にしている場合は RUNNING への変更が必要**（COMPLETED / FAILED 以外は継続、とする実装を推奨）。

処理中の例:

```json
{
  "generatedImageId": "gen-feb6f18b-7d8c-481a-9d81-705dfaa72935",
  "sourceImageId": "src-f366749e-02cb-429b-bddb-e15ca25f6117",
  "status": "RUNNING"
}
```

完了の例:

```json
{
  "generatedImageId": "gen-44ac1366-e147-454c-b741-7f864ffcc7b3",
  "sourceImageId": "src-bubble-1780448083182",
  "status": "COMPLETED",
  "fileKey": "generated-images/bubble-test-user/gen-44ac1366-e147-454c-b741-7f864ffcc7b3/output.png",
  "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-44ac1366-e147-454c-b741-7f864ffcc7b3/thumbnail.png"
}
```

失敗（429）の例:

```json
{
  "generatedImageId": "gen-1ead186a-f276-4018-a7a0-369fe145390d",
  "sourceImageId": "src-c4144f6c-0a2c-4a94-937d-a9a175f47be1",
  "status": "FAILED",
  "errorCode": "AI_GENERATION_FAILED",
  "errorMessage": "AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."
}
```

---

## 検証4: 生成画像の目視確認結果

保存先: `01_初回生成_印刷物/` `02_再生成_印刷物/`

| ファイル | 実寸 | 目視確認結果 |
|---|---|---|
| 01_初回生成_印刷物/初回_A4.png | 864×1248 | リクエスト文言・連絡先とも正しく描画。※リクエストにない固定文言「檀信徒の皆様はもちろん、…お寺へお参りください。」あり（ソースは削除済みの旧グループのため対応不要。文言混入の問題は確認事項Gで管理） |
| 01_初回生成_印刷物/初回_A3.png（sig3） | 592×1794 | NG: 縦長レイアウトのまま。「盂蘭盆会法要」「○○寺」「令和八年8○月○○日」などテンプレ文言が残存し、リクエスト文言と混在 |
| 01_初回生成_印刷物/初回_長尺縦.png（sig3） | 592×1794 | NG: 「ブラザー 盂蘭盆踊会 法要」のようにテンプレ文言と混在。日付も「令和八年年○月○○日」と崩れ。contactNameのみ描画 |
| 01_初回生成_印刷物/初回_長尺縦_制約外フィールド.png | 592×1794 | 参考: 「盂蘭盆テスト」とテンプレ文言に合成。eventDetails等の描画なし |
| 01_初回生成_印刷物/初回_長尺横.png（urabon1） | 1794×592 | eventName・contactName描画OK。eventPeriodは非描画（サイズ別描画項目マトリクスどおり） |
| 01_初回生成_印刷物/初回_ハガキ.png（urabon1） | 832×1280 | OK: eventName・eventPeriod・連絡先とも正しく描画。余計な文言なし |
| 01_初回生成_印刷物/初回_封筒長形3号.png（urabon1） | 736×1440 | 概ねOK: 連絡先描画OK。eventNameが「ブラザー夏祭り 夏祭り盆踊り会」と一部重複（確認事項D） |
| 01_初回生成_印刷物/初回_A3_urabon1.png | 896×1195 | OK: 全項目正しく描画。余計な文言なし。A3縦の比率 |
| 02_再生成_印刷物/再生成_A4.png | 864×1248 | OK: 再生成文言に全て更新。前回の「堀田太郎師」等の混入なし。※初回と同じ固定文言あり |
| 02_再生成_印刷物/再生成_A3.png（sig3） | 592×1794 | NG: 初回と同様、テンプレ文言残存＋縦長レイアウト（事象3由来） |
| 02_再生成_印刷物/再生成_A3_urabon1.png | 896×1195 | OK: 全項目正しく描画。余計な文言なし |
| 02_再生成_印刷物/再生成_長尺縦.png（sig3） | 592×1794 | OK: 「ブラザー 記念夏祭り工場見学会 / 2026年8月23日〜24日 / ブラザー工業 総務部 山田」のみ。**前回issueの文言混入は解消** |

---

## エビデンス: ファイルダウンロードURL取得（GET /files/url/{fileKey+}）レスポンス全文

※署名付きURLの有効期限は600秒のため、以下のURLは既に失効している。恒久参照はCloudFrontのthumbnailUrl、または再度 /files/url を叩くこと。

### gen-44ac1366（初回 A4）

```json
{
  "url": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/generated-images/bubble-test-user/gen-44ac1366-e147-454c-b741-7f864ffcc7b3/output.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3IFUVHKE2%2F20260706%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260706T020811Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEIL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLW5vcnRoZWFzdC0xIkUwQwIgARnZbhAiWLRlIkK9Z5Vw9gSNUvJ2L8MuuryUrx%2BPNicCH3InLp7b5rH1ShoPjxo%2FDQVRtC0mt%2Fn0kdeqErEyXggq9wQISxAAGgwzMzI4OTY5Mzg2MTQiDDeHJMs3g%2F9yi4qi1CrUBFlTmv%2B5YhzZ4siwoEBguN1zX96uczA54mFEtr91cgXnwNHHN6HrGhQNyPNB8FfnHItE8chd9byQ4A4AFDC2X45H%2FWRNCAJFgQMIwnfBFnF2MfneiHCHS%2BNZpjJ4m2UKYf98tyIav52w4an6gsd6vG0pkuPrIrxPb3nPPBCx4e1vM3zwdg58KjWk305hPr1npvSlyJfONsziOQNZUbQJ%2BK4DMBmITYNUDSQAiXjcBBIfcg8%2BRItOZ3VaiENfKDLN2sLspzKGbEYpcyefSCbKD7TiGO3ps320cC4CrmdnLpD8mrARHdzksp6kR%2BwCO64zDwhL67tQxAgZZKETtP6YEsVjRN1T6JmWUZ2B3r0H6Z4SOIvFnOgq5ND2mEv%2FS54QJnkGBwdSjpn4tKRtSXpo%2BSerMUzI5c8NrWdxwEVZFZy4%2BN28UhtkMzAeHal0VLdRLsDP4lHz%2FHLlTwRj8YiQTRwTCoesJ%2BtbJU%2FhrzGirSLNNF8Gbw5qwhJPSJS9xkN6Zic0RwT0ruQ7%2FROVhucra%2BdcqxdX7lyqUZ1bVskmmxTbnqZ4XwL7vH%2BNzKbfm3Z%2BWfkrqUnhbIS6oalx1Fhquod7PTRaPUmLQfacdFSWom6viXn57EbX4Ic0nmCiLsI0FSuMwiL8a2Kq8XtpC4XjiRe7IW3RhJg4x6u8YoOctXfHNh8CF%2FZgBrk%2BtVZuRCFzET%2BdC1rwsP%2F2jt%2BN8Ke0TkT4u%2BiYZXD6PO0EzjfPBtq5iE5HES%2FSTWvw%2By8a%2FG0n58j8C3ncTf8aFW6pYzzHS%2FrQX%2FNdMImcrNIGOqUBgR7k9jfQJHtLyjnfVwkMdLEsNp4xBzN%2Bu0rzBkO259Cnlq2KiI3Rl%2BkF1qASgpqzaZQ6e93FkmWYlpAo%2BYdKDUKDGWLWlSDs8BtBMWYK4PumpOz31WVpEpSrbRa8uBs0L9LcQfX%2FZbf%2BmI%2FDgvqvaaqIhYZ5qMNWSNJPC3VHL%2BR000wcHTG%2BQ4ph3DVCBpo4rPiCtq4ZVIEo88TCfgf8IqNvN6Fn&X-Amz-Signature=24d8e28d4fdcde4db87624dcdb6cdff4b12312723849b93e0d9692bcca2fd007&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
  "expiresAt": "2026-07-06T02:18:11.625Z"
}
```

### gen-4d9757c1（初回 A3 sig3）

```json
{
  "url": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/generated-images/bubble-test-user/gen-4d9757c1-d9fa-4e14-ad9b-059a4c95ec1f/output.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3IFUVHKE2%2F20260706%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260706T020810Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEIL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLW5vcnRoZWFzdC0xIkUwQwIgARnZbhAiWLRlIkK9Z5Vw9gSNUvJ2L8MuuryUrx%2BPNicCH3InLp7b5rH1ShoPjxo%2FDQVRtC0mt%2Fn0kdeqErEyXggq9wQISxAAGgwzMzI4OTY5Mzg2MTQiDDeHJMs3g%2F9yi4qi1CrUBFlTmv%2B5YhzZ4siwoEBguN1zX96uczA54mFEtr91cgXnwNHHN6HrGhQNyPNB8FfnHItE8chd9byQ4A4AFDC2X45H%2FWRNCAJFgQMIwnfBFnF2MfneiHCHS%2BNZpjJ4m2UKYf98tyIav52w4an6gsd6vG0pkuPrIrxPb3nPPBCx4e1vM3zwdg58KjWk305hPr1npvSlyJfONsziOQNZUbQJ%2BK4DMBmITYNUDSQAiXjcBBIfcg8%2BRItOZ3VaiENfKDLN2sLspzKGbEYpcyefSCbKD7TiGO3ps320cC4CrmdnLpD8mrARHdzksp6kR%2BwCO64zDwhL67tQxAgZZKETtP6YEsVjRN1T6JmWUZ2B3r0H6Z4SOIvFnOgq5ND2mEv%2FS54QJnkGBwdSjpn4tKRtSXpo%2BSerMUzI5c8NrWdxwEVZFZy4%2BN28UhtkMzAeHal0VLdRLsDP4lHz%2FHLlTwRj8YiQTRwTCoesJ%2BtbJU%2FhrzGirSLNNF8Gbw5qwhJPSJS9xkN6Zic0RwT0ruQ7%2FROVhucra%2BdcqxdX7lyqUZ1bVskmmxTbnqZ4XwL7vH%2BNzKbfm3Z%2BWfkrqUnhbIS6oalx1Fhquod7PTRaPUmLQfacdFSWom6viXn57EbX4Ic0nmCiLsI0FSuMwiL8a2Kq8XtpC4XjiRe7IW3RhJg4x6u8YoOctXfHNh8CF%2FZgBrk%2BtVZuRCFzET%2BdC1rwsP%2F2jt%2BN8Ke0TkT4u%2BiYZXD6PO0EzjfPBtq5iE5HES%2FSTWvw%2By8a%2FG0n58j8C3ncTf8aFW6pYzzHS%2FrQX%2FNdMImcrNIGOqUBgR7k9jfQJHtLyjnfVwkMdLEsNp4xBzN%2Bu0rzBkO259Cnlq2KiI3Rl%2BkF1qASgpqzaZQ6e93FkmWYlpAo%2BYdKDUKDGWLWlSDs8BtBMWYK4PumpOz31WVpEpSrbRa8uBs0L9LcQfX%2FZbf%2BmI%2FDgvqvaaqIhYZ5qMNWSNJPC3VHL%2BR000wcHTG%2BQ4ph3DVCBpo4rPiCtq4ZVIEo88TCfgf8IqNvN6Fn&X-Amz-Signature=3fc46c220a54b2555f38c784a159c8185dd98dfcf512bef86e897fab188cc591&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
  "expiresAt": "2026-07-06T02:18:10.651Z"
}
```

### gen-feb6f18b（初回 長尺縦 sig3）

```json
{
  "url": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/generated-images/bubble-test-user/gen-feb6f18b-7d8c-481a-9d81-705dfaa72935/output.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3IFUVHKE2%2F20260706%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260706T020812Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEIL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLW5vcnRoZWFzdC0xIkUwQwIgARnZbhAiWLRlIkK9Z5Vw9gSNUvJ2L8MuuryUrx%2BPNicCH3InLp7b5rH1ShoPjxo%2FDQVRtC0mt%2Fn0kdeqErEyXggq9wQISxAAGgwzMzI4OTY5Mzg2MTQiDDeHJMs3g%2F9yi4qi1CrUBFlTmv%2B5YhzZ4siwoEBguN1zX96uczA54mFEtr91cgXnwNHHN6HrGhQNyPNB8FfnHItE8chd9byQ4A4AFDC2X45H%2FWRNCAJFgQMIwnfBFnF2MfneiHCHS%2BNZpjJ4m2UKYf98tyIav52w4an6gsd6vG0pkuPrIrxPb3nPPBCx4e1vM3zwdg58KjWk305hPr1npvSlyJfONsziOQNZUbQJ%2BK4DMBmITYNUDSQAiXjcBBIfcg8%2BRItOZ3VaiENfKDLN2sLspzKGbEYpcyefSCbKD7TiGO3ps320cC4CrmdnLpD8mrARHdzksp6kR%2BwCO64zDwhL67tQxAgZZKETtP6YEsVjRN1T6JmWUZ2B3r0H6Z4SOIvFnOgq5ND2mEv%2FS54QJnkGBwdSjpn4tKRtSXpo%2BSerMUzI5c8NrWdxwEVZFZy4%2BN28UhtkMzAeHal0VLdRLsDP4lHz%2FHLlTwRj8YiQTRwTCoesJ%2BtbJU%2FhrzGirSLNNF8Gbw5qwhJPSJS9xkN6Zic0RwT0ruQ7%2FROVhucra%2BdcqxdX7lyqUZ1bVskmmxTbnqZ4XwL7vH%2BNzKbfm3Z%2BWfkrqUnhbIS6oalx1Fhquod7PTRaPUmLQfacdFSWom6viXn57EbX4Ic0nmCiLsI0FSuMwiL8a2Kq8XtpC4XjiRe7IW3RhJg4x6u8YoOctXfHNh8CF%2FZgBrk%2BtVZuRCFzET%2BdC1rwsP%2F2jt%2BN8Ke0TkT4u%2BiYZXD6PO0EzjfPBtq5iE5HES%2FSTWvw%2By8a%2FG0n58j8C3ncTf8aFW6pYzzHS%2FrQX%2FNdMImcrNIGOqUBgR7k9jfQJHtLyjnfVwkMdLEsNp4xBzN%2Bu0rzBkO259Cnlq2KiI3Rl%2BkF1qASgpqzaZQ6e93FkmWYlpAo%2BYdKDUKDGWLWlSDs8BtBMWYK4PumpOz31WVpEpSrbRa8uBs0L9LcQfX%2FZbf%2BmI%2FDgvqvaaqIhYZ5qMNWSNJPC3VHL%2BR000wcHTG%2BQ4ph3DVCBpo4rPiCtq4ZVIEo88TCfgf8IqNvN6Fn&X-Amz-Signature=e77345387e0cc5a788c530181f3bcec2054d37ca509cd0bfc9371b9ab6431ddd&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
  "expiresAt": "2026-07-06T02:18:12.953Z"
}
```

### gen-b1a822cb（初回 封筒 長形3号 urabon1）

```json
{
  "url": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/generated-images/bubble-test-user/gen-b1a822cb-55a5-4c83-a69b-904a07cfd0c6/output.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3IFUVHKE2%2F20260706%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260706T020813Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEIL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLW5vcnRoZWFzdC0xIkUwQwIgARnZbhAiWLRlIkK9Z5Vw9gSNUvJ2L8MuuryUrx%2BPNicCH3InLp7b5rH1ShoPjxo%2FDQVRtC0mt%2Fn0kdeqErEyXggq9wQISxAAGgwzMzI4OTY5Mzg2MTQiDDeHJMs3g%2F9yi4qi1CrUBFlTmv%2B5YhzZ4siwoEBguN1zX96uczA54mFEtr91cgXnwNHHN6HrGhQNyPNB8FfnHItE8chd9byQ4A4AFDC2X45H%2FWRNCAJFgQMIwnfBFnF2MfneiHCHS%2BNZpjJ4m2UKYf98tyIav52w4an6gsd6vG0pkuPrIrxPb3nPPBCx4e1vM3zwdg58KjWk305hPr1npvSlyJfONsziOQNZUbQJ%2BK4DMBmITYNUDSQAiXjcBBIfcg8%2BRItOZ3VaiENfKDLN2sLspzKGbEYpcyefSCbKD7TiGO3ps320cC4CrmdnLpD8mrARHdzksp6kR%2BwCO64zDwhL67tQxAgZZKETtP6YEsVjRN1T6JmWUZ2B3r0H6Z4SOIvFnOgq5ND2mEv%2FS54QJnkGBwdSjpn4tKRtSXpo%2BSerMUzI5c8NrWdxwEVZFZy4%2BN28UhtkMzAeHal0VLdRLsDP4lHz%2FHLlTwRj8YiQTRwTCoesJ%2BtbJU%2FhrzGirSLNNF8Gbw5qwhJPSJS9xkN6Zic0RwT0ruQ7%2FROVhucra%2BdcqxdX7lyqUZ1bVskmmxTbnqZ4XwL7vH%2BNzKbfm3Z%2BWfkrqUnhbIS6oalx1Fhquod7PTRaPUmLQfacdFSWom6viXn57EbX4Ic0nmCiLsI0FSuMwiL8a2Kq8XtpC4XjiRe7IW3RhJg4x6u8YoOctXfHNh8CF%2FZgBrk%2BtVZuRCFzET%2BdC1rwsP%2F2jt%2BN8Ke0TkT4u%2BiYZXD6PO0EzjfPBtq5iE5HES%2FSTWvw%2By8a%2FG0n58j8C3ncTf8aFW6pYzzHS%2FrQX%2FNdMImcrNIGOqUBgR7k9jfQJHtLyjnfVwkMdLEsNp4xBzN%2Bu0rzBkO259Cnlq2KiI3Rl%2BkF1qASgpqzaZQ6e93FkmWYlpAo%2BYdKDUKDGWLWlSDs8BtBMWYK4PumpOz31WVpEpSrbRa8uBs0L9LcQfX%2FZbf%2BmI%2FDgvqvaaqIhYZ5qMNWSNJPC3VHL%2BR000wcHTG%2BQ4ph3DVCBpo4rPiCtq4ZVIEo88TCfgf8IqNvN6Fn&X-Amz-Signature=a254e9db423d7b6a7315d081d585a80f2b4a86fd504e112ce6bc6dbb62b48d57&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
  "expiresAt": "2026-07-06T02:18:13.781Z"
}
```

### gen-5ee17a58（初回 長尺縦 制約外フィールド）

```json
{
  "url": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/generated-images/bubble-test-user/gen-5ee17a58-98de-4e7d-8821-4c2d919f1342/output.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3IFUVHKE2%2F20260706%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260706T020814Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEIL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLW5vcnRoZWFzdC0xIkUwQwIgARnZbhAiWLRlIkK9Z5Vw9gSNUvJ2L8MuuryUrx%2BPNicCH3InLp7b5rH1ShoPjxo%2FDQVRtC0mt%2Fn0kdeqErEyXggq9wQISxAAGgwzMzI4OTY5Mzg2MTQiDDeHJMs3g%2F9yi4qi1CrUBFlTmv%2B5YhzZ4siwoEBguN1zX96uczA54mFEtr91cgXnwNHHN6HrGhQNyPNB8FfnHItE8chd9byQ4A4AFDC2X45H%2FWRNCAJFgQMIwnfBFnF2MfneiHCHS%2BNZpjJ4m2UKYf98tyIav52w4an6gsd6vG0pkuPrIrxPb3nPPBCx4e1vM3zwdg58KjWk305hPr1npvSlyJfONsziOQNZUbQJ%2BK4DMBmITYNUDSQAiXjcBBIfcg8%2BRItOZ3VaiENfKDLN2sLspzKGbEYpcyefSCbKD7TiGO3ps320cC4CrmdnLpD8mrARHdzksp6kR%2BwCO64zDwhL67tQxAgZZKETtP6YEsVjRN1T6JmWUZ2B3r0H6Z4SOIvFnOgq5ND2mEv%2FS54QJnkGBwdSjpn4tKRtSXpo%2BSerMUzI5c8NrWdxwEVZFZy4%2BN28UhtkMzAeHal0VLdRLsDP4lHz%2FHLlTwRj8YiQTRwTCoesJ%2BtbJU%2FhrzGirSLNNF8Gbw5qwhJPSJS9xkN6Zic0RwT0ruQ7%2FROVhucra%2BdcqxdX7lyqUZ1bVskmmxTbnqZ4XwL7vH%2BNzKbfm3Z%2BWfkrqUnhbIS6oalx1Fhquod7PTRaPUmLQfacdFSWom6viXn57EbX4Ic0nmCiLsI0FSuMwiL8a2Kq8XtpC4XjiRe7IW3RhJg4x6u8YoOctXfHNh8CF%2FZgBrk%2BtVZuRCFzET%2BdC1rwsP%2F2jt%2BN8Ke0TkT4u%2BiYZXD6PO0EzjfPBtq5iE5HES%2FSTWvw%2By8a%2FG0n58j8C3ncTf8aFW6pYzzHS%2FrQX%2FNdMImcrNIGOqUBgR7k9jfQJHtLyjnfVwkMdLEsNp4xBzN%2Bu0rzBkO259Cnlq2KiI3Rl%2BkF1qASgpqzaZQ6e93FkmWYlpAo%2BYdKDUKDGWLWlSDs8BtBMWYK4PumpOz31WVpEpSrbRa8uBs0L9LcQfX%2FZbf%2BmI%2FDgvqvaaqIhYZ5qMNWSNJPC3VHL%2BR000wcHTG%2BQ4ph3DVCBpo4rPiCtq4ZVIEo88TCfgf8IqNvN6Fn&X-Amz-Signature=3c201a03db9cf1adebf05ba1fd4a9209f441006c7cfdc8ef39de8a1497a1c320&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
  "expiresAt": "2026-07-06T02:18:14.377Z"
}
```

### gen-e652baf8（再生成 A4）

```json
{
  "url": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/generated-images/bubble-test-user/gen-e652baf8-2268-41e7-809b-f7134f0a40cd/output.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3IFUVHKE2%2F20260706%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260706T021116Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEIL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLW5vcnRoZWFzdC0xIkUwQwIgARnZbhAiWLRlIkK9Z5Vw9gSNUvJ2L8MuuryUrx%2BPNicCH3InLp7b5rH1ShoPjxo%2FDQVRtC0mt%2Fn0kdeqErEyXggq9wQISxAAGgwzMzI4OTY5Mzg2MTQiDDeHJMs3g%2F9yi4qi1CrUBFlTmv%2B5YhzZ4siwoEBguN1zX96uczA54mFEtr91cgXnwNHHN6HrGhQNyPNB8FfnHItE8chd9byQ4A4AFDC2X45H%2FWRNCAJFgQMIwnfBFnF2MfneiHCHS%2BNZpjJ4m2UKYf98tyIav52w4an6gsd6vG0pkuPrIrxPb3nPPBCx4e1vM3zwdg58KjWk305hPr1npvSlyJfONsziOQNZUbQJ%2BK4DMBmITYNUDSQAiXjcBBIfcg8%2BRItOZ3VaiENfKDLN2sLspzKGbEYpcyefSCbKD7TiGO3ps320cC4CrmdnLpD8mrARHdzksp6kR%2BwCO64zDwhL67tQxAgZZKETtP6YEsVjRN1T6JmWUZ2B3r0H6Z4SOIvFnOgq5ND2mEv%2FS54QJnkGBwdSjpn4tKRtSXpo%2BSerMUzI5c8NrWdxwEVZFZy4%2BN28UhtkMzAeHal0VLdRLsDP4lHz%2FHLlTwRj8YiQTRwTCoesJ%2BtbJU%2FhrzGirSLNNF8Gbw5qwhJPSJS9xkN6Zic0RwT0ruQ7%2FROVhucra%2BdcqxdX7lyqUZ1bVskmmxTbnqZ4XwL7vH%2BNzKbfm3Z%2BWfkrqUnhbIS6oalx1Fhquod7PTRaPUmLQfacdFSWom6viXn57EbX4Ic0nmCiLsI0FSuMwiL8a2Kq8XtpC4XjiRe7IW3RhJg4x6u8YoOctXfHNh8CF%2FZgBrk%2BtVZuRCFzET%2BdC1rwsP%2F2jt%2BN8Ke0TkT4u%2BiYZXD6PO0EzjfPBtq5iE5HES%2FSTWvw%2By8a%2FG0n58j8C3ncTf8aFW6pYzzHS%2FrQX%2FNdMImcrNIGOqUBgR7k9jfQJHtLyjnfVwkMdLEsNp4xBzN%2Bu0rzBkO259Cnlq2KiI3Rl%2BkF1qASgpqzaZQ6e93FkmWYlpAo%2BYdKDUKDGWLWlSDs8BtBMWYK4PumpOz31WVpEpSrbRa8uBs0L9LcQfX%2FZbf%2BmI%2FDgvqvaaqIhYZ5qMNWSNJPC3VHL%2BR000wcHTG%2BQ4ph3DVCBpo4rPiCtq4ZVIEo88TCfgf8IqNvN6Fn&X-Amz-Signature=9ab44904efa9f70706575cb8a026b77c21d05e8f31e06358e0f9b984207b4daa&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
  "expiresAt": "2026-07-06T02:21:16.576Z"
}
```

### gen-7013514e（再生成 A3 sig3）

```json
{
  "url": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/generated-images/bubble-test-user/gen-7013514e-543e-4e4a-9f4a-7cc2d7c7805b/output.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3IFUVHKE2%2F20260706%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260706T021118Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEIL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLW5vcnRoZWFzdC0xIkUwQwIgARnZbhAiWLRlIkK9Z5Vw9gSNUvJ2L8MuuryUrx%2BPNicCH3InLp7b5rH1ShoPjxo%2FDQVRtC0mt%2Fn0kdeqErEyXggq9wQISxAAGgwzMzI4OTY5Mzg2MTQiDDeHJMs3g%2F9yi4qi1CrUBFlTmv%2B5YhzZ4siwoEBguN1zX96uczA54mFEtr91cgXnwNHHN6HrGhQNyPNB8FfnHItE8chd9byQ4A4AFDC2X45H%2FWRNCAJFgQMIwnfBFnF2MfneiHCHS%2BNZpjJ4m2UKYf98tyIav52w4an6gsd6vG0pkuPrIrxPb3nPPBCx4e1vM3zwdg58KjWk305hPr1npvSlyJfONsziOQNZUbQJ%2BK4DMBmITYNUDSQAiXjcBBIfcg8%2BRItOZ3VaiENfKDLN2sLspzKGbEYpcyefSCbKD7TiGO3ps320cC4CrmdnLpD8mrARHdzksp6kR%2BwCO64zDwhL67tQxAgZZKETtP6YEsVjRN1T6JmWUZ2B3r0H6Z4SOIvFnOgq5ND2mEv%2FS54QJnkGBwdSjpn4tKRtSXpo%2BSerMUzI5c8NrWdxwEVZFZy4%2BN28UhtkMzAeHal0VLdRLsDP4lHz%2FHLlTwRj8YiQTRwTCoesJ%2BtbJU%2FhrzGirSLNNF8Gbw5qwhJPSJS9xkN6Zic0RwT0ruQ7%2FROVhucra%2BdcqxdX7lyqUZ1bVskmmxTbnqZ4XwL7vH%2BNzKbfm3Z%2BWfkrqUnhbIS6oalx1Fhquod7PTRaPUmLQfacdFSWom6viXn57EbX4Ic0nmCiLsI0FSuMwiL8a2Kq8XtpC4XjiRe7IW3RhJg4x6u8YoOctXfHNh8CF%2FZgBrk%2BtVZuRCFzET%2BdC1rwsP%2F2jt%2BN8Ke0TkT4u%2BiYZXD6PO0EzjfPBtq5iE5HES%2FSTWvw%2By8a%2FG0n58j8C3ncTf8aFW6pYzzHS%2FrQX%2FNdMImcrNIGOqUBgR7k9jfQJHtLyjnfVwkMdLEsNp4xBzN%2Bu0rzBkO259Cnlq2KiI3Rl%2BkF1qASgpqzaZQ6e93FkmWYlpAo%2BYdKDUKDGWLWlSDs8BtBMWYK4PumpOz31WVpEpSrbRa8uBs0L9LcQfX%2FZbf%2BmI%2FDgvqvaaqIhYZ5qMNWSNJPC3VHL%2BR000wcHTG%2BQ4ph3DVCBpo4rPiCtq4ZVIEo88TCfgf8IqNvN6Fn&X-Amz-Signature=df1ce2657b0eac4d58640f6825f0d4b45b5bcc085b82cd31ea2badc62464ef6c&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
  "expiresAt": "2026-07-06T02:21:18.011Z"
}
```

### gen-59c52e75（初回 長尺横 urabon1）

```json
{
  "url": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/generated-images/bubble-test-user/gen-59c52e75-8efc-4d9f-a7c3-c253a3299594/output.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3IFUVHKE2%2F20260706%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260706T021118Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEIL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLW5vcnRoZWFzdC0xIkUwQwIgARnZbhAiWLRlIkK9Z5Vw9gSNUvJ2L8MuuryUrx%2BPNicCH3InLp7b5rH1ShoPjxo%2FDQVRtC0mt%2Fn0kdeqErEyXggq9wQISxAAGgwzMzI4OTY5Mzg2MTQiDDeHJMs3g%2F9yi4qi1CrUBFlTmv%2B5YhzZ4siwoEBguN1zX96uczA54mFEtr91cgXnwNHHN6HrGhQNyPNB8FfnHItE8chd9byQ4A4AFDC2X45H%2FWRNCAJFgQMIwnfBFnF2MfneiHCHS%2BNZpjJ4m2UKYf98tyIav52w4an6gsd6vG0pkuPrIrxPb3nPPBCx4e1vM3zwdg58KjWk305hPr1npvSlyJfONsziOQNZUbQJ%2BK4DMBmITYNUDSQAiXjcBBIfcg8%2BRItOZ3VaiENfKDLN2sLspzKGbEYpcyefSCbKD7TiGO3ps320cC4CrmdnLpD8mrARHdzksp6kR%2BwCO64zDwhL67tQxAgZZKETtP6YEsVjRN1T6JmWUZ2B3r0H6Z4SOIvFnOgq5ND2mEv%2FS54QJnkGBwdSjpn4tKRtSXpo%2BSerMUzI5c8NrWdxwEVZFZy4%2BN28UhtkMzAeHal0VLdRLsDP4lHz%2FHLlTwRj8YiQTRwTCoesJ%2BtbJU%2FhrzGirSLNNF8Gbw5qwhJPSJS9xkN6Zic0RwT0ruQ7%2FROVhucra%2BdcqxdX7lyqUZ1bVskmmxTbnqZ4XwL7vH%2BNzKbfm3Z%2BWfkrqUnhbIS6oalx1Fhquod7PTRaPUmLQfacdFSWom6viXn57EbX4Ic0nmCiLsI0FSuMwiL8a2Kq8XtpC4XjiRe7IW3RhJg4x6u8YoOctXfHNh8CF%2FZgBrk%2BtVZuRCFzET%2BdC1rwsP%2F2jt%2BN8Ke0TkT4u%2BiYZXD6PO0EzjfPBtq5iE5HES%2FSTWvw%2By8a%2FG0n58j8C3ncTf8aFW6pYzzHS%2FrQX%2FNdMImcrNIGOqUBgR7k9jfQJHtLyjnfVwkMdLEsNp4xBzN%2Bu0rzBkO259Cnlq2KiI3Rl%2BkF1qASgpqzaZQ6e93FkmWYlpAo%2BYdKDUKDGWLWlSDs8BtBMWYK4PumpOz31WVpEpSrbRa8uBs0L9LcQfX%2FZbf%2BmI%2FDgvqvaaqIhYZ5qMNWSNJPC3VHL%2BR000wcHTG%2BQ4ph3DVCBpo4rPiCtq4ZVIEo88TCfgf8IqNvN6Fn&X-Amz-Signature=69ec13114fefe99174dd19bf5a016ba5142a9e5a662df504511a5162efa588ae&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
  "expiresAt": "2026-07-06T02:21:18.771Z"
}
```

### gen-ac8ff65d（初回 A3 urabon1）

```json
{
  "url": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/generated-images/bubble-test-user/gen-ac8ff65d-cbd0-48e3-b2c0-45e2f9f770c1/output.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3IFUVHKE2%2F20260706%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260706T021334Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEIL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLW5vcnRoZWFzdC0xIkUwQwIgARnZbhAiWLRlIkK9Z5Vw9gSNUvJ2L8MuuryUrx%2BPNicCH3InLp7b5rH1ShoPjxo%2FDQVRtC0mt%2Fn0kdeqErEyXggq9wQISxAAGgwzMzI4OTY5Mzg2MTQiDDeHJMs3g%2F9yi4qi1CrUBFlTmv%2B5YhzZ4siwoEBguN1zX96uczA54mFEtr91cgXnwNHHN6HrGhQNyPNB8FfnHItE8chd9byQ4A4AFDC2X45H%2FWRNCAJFgQMIwnfBFnF2MfneiHCHS%2BNZpjJ4m2UKYf98tyIav52w4an6gsd6vG0pkuPrIrxPb3nPPBCx4e1vM3zwdg58KjWk305hPr1npvSlyJfONsziOQNZUbQJ%2BK4DMBmITYNUDSQAiXjcBBIfcg8%2BRItOZ3VaiENfKDLN2sLspzKGbEYpcyefSCbKD7TiGO3ps320cC4CrmdnLpD8mrARHdzksp6kR%2BwCO64zDwhL67tQxAgZZKETtP6YEsVjRN1T6JmWUZ2B3r0H6Z4SOIvFnOgq5ND2mEv%2FS54QJnkGBwdSjpn4tKRtSXpo%2BSerMUzI5c8NrWdxwEVZFZy4%2BN28UhtkMzAeHal0VLdRLsDP4lHz%2FHLlTwRj8YiQTRwTCoesJ%2BtbJU%2FhrzGirSLNNF8Gbw5qwhJPSJS9xkN6Zic0RwT0ruQ7%2FROVhucra%2BdcqxdX7lyqUZ1bVskmmxTbnqZ4XwL7vH%2BNzKbfm3Z%2BWfkrqUnhbIS6oalx1Fhquod7PTRaPUmLQfacdFSWom6viXn57EbX4Ic0nmCiLsI0FSuMwiL8a2Kq8XtpC4XjiRe7IW3RhJg4x6u8YoOctXfHNh8CF%2FZgBrk%2BtVZuRCFzET%2BdC1rwsP%2F2jt%2BN8Ke0TkT4u%2BiYZXD6PO0EzjfPBtq5iE5HES%2FSTWvw%2By8a%2FG0n58j8C3ncTf8aFW6pYzzHS%2FrQX%2FNdMImcrNIGOqUBgR7k9jfQJHtLyjnfVwkMdLEsNp4xBzN%2Bu0rzBkO259Cnlq2KiI3Rl%2BkF1qASgpqzaZQ6e93FkmWYlpAo%2BYdKDUKDGWLWlSDs8BtBMWYK4PumpOz31WVpEpSrbRa8uBs0L9LcQfX%2FZbf%2BmI%2FDgvqvaaqIhYZ5qMNWSNJPC3VHL%2BR000wcHTG%2BQ4ph3DVCBpo4rPiCtq4ZVIEo88TCfgf8IqNvN6Fn&X-Amz-Signature=0a64bb6d38795f692bce579a32ba8402fe1712f2609d29d04196e4c9d5050602&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
  "expiresAt": "2026-07-06T02:23:34.152Z"
}
```

### gen-cbbd64cc（初回 ハガキ urabon1）

```json
{
  "url": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/generated-images/bubble-test-user/gen-cbbd64cc-6ec4-4fa2-9509-b6540526e702/output.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3IFUVHKE2%2F20260706%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260706T021334Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEIL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLW5vcnRoZWFzdC0xIkUwQwIgARnZbhAiWLRlIkK9Z5Vw9gSNUvJ2L8MuuryUrx%2BPNicCH3InLp7b5rH1ShoPjxo%2FDQVRtC0mt%2Fn0kdeqErEyXggq9wQISxAAGgwzMzI4OTY5Mzg2MTQiDDeHJMs3g%2F9yi4qi1CrUBFlTmv%2B5YhzZ4siwoEBguN1zX96uczA54mFEtr91cgXnwNHHN6HrGhQNyPNB8FfnHItE8chd9byQ4A4AFDC2X45H%2FWRNCAJFgQMIwnfBFnF2MfneiHCHS%2BNZpjJ4m2UKYf98tyIav52w4an6gsd6vG0pkuPrIrxPb3nPPBCx4e1vM3zwdg58KjWk305hPr1npvSlyJfONsziOQNZUbQJ%2BK4DMBmITYNUDSQAiXjcBBIfcg8%2BRItOZ3VaiENfKDLN2sLspzKGbEYpcyefSCbKD7TiGO3ps320cC4CrmdnLpD8mrARHdzksp6kR%2BwCO64zDwhL67tQxAgZZKETtP6YEsVjRN1T6JmWUZ2B3r0H6Z4SOIvFnOgq5ND2mEv%2FS54QJnkGBwdSjpn4tKRtSXpo%2BSerMUzI5c8NrWdxwEVZFZy4%2BN28UhtkMzAeHal0VLdRLsDP4lHz%2FHLlTwRj8YiQTRwTCoesJ%2BtbJU%2FhrzGirSLNNF8Gbw5qwhJPSJS9xkN6Zic0RwT0ruQ7%2FROVhucra%2BdcqxdX7lyqUZ1bVskmmxTbnqZ4XwL7vH%2BNzKbfm3Z%2BWfkrqUnhbIS6oalx1Fhquod7PTRaPUmLQfacdFSWom6viXn57EbX4Ic0nmCiLsI0FSuMwiL8a2Kq8XtpC4XjiRe7IW3RhJg4x6u8YoOctXfHNh8CF%2FZgBrk%2BtVZuRCFzET%2BdC1rwsP%2F2jt%2BN8Ke0TkT4u%2BiYZXD6PO0EzjfPBtq5iE5HES%2FSTWvw%2By8a%2FG0n58j8C3ncTf8aFW6pYzzHS%2FrQX%2FNdMImcrNIGOqUBgR7k9jfQJHtLyjnfVwkMdLEsNp4xBzN%2Bu0rzBkO259Cnlq2KiI3Rl%2BkF1qASgpqzaZQ6e93FkmWYlpAo%2BYdKDUKDGWLWlSDs8BtBMWYK4PumpOz31WVpEpSrbRa8uBs0L9LcQfX%2FZbf%2BmI%2FDgvqvaaqIhYZ5qMNWSNJPC3VHL%2BR000wcHTG%2BQ4ph3DVCBpo4rPiCtq4ZVIEo88TCfgf8IqNvN6Fn&X-Amz-Signature=f3c1c64f7c8e02fa64ecbda7f7f0dd0882fd46cd7d25b81080db03b0dfea1664&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
  "expiresAt": "2026-07-06T02:23:34.846Z"
}
```

### gen-add6a20f（再生成 長尺縦）

```json
{
  "url": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/generated-images/bubble-test-user/gen-add6a20f-8e00-4fd9-9869-634d60aacb67/output.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3IFUVHKE2%2F20260706%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260706T021529Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEIL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLW5vcnRoZWFzdC0xIkUwQwIgARnZbhAiWLRlIkK9Z5Vw9gSNUvJ2L8MuuryUrx%2BPNicCH3InLp7b5rH1ShoPjxo%2FDQVRtC0mt%2Fn0kdeqErEyXggq9wQISxAAGgwzMzI4OTY5Mzg2MTQiDDeHJMs3g%2F9yi4qi1CrUBFlTmv%2B5YhzZ4siwoEBguN1zX96uczA54mFEtr91cgXnwNHHN6HrGhQNyPNB8FfnHItE8chd9byQ4A4AFDC2X45H%2FWRNCAJFgQMIwnfBFnF2MfneiHCHS%2BNZpjJ4m2UKYf98tyIav52w4an6gsd6vG0pkuPrIrxPb3nPPBCx4e1vM3zwdg58KjWk305hPr1npvSlyJfONsziOQNZUbQJ%2BK4DMBmITYNUDSQAiXjcBBIfcg8%2BRItOZ3VaiENfKDLN2sLspzKGbEYpcyefSCbKD7TiGO3ps320cC4CrmdnLpD8mrARHdzksp6kR%2BwCO64zDwhL67tQxAgZZKETtP6YEsVjRN1T6JmWUZ2B3r0H6Z4SOIvFnOgq5ND2mEv%2FS54QJnkGBwdSjpn4tKRtSXpo%2BSerMUzI5c8NrWdxwEVZFZy4%2BN28UhtkMzAeHal0VLdRLsDP4lHz%2FHLlTwRj8YiQTRwTCoesJ%2BtbJU%2FhrzGirSLNNF8Gbw5qwhJPSJS9xkN6Zic0RwT0ruQ7%2FROVhucra%2BdcqxdX7lyqUZ1bVskmmxTbnqZ4XwL7vH%2BNzKbfm3Z%2BWfkrqUnhbIS6oalx1Fhquod7PTRaPUmLQfacdFSWom6viXn57EbX4Ic0nmCiLsI0FSuMwiL8a2Kq8XtpC4XjiRe7IW3RhJg4x6u8YoOctXfHNh8CF%2FZgBrk%2BtVZuRCFzET%2BdC1rwsP%2F2jt%2BN8Ke0TkT4u%2BiYZXD6PO0EzjfPBtq5iE5HES%2FSTWvw%2By8a%2FG0n58j8C3ncTf8aFW6pYzzHS%2FrQX%2FNdMImcrNIGOqUBgR7k9jfQJHtLyjnfVwkMdLEsNp4xBzN%2Bu0rzBkO259Cnlq2KiI3Rl%2BkF1qASgpqzaZQ6e93FkmWYlpAo%2BYdKDUKDGWLWlSDs8BtBMWYK4PumpOz31WVpEpSrbRa8uBs0L9LcQfX%2FZbf%2BmI%2FDgvqvaaqIhYZ5qMNWSNJPC3VHL%2BR000wcHTG%2BQ4ph3DVCBpo4rPiCtq4ZVIEo88TCfgf8IqNvN6Fn&X-Amz-Signature=fed0d89e67ad8dbbe64a9d5d70c3e9ec77a4c89c08ee0e1f4d6d41b4fee58f6f&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
  "expiresAt": "2026-07-06T02:25:29.676Z"
}
```

### gen-fa32c8b2（再生成 A3 urabon1）

```json
{
  "url": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/generated-images/bubble-test-user/gen-fa32c8b2-5f08-4177-995a-5feb6f619d21/output.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3IFUVHKE2%2F20260706%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260706T021530Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEIL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLW5vcnRoZWFzdC0xIkUwQwIgARnZbhAiWLRlIkK9Z5Vw9gSNUvJ2L8MuuryUrx%2BPNicCH3InLp7b5rH1ShoPjxo%2FDQVRtC0mt%2Fn0kdeqErEyXggq9wQISxAAGgwzMzI4OTY5Mzg2MTQiDDeHJMs3g%2F9yi4qi1CrUBFlTmv%2B5YhzZ4siwoEBguN1zX96uczA54mFEtr91cgXnwNHHN6HrGhQNyPNB8FfnHItE8chd9byQ4A4AFDC2X45H%2FWRNCAJFgQMIwnfBFnF2MfneiHCHS%2BNZpjJ4m2UKYf98tyIav52w4an6gsd6vG0pkuPrIrxPb3nPPBCx4e1vM3zwdg58KjWk305hPr1npvSlyJfONsziOQNZUbQJ%2BK4DMBmITYNUDSQAiXjcBBIfcg8%2BRItOZ3VaiENfKDLN2sLspzKGbEYpcyefSCbKD7TiGO3ps320cC4CrmdnLpD8mrARHdzksp6kR%2BwCO64zDwhL67tQxAgZZKETtP6YEsVjRN1T6JmWUZ2B3r0H6Z4SOIvFnOgq5ND2mEv%2FS54QJnkGBwdSjpn4tKRtSXpo%2BSerMUzI5c8NrWdxwEVZFZy4%2BN28UhtkMzAeHal0VLdRLsDP4lHz%2FHLlTwRj8YiQTRwTCoesJ%2BtbJU%2FhrzGirSLNNF8Gbw5qwhJPSJS9xkN6Zic0RwT0ruQ7%2FROVhucra%2BdcqxdX7lyqUZ1bVskmmxTbnqZ4XwL7vH%2BNzKbfm3Z%2BWfkrqUnhbIS6oalx1Fhquod7PTRaPUmLQfacdFSWom6viXn57EbX4Ic0nmCiLsI0FSuMwiL8a2Kq8XtpC4XjiRe7IW3RhJg4x6u8YoOctXfHNh8CF%2FZgBrk%2BtVZuRCFzET%2BdC1rwsP%2F2jt%2BN8Ke0TkT4u%2BiYZXD6PO0EzjfPBtq5iE5HES%2FSTWvw%2By8a%2FG0n58j8C3ncTf8aFW6pYzzHS%2FrQX%2FNdMImcrNIGOqUBgR7k9jfQJHtLyjnfVwkMdLEsNp4xBzN%2Bu0rzBkO259Cnlq2KiI3Rl%2BkF1qASgpqzaZQ6e93FkmWYlpAo%2BYdKDUKDGWLWlSDs8BtBMWYK4PumpOz31WVpEpSrbRa8uBs0L9LcQfX%2FZbf%2BmI%2FDgvqvaaqIhYZ5qMNWSNJPC3VHL%2BR000wcHTG%2BQ4ph3DVCBpo4rPiCtq4ZVIEo88TCfgf8IqNvN6Fn&X-Amz-Signature=ae8e6b99e73843786b708e4272cadba0698a930cf42d5ee0b9fe4a32f85b3eee&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
  "expiresAt": "2026-07-06T02:25:30.334Z"
}
```

### 生成画像サムネイル（CloudFront・恒久URL）

| ジョブ | thumbnailUrl |
|---|---|
| gen-44ac1366（初回A4） | https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-44ac1366-e147-454c-b741-7f864ffcc7b3/thumbnail.png |
| gen-4d9757c1（初回A3 sig3） | https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-4d9757c1-d9fa-4e14-ad9b-059a4c95ec1f/thumbnail.png |
| gen-feb6f18b（初回長尺縦 sig3） | https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-feb6f18b-7d8c-481a-9d81-705dfaa72935/thumbnail.png |
| gen-b1a822cb（初回封筒） | https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-b1a822cb-55a5-4c83-a69b-904a07cfd0c6/thumbnail.png |
| gen-5ee17a58（初回長尺縦 制約外） | https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-5ee17a58-98de-4e7d-8821-4c2d919f1342/thumbnail.png |
| gen-e652baf8（再生成A4） | https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-e652baf8-2268-41e7-809b-f7134f0a40cd/thumbnail.png |
| gen-7013514e（再生成A3 sig3） | https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-7013514e-543e-4e4a-9f4a-7cc2d7c7805b/thumbnail.png |
| gen-59c52e75（初回長尺横） | https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-59c52e75-8efc-4d9f-a7c3-c253a3299594/thumbnail.png |
| gen-ac8ff65d（初回A3 urabon1） | https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-ac8ff65d-cbd0-48e3-b2c0-45e2f9f770c1/thumbnail.png |
| gen-cbbd64cc（初回ハガキ） | https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-cbbd64cc-6ec4-4fa2-9509-b6540526e702/thumbnail.png |
| gen-add6a20f（再生成長尺縦） | https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-add6a20f-8e00-4fd9-9869-634d60aacb67/thumbnail.png |
| gen-fa32c8b2（再生成A3 urabon1） | https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-fa32c8b2-5f08-4177-995a-5feb6f619d21/thumbnail.png |

---

## 追記1（2026-07-08）: API Key認証への切替と追加テスト

### API Key認証への切替

2026-07-08時点で、全エンドポイントがAPI Key認証必須に変更されていることを確認（API Keyなしは `403 ForbiddenException`）。認証方式は `API認証方式/bubble_api_key_and_waf_ip_whitelist_guide.md` 参照（`x-api-key` ヘッダーで送信）。

```
API Keyなし: GET /source-image-groups → HTTP 403 {"message":"Forbidden"}
API Keyあり: GET /source-image-groups → HTTP 200
```

### 追加テスト: 描画項目マトリクスの「○」網羅確認

`01_functional/サイズ別描画項目マトリクス.md`（仕様_行事テンプレート_共通印刷.xlsx「行事について」シート）の受領を受け、前回未確認だった組み合わせを検証。

| No | 種別 | サイズ（グループ） | sourceImageId | generatedImageId | 結果 |
|---|---|---|---|---|---|
| 18 | 初回 | ハガキ・全7項目（urabon1） | src-385b23c9-5b69-4bc1-9615-7964f9db4b64 | gen-5e0461c8-fb23-4885-a296-aa2ba6b4337b | COMPLETED（文言混入NG） |
| 19 | 初回 | 封筒 長形3号・期間+連絡先詳細あり（urabon1） | src-cf4d527e-a233-4f51-84c7-10a223e1ce0a | gen-e852340b-d983-4d0d-b0c8-3b4dfc63f438 | COMPLETED（描画不足NG） |

リクエスト（No.18 ハガキ、全7項目）:

```json
{
  "sourceImageId": "src-385b23c9-5b69-4bc1-9615-7964f9db4b64",
  "eventName": "ブラザー夏祭り盆踊り会",
  "eventPeriod": "2026年8月23日",
  "eventDetails": "境内にて夏祭りを開催いたします",
  "eventProgram": "10:00 開会 / 12:00 盆踊り / 15:00 閉会",
  "notes": "雨天時は中止となります",
  "contactName": "ブラザー工業 総務部 山田",
  "contactDetails": "TEL 052-123-4567（平日9:00〜17:00）"
}
```

リクエスト（No.19 封筒、マトリクスで表面○の項目のみ）:

```json
{
  "sourceImageId": "src-cf4d527e-a233-4f51-84c7-10a223e1ce0a",
  "eventName": "ブラザー夏祭り盆踊り会",
  "eventPeriod": "2026年8月23日",
  "contactName": "ブラザー工業 総務部 山田",
  "contactDetails": "TEL 052-123-4567（平日9:00〜17:00）"
}
```

### 結果と新規事象

| ファイル | 実寸 | 目視確認結果 |
|---|---|---|
| 01_初回生成_印刷物/初回_ハガキ_全項目.png | 832×1280 | **NG（確認事項G）**: 全7項目は描画されたが、リクエストにない文言「兄弟舎主宰 堀田 太郎 師 演題:「つながる命、感謝の心」」がプログラム欄の直後に混入。前回issue（事象1）でA4再生成時に混入したものと同一内容の文言が、**urabon1ソースの初回生成**で再発 |
| 01_初回生成_印刷物/初回_封筒長形3号_期間連絡先詳細.png | 736×1440 | 開催期間・連絡先名・連絡先詳細は非描画（当初Hとして起票したが、7/2 Q&Aの仕様変更「封筒は行事名のみ記載」どおりで**問題なし**と判明）。ただしeventNameが「ブラザー夏祭り会」と描画され「盆踊り」が欠落（文言精度の問題として確認事項Gの枠） |

- **確認事項G（重要度: 高）**: 「盂蘭盆会法要」「堀田太郎師」系のテンプレ文言混入は、**urabon1のハガキ初回生成でも発生**する。事象1は「再生成固有」ではなく生成全般で稀発する精度問題（確認事項Dと同じくBubble側回答で検討中の枠）とみられる。発生条件の特定・抑止をブラザー側に依頼したい。
- **確認事項H → 解消（仕様どおり）**: 封筒(長形3号)で開催期間・連絡先名・連絡先詳細が描画されない件は、当初マトリクスの読み違い（表:○/裏:N/Aと解釈）により不具合として起票したが、**2026-07-02 Q&Aで「PoCでは封筒は行事名のみ記載」に仕様変更済み**であり、実測どおりの挙動が正。なお、封筒はマトリクス上N/Aの項目（eventPeriod / contactName / contactDetails）をリクエストに含めない実装とする。eventNameの「盆踊り」欠落のみ、文言精度の問題（確認事項G）として残る。

### エビデンス: ファイルダウンロードURL取得レスポンス全文（追加分）

#### gen-5e0461c8（初回 ハガキ 全項目）

```json
{"url":"https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/generated-images/bubble-test-user/gen-5e0461c8-fb23-4885-a296-aa2ba6b4337b/output.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3JTHEQ2JL%2F20260707%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260707T235345Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjELD%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLW5vcnRoZWFzdC0xIkYwRAIgDPbvKZDG0mQ8aOvesRXjjMtpUa82%2FMDxz0QEVYQkI5MCIH9T3JdZZ9x%2BMLuNFaO9GuTpwG5CF1mu7qmjFNy%2FkBWeKvcECHkQABoMMzMyODk2OTM4NjE0IgyN5WE0OBu5NQZSrscq1ARgsYFehF89mlR0SzrYUzsNZa4dCreT8RxE6%2FiMMe%2FAt8LXPvB5Q2I3MzO6TYmXcpGT6JTk0Re973Ka0ZVUq73nRAtNZwkIycNdH6zj1E4ggopdUSqNc0ShgPxwn0oHR5BUPt75HXDmpOz00c3abUnpydNDuwWjyd4VVbFaSXqLhr0iK7%2BSOg0sSfHoG7S%2Baoucq0pdyl5tOognVu9SyJWEFinWLkbdHRwXPHJARRTHQoALIA3XsbeWohb4uVsArpSi81mGDnltA4QZcZWvUlHi1u2Q%2BPpqCPIgcfWU7J0t2XYDf41ACr6K%2Fz1QhQSjFu5kvga3te6bMzo6NBeu%2B876GV%2B7B4Qaf5Q7Fqn89OpoRPp35H9dcv35CD068Wyulk4UEBedYJKfxITk1Fh8IMnmOYAfu7%2BgA8A9Qvf3RInPENw%2FYqfxlNG04b02rq8mGEgxcNfMpJc6NUC2%2F5%2Fz5LCap%2BGRdnHOgS1a%2BlcRr4XLbyQXxxXcBR%2Fkn0FaF%2FbBkIXibkeqgHAqXtWxG5qyZhf9CUuky3%2FFhA%2B2jzi9zR0oFPVtv2gByF78KynBNIvtDj9SZMlSSLz0i9PqtXvmZgEHqZxTy1SSvIlBWk0uBfGa42Jm90qotT6Y4wUlpGwVx0klb40y6QsP%2FnXcPGs6HDWxHPkmfuNeQn2WlaGdVa7exIpShzl8REmjwWHW%2BctvJFPs8TxTQnfam5cK22LUPDKraj0LX3Q2yfpFwVCn%2BenTDU08QSeAEu9ZXPResNSRnFTsEKijz%2B0BI%2FylnuiOMp9Vp7MSKTCIo7bSBjqkAWbMYe7m94fLNqbxoyo%2Fgaj%2BHpWF%2Bdhp8DvbYzhXIi2azOebaYMkHLZ854sxAFauuf7NI50da5qjpJeHvJYFaJtEUUvhqporSrxsxusJSz0E6aTwfUtINRqltW%2BQmGRup5kTF2PbkxuvI4w3MQghotNhKuZKWkXZIuUkTB22WndXgmy6A29ducjElRt1X1LVxkLRLdyctZI%2BIWK4YvSTx4%2BT2pfO&X-Amz-Signature=fc9f429f131e04817cf28069d759f612cd8d766aa6a2de7ed4f483e3f5c96b95&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject","expiresAt":"2026-07-08T00:03:45.597Z"}
```

#### gen-e852340b（初回 封筒 長形3号 期間+連絡先詳細）

```json
{"url":"https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/generated-images/bubble-test-user/gen-e852340b-d983-4d0d-b0c8-3b4dfc63f438/output.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3JTHEQ2JL%2F20260707%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260707T235346Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjELD%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLW5vcnRoZWFzdC0xIkYwRAIgDPbvKZDG0mQ8aOvesRXjjMtpUa82%2FMDxz0QEVYQkI5MCIH9T3JdZZ9x%2BMLuNFaO9GuTpwG5CF1mu7qmjFNy%2FkBWeKvcECHkQABoMMzMyODk2OTM4NjE0IgyN5WE0OBu5NQZSrscq1ARgsYFehF89mlR0SzrYUzsNZa4dCreT8RxE6%2FiMMe%2FAt8LXPvB5Q2I3MzO6TYmXcpGT6JTk0Re973Ka0ZVUq73nRAtNZwkIycNdH6zj1E4ggopdUSqNc0ShgPxwn0oHR5BUPt75HXDmpOz00c3abUnpydNDuwWjyd4VVbFaSXqLhr0iK7%2BSOg0sSfHoG7S%2Baoucq0pdyl5tOognVu9SyJWEFinWLkbdHRwXPHJARRTHQoALIA3XsbeWohb4uVsArpSi81mGDnltA4QZcZWvUlHi1u2Q%2BPpqCPIgcfWU7J0t2XYDf41ACr6K%2Fz1QhQSjFu5kvga3te6bMzo6NBeu%2B876GV%2B7B4Qaf5Q7Fqn89OpoRPp35H9dcv35CD068Wyulk4UEBedYJKfxITk1Fh8IMnmOYAfu7%2BgA8A9Qvf3RInPENw%2FYqfxlNG04b02rq8mGEgxcNfMpJc6NUC2%2F5%2Fz5LCap%2BGRdnHOgS1a%2BlcRr4XLbyQXxxXcBR%2Fkn0FaF%2FbBkIXibkeqgHAqXtWxG5qyZhf9CUuky3%2FFhA%2B2jzi9zR0oFPVtv2gByF78KynBNIvtDj9SZMlSSLz0i9PqtXvmZgEHqZxTy1SSvIlBWk0uBfGa42Jm90qotT6Y4wUlpGwVx0klb40y6QsP%2FnXcPGs6HDWxHPkmfuNeQn2WlaGdVa7exIpShzl8REmjwWHW%2BctvJFPs8TxTQnfam5cK22LUPDKraj0LX3Q2yfpFwVCn%2BenTDU08QSeAEu9ZXPResNSRnFTsEKijz%2B0BI%2FylnuiOMp9Vp7MSKTCIo7bSBjqkAWbMYe7m94fLNqbxoyo%2Fgaj%2BHpWF%2Bdhp8DvbYzhXIi2azOebaYMkHLZ854sxAFauuf7NI50da5qjpJeHvJYFaJtEUUvhqporSrxsxusJSz0E6aTwfUtINRqltW%2BQmGRup5kTF2PbkxuvI4w3MQghotNhKuZKWkXZIuUkTB22WndXgmy6A29ducjElRt1X1LVxkLRLdyctZI%2BIWK4YvSTx4%2BT2pfO&X-Amz-Signature=f34785e82864a7cc71fd55ac7c867cc1cf0a1e22748df70a511640cc4fbe66ba&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject","expiresAt":"2026-07-08T00:03:46.210Z"}
```

#### 生成画像サムネイル（CloudFront・恒久URL、追加分）

| ジョブ | thumbnailUrl |
|---|---|
| gen-5e0461c8（初回ハガキ全項目） | https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-5e0461c8-fb23-4885-a296-aa2ba6b4337b/thumbnail.png |
| gen-e852340b（初回封筒 期間+連絡先詳細） | https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-e852340b-d983-4d0d-b0c8-3b4dfc63f438/thumbnail.png |

---

## 追記2（2026-07-08）: sig1/sig3削除とソースイメージグループ入れ替えの確認

宮田さまメール（7/6）「旧グループ（sig1/sig3）は削除していきます」のとおり、7/8時点のGET /source-image-groups で**sig1/sig3が削除済み**であることを確認した。事象3の不備ソース（src-b9bfb2fc、実画像1123×3402）もこれにより消滅。

現在のグループは行事4種 × デザインタイプ3件の計12グループ:

| 行事 | グループ |
|---|---|
| 盂蘭盆 | urabon1 / urabon2 / urabon3 |
| 十夜 | jyuya1 / jyuya2 / jyuya3 |
| 報恩講 | houonnkou1 / houonnkou2 / houonnkou3 |
| 秋彼岸 | akihigan1 / akihigan2 / akihigan3 |

- 仕様（`../../01_functional/サイズ別描画項目マトリクス.md`）の「1つの行事に含まれるデザインタイプは最小1件、最大3件」と整合する構成。
- サイズ登録はjyuya1 / houonnkou1 / akihigan1でサンプル確認し、いずれも7サイズ（A3 / A4 / A4便箋 / ハガキ / 封筒(長形3号) / 長尺横 / 長尺縦）が登録済み。

### GET /source-image-groups レスポンス（2026-07-08、x-api-key付与）

```json
{
  "items": [
    {
      "sourceImageGroupId": "jyuya1",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/jyuya1/src-a0a8ec9b-73a9-4332-8271-136fe6ff0987/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageGroupId": "houonnkou2",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/houonnkou2/src-a357f698-1369-47bc-94e2-3932de690f08/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageGroupId": "houonnkou3",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/houonnkou3/src-1adc83e7-ed22-4355-aafa-bb9c3aa6d42c/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageGroupId": "jyuya3",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/jyuya3/src-b5b276af-931b-4070-b40f-515096b4a95b/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageGroupId": "akihigan2",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/akihigan2/src-7e6b02c2-0c90-49ec-a107-7fe0e2b839c8/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageGroupId": "houonnkou1",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/houonnkou1/src-c6b00009-6616-404e-9529-3a083ceb6b19/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageGroupId": "akihigan1",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/akihigan1/src-3116f181-1db0-4fdb-bec5-7ccc1ffdb14e/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageGroupId": "akihigan3",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/akihigan3/src-14fcb3dd-770b-458a-8b63-1eed962774e4/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageGroupId": "urabon3",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/urabon3/src-2ac977ea-b2bf-4011-8612-8683ee555792/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageGroupId": "urabon1",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/urabon1/src-272cbd66-267f-4b5c-bd36-20747213c44f/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageGroupId": "jyuya2",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/jyuya2/src-7d7865a3-4da8-4543-b57b-4121f4c9a7e1/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageGroupId": "urabon2",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/urabon2/src-cd2a6c4a-9f11-40a7-8722-40276e6b57a1/thumbnail.png",
      "status": "active"
    }
  ]
}
```

### 残課題（7/8時点）

| # | 事象 | 状態 |
|---|---|---|
| A | 429クォータ制限 | ブラザー側検討中 |
| G | リクエストにない文言混入（urabon1ハガキ初回生成で再発） | ブラザー側検討中（生成精度の枠） |
| H | 封筒(長形3号)で開催期間・連絡先名・連絡先詳細が非描画 | 解消（7/2 Q&Aの仕様変更「封筒は行事名のみ記載」どおり。eventNameの文言欠落のみGへ） |
| - | 新グループ（jyuya / houonnkou / akihigan）での生成疎通 | 未実施（必要に応じて実施） |

