# Bubble向けAPIリリース説明書（公開5API）

作成日: 2026-05-29
対象環境: brother-backend-dev-yokinini
API Base URL: https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod

---

## 1. 本書の目的

本書は、Bubble側に公開する以下5つのAPIについて、
利用方法・レスポンス仕様・実運用時の注意点・テスト結果を共有するためのリリース用ドキュメントです。

対象API:

1. GET /source-image-groups
2. GET /source-image-groups/{sourceImageGroupId}/source-images
3. GET /common-print-groups
4. GET /common-print-groups/{commonPrintGroupId}/prints
5. GET /files/url/{fileKey+}

---

## 2. 共通仕様

- メソッド: すべて GET
- レスポンス形式: JSON
- Content-Type: application/json; charset=utf-8
- 文字コード: UTF-8（日本語項目を含む）

推奨リクエストヘッダ:

- Accept: application/json

---

## 3. API詳細

### 3.1 GET /source-image-groups

用途:

- Bubbleでテンプレート一覧（ソースイメージグループ）を取得する。

レスポンス例:

```json
{
  "items": [
    {
      "sourceImageGroupId": "sig1",
      "thumbnailUrl": "http://test.com",
      "status": "active"
    }
  ]
}
```

主な項目:

- sourceImageGroupId: グループ識別子（後続APIで使用）
- thumbnailUrl: 一覧表示用サムネイルURL
- status: 利用状態

---

### 3.2 GET /source-image-groups/{sourceImageGroupId}/source-images

用途:

- 指定グループ配下のサイズ別ソース画像を取得する。

パスパラメータ:

- sourceImageGroupId: 3.1で取得したID

レスポンス例:

```json
{
  "items": [
    {
      "sourceImageId": "src-9612d898-76e8-4e37-ba86-7eb6a36d3708",
      "size": "A4",
      "thumbnailUrl": "https://...signed-url...",
      "status": "active"
    }
  ]
}
```

主な項目:

- sourceImageId: 生成API等で参照する画像ID
- size: サイズ種別
- thumbnailUrl: 該当画像のサムネイル表示URL
- status: 利用状態

---

### 3.3 GET /common-print-groups

用途:

- 共通印刷物グループ一覧を取得する。

レスポンス例:

```json
{
  "items": [
    {
      "commonPrintGroupId": "cpg1",
      "name": "サイン・掲示"
    }
  ]
}
```

主な項目:

- commonPrintGroupId: グループ識別子（後続APIで使用）
- name: グループ名

---

### 3.4 GET /common-print-groups/{commonPrintGroupId}/prints

用途:

- 指定グループの共通印刷物一覧を取得する。

パスパラメータ:

- commonPrintGroupId: 3.3で取得したID

レスポンス例:

```json
{
  "items": [
    {
      "commonPrintId": "cpr-635fcd6f-6222-45ba-ada9-f22fbe5bdeec",
      "fileName": "do-not-touch-h1000-p.pptx",
      "fileKey": "common-prints/1780043657150-e7f17ed0-4355-4f1c-9d30-727c64e50b1c-do-not-touch-h1000-p.pptx",
      "thumbnailUrl": "https://commonPrint.com",
      "status": "active"
    }
  ]
}
```

主な項目:

- commonPrintId: 共通印刷物ID
- fileName: ファイル名
- fileKey: ダウンロードURL発行に使う永続キー
- thumbnailUrl: 一覧表示用URL
- status: 利用状態

---

### 3.5 GET /files/url/{fileKey+}

用途:

- fileKeyをもとに、ダウンロード用の署名付き一時URLを取得する。

パスパラメータ:

- fileKey: 3.4等で取得したキー（/ を含む）

レスポンス例:

```json
{
  "url": "https://...signed-url...",
  "expiresAt": "2026-05-29T11:20:44.216Z"
}
```

主な項目:

- url: 実ダウンロードURL（署名付き）
- expiresAt: URLの有効期限

重要:

- 画面表示用URLと、実ダウンロード用URLは用途が異なります。
- ダウンロード時は必ず本APIを都度呼び出してください。

---

## 4. Bubble実装時の推奨フロー

1. テンプレート選択用に GET /source-image-groups を取得
2. 選択後、GET /source-image-groups/{sourceImageGroupId}/source-images を取得
3. 共通印刷物選択時は GET /common-print-groups を取得
4. グループ選択後、GET /common-print-groups/{commonPrintGroupId}/prints を取得
5. ファイルダウンロード直前に GET /files/url/{fileKey+} を呼び、返却urlへ遷移

---

## 5. テストエビデンス（2026-05-29 実施）

### 5.1 実行コマンド（PowerShell）

```powershell
Invoke-WebRequest -Uri 'https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/common-print-groups' -Method GET -UseBasicParsing
Invoke-WebRequest -Uri 'https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/common-print-groups/cpg1/prints' -Method GET -UseBasicParsing
Invoke-WebRequest -Uri 'https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/source-image-groups' -Method GET -UseBasicParsing
Invoke-WebRequest -Uri 'https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/source-image-groups/sig1/source-images' -Method GET -UseBasicParsing
Invoke-WebRequest -Uri 'https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/files/url/common-prints/1780043657150-e7f17ed0-4355-4f1c-9d30-727c64e50b1c-do-not-touch-h1000-p.pptx' -Method GET -UseBasicParsing
```

### 5.2 結果サマリ

- GET /common-print-groups: 200
- GET /common-print-groups/cpg1/prints: 200
- GET /source-image-groups: 200
- GET /source-image-groups/sig1/source-images: 200
- GET /files/url/{fileKey+}: 200

すべて Content-Type は application/json; charset=utf-8 で応答。

