# API仕様書（ルールベース）— Bubble フロントエンド向け


## 0. 要約: Bubble の改修は不要です

- **変わらないもの**: 公開 API 9 本の URL、メソッド、リクエストのパラメータ、レスポンスのキー集合、エラー応答の形式、認証方式（x-api-key）。現在の Bubble の実装のまま動きます。
- **変わるもの**は次の 3 点だけです。

| 項目 | これまで（Gemini 版） | これから（ルールベース版） |
|---|---|---|
| 生成の精度 | 文字の欠落・残り・誤字が起こり得た | 入力したテキストがそのまま入る（テキスト精度 100%）。背景・イラストは原稿のまま変わらない |
| 生成の速度 | 30〜60 秒／枚 | 2〜4 秒／枚（dev 環境での実測） |
| 生成できる原稿 | — | 生成版グループの A4・A3・長尺縦・長尺横・ハガキ・封筒。A4便箋と旧方式（DL 版）の原稿は生成できない（§3・§5） |

- Bubble 側で念のため確認いただきたい点は §8 にまとめています（いずれも既存の挙動の範囲内で、必須の対応ではありません）。

## 1. 共通事項

| 項目 | 内容 |
|---|---|
| エンドポイント | 環境ごとのベース URL は別途共有します（本ページには記載しません）。パスは §2 のとおり |
| 認証 | リクエストヘッダー `x-api-key` が必須。無い、または誤っている場合は **403**（API Gateway の応答 `{"message":"Forbidden"}`）。**キーの値は本ページに書かず、別途共有**します |
| 形式 | リクエスト・レスポンスとも `application/json; charset=utf-8`。改行は `\n` |
| レート制限 | API キー単位で 15 リクエスト／秒、バースト 30、月 20,000 リクエスト。超過時は 429 |
| エラー応答 | `{ "errorCode": "SOURCE_IMAGE_NOT_FOUND", "message": "..." }`。分岐は `errorCode` で行い、`message` は画面表示に使わない（英文で、変わることがある） |
| ID | 原稿 ID・生成画像 ID は文字列。`src-` / `rsrc-` / `gen-` / `rgen-` などの接頭辞が付くが、**Bubble は接頭辞を解釈しなくてよい** |
| 画像の URL | `thumbnailUrl`・`downloadUrl` は期限の無い恒久 URL。`GET /files/url` が返す署名付き URL は 600 秒で失効 |
| サイズ | `A4` / `A3` / `長尺縦` / `長尺横` / `ハガキ` / `封筒(長形3号)` / `A4便箋` の 7 種（文字列そのまま） |

## 2. エンドポイント一覧（すべて変更なし）

| # | メソッド / パス | 用途 |
|---|---|---|
| 1 | GET /source-image-groups | 行事テンプレートのグループ一覧 |
| 2 | GET /source-image-groups/{groupId}/source-images | グループ内の原稿一覧（サイズ別） |
| 3 | GET /source-images/{sourceImageId} | 原稿の詳細（ダウンロード URL を含む） |
| 4 | GET /files/url/{fileKey}?filename= | 生成画像の保存用 URL（署名付き） |
| 5 | POST /generated-images | ポスター画像の生成を依頼 |
| 6 | GET /generated-images/{generatedImageId} | 生成の状態を取得（ポーリング） |
| 7 | POST /generated-images/{generatedImageId}/regenerate | 同じ原稿で文言を変えて作り直す |
| 8 | GET /common-print-groups | 共通印刷物のグループ一覧（変更なしのため本書では省略） |
| 9 | GET /common-print-groups/{groupId}/common-prints | 共通印刷物の一覧（同上） |

## 3. 原稿とグループ

### 3.1 グループ一覧（GET /source-image-groups）

- status が `active` のグループだけを返します。生成版のグループとダウンロード版（旧方式）のグループが同じ一覧に並びます。
- **グループのサムネイル（`thumbnailUrl`）は、常にそのグループの A4 の原稿のサムネイル**です。
- 今回、生成版として公開するのは **houwa_11_1（法話会_11月_親しみ）** の 1 グループです。他の行事は準備ができ次第、同じ形で追加します。

```json
200 OK
{
  "items": [
    { "sourceImageGroupId": "houwa_11_1", "thumbnailUrl": "https://…/thumbnail.jpg", "status": "active" }
  ]
}
```

### 3.2 原稿一覧（GET /source-image-groups/{groupId}/source-images）

- **1 グループ 7 サイズ**（旧方式と同じ形）で、size 順に返します。
- **A4便箋は Word のダウンロード専用**で、生成はできません。それ以外の 6 サイズが生成用です。A4便箋もサムネイル付きで一覧に並びます。
- ボタンの出し分けは、**旧方式と同じく size で判定**できます。

| size | 生成版グループでのボタン |
|---|---|
| A4 / A3 / 長尺縦 / 長尺横 / ハガキ / 封筒(長形3号) | 「作成する」（§4） |
| A4便箋 | 「ダウンロード」（§3.3。作成ボタンは出さない） |

ダウンロード版（旧方式）のグループは、従来どおり全サイズ「ダウンロード」です。

```json
200 OK
{
  "items": [
    { "sourceImageId": "rsrc-7f3a…", "size": "A4",     "thumbnailUrl": "https://…/thumbnail.jpg", "status": "active" },
    …（A3・長尺縦・長尺横・ハガキ・封筒）…
    { "sourceImageId": "rsrc-c210…", "size": "A4便箋", "thumbnailUrl": "https://…/thumbnail.jpg", "status": "active" }
  ]
}
404 { "errorCode": "NOT_FOUND", "message": "..." }   ※ グループが無い場合は空配列ではなく 404
```

### 3.3 原稿の詳細（GET /source-images/{sourceImageId}）

```json
200 OK
{
  "sourceImageId": "rsrc-c210…",
  "size": "A4便箋",
  "fileName": "法話会11月_1_A4便箋.docx",
  "thumbnailUrl": "https://…/thumbnail.jpg",
  "fileKey": "rule-source-images/houwa_11_1/rsrc-c210…/original.docx",
  "downloadUrl": "https://…/rule-source-images/houwa_11_1/rsrc-c210…/original.docx",
  "status": "active"
}
404 { "errorCode": "SOURCE_IMAGE_NOT_FOUND", "message": "Specified source image was not found." }
```

- ダウンロード用の原稿（A4便箋、旧方式の PPT/Word）は、`downloadUrl` をそのままダウンロードボタンに使います。保存名は §6 のとおり `original_<fileName>` です。
- 生成用の原稿（A4〜封筒）の `downloadUrl` は文字の無い背景画像を指します。**生成用のサイズではダウンロードボタンを出さないでください。**

### 3.4 生成画像の保存用 URL（GET /files/url/{fileKey}?filename=保存名）

```json
200 OK
{ "url": "https://…（署名付き。600 秒で失効）", "expiresAt": "2026-10-09T06:10:00.000Z" }
```

生成画像の `fileKey`（§4.2）を渡します。`filename` を付けるとその名前で保存されます。URL は 600 秒で失効するので、保存ボタンを押したときに取得してください。

## 4. 画像の生成

### 4.1 生成の依頼（POST /generated-images）

```json
POST /generated-images
{
  "sourceImageId": "rsrc-7f3a…",
  "eventName": "法話会",
  "eventPeriod": "令和8年 11月15日（日）",
  "eventProgram": "13:30 勤行\n14:00 法話",
  "contactName": "〇〇寺",
  "contactDetails": "TEL 052-000-0000"
}

200 OK
{ "generatedImageId": "rgen-2b9e…", "sourceImageId": "rsrc-7f3a…", "status": "RUNNING" }
```

| 項目 | 説明 |
|---|---|
| sourceImageId（必須） | 生成用の原稿（A4〜封筒）。生成できない原稿（A4便箋、旧方式の原稿）は 400（§5） |
| eventName / eventPeriod / eventDetails / eventProgram / contactName / contactDetails / notes | すべて任意。従来と同じ 7 項目 |

**入力の扱い（エラーにはなりません）**

| 状況 | 結果 |
|---|---|
| 項目が空、または省略 | 200。その枠は**描画しない**（空欄のまま）。残りの枠は崩れない |
| そのサイズ・デザインに無い項目を送った | 200。その項目は**無視**される |
| 最大文字数を超えた | 200。文字を**自動で縮小**して枠に収める（収まりきらない場合も生成は完了） |

必須項目の確認と、文字数の注意喚起は、Bubble のフォーム側で行ってください。目安は §9 にあります。

**サイズごとに使われる項目**（それ以外の項目は無視されます）

| size | 使われる項目 |
|---|---|
| A4 / A3 / ハガキ | 7 項目（ただしデザインに枠が無い項目は無視。houwa_11_1 は §9） |
| 長尺縦 | eventName, eventPeriod, contactName |
| 長尺横 | eventName, contactName |
| 封筒(長形3号) | eventName |
| A4便箋 | 生成不可（Word のダウンロード専用） |

### 4.2 生成の状態（GET /generated-images/{generatedImageId}）

```json
200 OK（処理中）
{ "generatedImageId": "rgen-2b9e…", "sourceImageId": "rsrc-7f3a…", "status": "RUNNING" }

200 OK（完了）
{ "generatedImageId": "rgen-2b9e…", "sourceImageId": "rsrc-7f3a…", "status": "COMPLETED",
  "fileKey": "rule-generated-images/…/output.jpg",
  "thumbnailUrl": "https://…/thumbnail.jpg" }

200 OK（失敗）
{ "generatedImageId": "rgen-2b9e…", "sourceImageId": "rsrc-7f3a…", "status": "FAILED",
  "errorCode": "AI_GENERATION_FAILED", "errorMessage": "Image generation failed." }

404 { "errorCode": "NOT_FOUND", "message": "Generated image was not found." }
```

- `status` は `RUNNING` → `COMPLETED` または `FAILED` の 3 値（従来と同じ）。
- 完了時は `thumbnailUrl`（恒久 URL）を画面に表示し、保存には `fileKey` を §3.4 に渡します。
- 差し込んだテキストはレスポンスに含まれません（従来と同じ。フロント側で保持してください）。

### 4.3 作り直し（POST /generated-images/{generatedImageId}/regenerate）

```json
POST /generated-images/rgen-2b9e…/regenerate
{ "eventName": "法話会", "eventPeriod": "令和8年 11月22日（日）", "contactName": "〇〇寺" }

200 OK
{ "generatedImageId": "rgen-5c11…", "parentGeneratedImageId": "rgen-2b9e…", "sourceImageId": "rsrc-7f3a…", "status": "RUNNING" }
```

- 親と同じ原稿で、送ったテキストで描き直します。**全項目を送り直してください**（省略した項目は空欄になります。差分のマージはしません）。
- 作り直しても前の文字が残ることはありません。背景・イラストも変わりません。

### 4.4 生成の流れと推奨ポーリング

1. POST /generated-images → `generatedImageId` を保持
2. GET /generated-images/{id} を **2 秒間隔**でポーリング。`COMPLETED` か `FAILED` で停止。上限は **300 秒**（通常は 2〜4 秒で完了）
3. `COMPLETED`: `thumbnailUrl` を表示。保存ボタンで GET /files/url/{fileKey}?filename=… を取得して開く
4. 文言を直す場合は POST …/regenerate → 2 へ

## 5. エラー一覧

| HTTP | errorCode | 発生する API | 意味と Bubble 側の扱い |
|---|---|---|---|
| 400 | INVALID_REQUEST | 5, 7 | JSON の形式不正・型の不一致。入力の不足・文字数超過では発生しない |
| 400 | SOURCE_IMAGE_GENERATION_NOT_SUPPORTED | 5, 7 | **生成できない原稿**（A4便箋、旧方式の原稿）を指定した。作成ボタンを出さない（§3.2） |
| 401 | UNAUTHORIZED | — | 形式のみ定義（現在の PoC 運用では発生しない） |
| 403 | FORBIDDEN | 7 | 他のユーザーの生成画像を作り直そうとした |
| 403 | （API Gateway: `{"message":"Forbidden"}`） | すべて | x-api-key が無い、または誤り。設定を確認 |
| 404 | SOURCE_IMAGE_NOT_FOUND | 3, 5 | 原稿が無い、または非公開になった。一覧を取り直す |
| 404 | NOT_FOUND | 2, 6, 7 | グループまたは生成画像が無い |
| 429 | （API Gateway） | すべて | レート制限。1〜2 秒待って再試行 |
| 500 | INTERNAL_ERROR | すべて | 再試行しても直らなければ連絡 |

**生成の失敗（status = FAILED）**: `errorCode` は従来どおり既定値 `AI_GENERATION_FAILED` です。原稿データ側の不整合を示す `RULE_RENDER_FAILED` / `TEMPLATE_CANVAS_MISMATCH` / `FONT_NOT_AVAILABLE` が入ることもありますが、**Bubble では errorCode の種類に関係なく「生成に失敗しました。時間をおいて再度お試しください」として同じ扱いで構いません**。続く場合はご連絡ください（BE 側でデータを確認します）。

**再試行の目安**: 429 と 5xx は 1・2・4 秒の間隔で最大 3 回。400 と 404 は再試行しない。POST の応答が届かなかった場合に同じ内容を再送すると生成が 2 件になるため、再送前に直近の generatedImageId を確認してください。

## 6. ダウンロードの保存名

- ダウンロード用の原稿（A4便箋、旧方式の PPT/Word）は、ブラウザで `downloadUrl` を開くと **`original_<fileName>`**（例 `original_法話会11月_1_A4便箋.docx`）の名前で保存されます。
- 日本語の名前は `Content-Disposition` の `filename*`（UTF-8）で渡しています。あわせて ASCII だけの代替名（`filename=`）も付けており、日本語の部分は `_` に置き換わります（例 `original____11__1_A4__.docx`）。
- **注意**: Bubble のサーバーで一度取得してから利用者に再配信するなど、`filename*` を解釈しない経路を通すと、ASCII の代替名で保存されることがあります。日本語名で保存させたい場合は、利用者のブラウザで `downloadUrl` を直接開いてください。
- 生成画像を保存する場合は、§3.4 の `filename` パラメータで保存名を指定できます。

## 7. 環境

| 環境 | 状態 |
|---|---|
| dev | ルールベース版を配備し、houwa_11_1 で動作確認済み（2026-10-09） |
| qas | 従来のDL版稼働中 |

各環境のベース URL と API キーは、本ページには記載せず、別途共有します。

## 8. Gemini 版からの変更点

| # | 変更点 | Bubble への影響 |
|---|---|---|
| 1 | 生成が 30〜60 秒 → 2〜4 秒 | 影響なし。ポーリングの間隔・上限は §4.4 の値を推奨（従来値のままでも動く） |
| 2 | テキストが入力どおりに入り、作り直しで前の文字が残らない | 影響なし。作り直し前の確認ダイアログなどは簡略化できる |
| 3 | 入力の不足・未知の項目・文字数超過はエラーにせず、空欄は描かない／無視／縮小 | 影響なし（従来も 200）。必須チェックと文字数の注意はフォームで行う |
| 4 | 生成版グループは 1 グループ 7 サイズ、A4便箋は Word のダウンロード専用 | 影響なし（旧方式と同じく size で振り分け） |
| 5 | グループのサムネイルは常に A4 | 影響なし |
| 6 | 生成できない原稿への生成依頼は 400 SOURCE_IMAGE_GENERATION_NOT_SUPPORTED | 従来と同じコード。作成ボタンを出さなければ発生しない |
| 7 | FAILED の errorCode に原稿データ不整合のコードが入ることがある | errorCode で表示を分けていなければ影響なし（§5） |

## 9. 付録: houwa_11_1（法話会_11月_親しみ）の項目と最大文字数

| 項目 | A4 / A3 / ハガキ | 長尺縦 | 長尺横 | 封筒 |
|---|---|---|---|---|
| eventName（行事名） | 6 | 6 | 6 | 6 |
| eventPeriod（日時） | 20 | 20 | — | — |
| eventProgram（プログラム） | 70 | — | — | — |
| contactName（寺院名） | 8 | 8 | 8 | — |
| contactDetails（連絡先） | 35 | — | — | — |
| eventDetails / notes | このデザインには枠が無い（送っても無視） | — | — | — |

- 日時（eventPeriod）は「令和8年 11月15日（日）」のように書くと、年は小さく・数字は大きく、デザインに合わせて自動で組まれます。
- プログラム（eventProgram）は「13:30 勤行」のように 1 行に 1 項目（時刻＋内容）で、改行で区切ってください。

---

