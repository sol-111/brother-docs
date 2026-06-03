# API疎通確認 - 確認事項まとめ

- 実施日時: 2026-06-03
- 対象環境: brother-backend-dev-yokinini
- 対象API: GET /source-image-groups/{sourceImageGroupId}/source-images

---

## 事象1: 仕様書に記載のないフィールドが返却されている

レスポンスに以下3フィールドが含まれているが、API仕様書（brother_bubble_api_pre-release_guide.md）にもシステム仕様書（docs/api_spec.html）にも記載がない。

- `allowedEventFields`
- `disallowedEventFields`
- `allowedUploadExtensions`

### 実際のレスポンス（抜粋）

```json
{
  "sourceImageId": "src-bubble-1780448083182",
  "size": "A4",
  "thumbnailUrl": "...",
  "status": "active",
  "allowedEventFields": ["eventName", "eventPeriod", "eventDetails", "eventProgram", "notes"],
  "disallowedEventFields": [],
  "allowedUploadExtensions": [".jpg"]
}
```

### 確認したいこと

- これらのフィールドの用途・仕様

---

## 事象2: 既に期限切れの署名付きURLが返却されている

3件中1件（`src-9612d898-...`）のthumbnailUrlの署名発行日時が `2026-05-29` であり、既に期限切れの状態で返却されている。

APIを呼び出した時点で既にアクセスできないURLが返ってきている。

### 実際のレスポンス

| sourceImageId | X-Amz-Date | X-Amz-Expires | 状態 |
|---|---|---|---|
| src-9612d898-... | 20260529T083609Z | 600秒 | 既に期限切れ |

### 確認したいこと

- DBに署名付きURLをそのまま保存していないか（API呼び出し時にリアルタイム生成すべき）

---

## 事象3: thumbnailUrlが署名付き一時URLになっている

docs/api_spec.html の仕様では、thumbnailUrlは「**公開URL・期限なし。Bubble DBに保存して再利用**」と定義されている。

しかし実際のレスポンスでは署名付きURL（`X-Amz-Expires=600`、有効期限10分）が返却されており、Bubble DBに保存しても期限切れでアクセスできなくなる。

### 仕様（docs/api_spec.html）

> thumbnailUrl — サムネイルURL（公開URL・期限なし。Bubble DBに保存して再利用）

### 実際のレスポンス

| sourceImageId | X-Amz-Date | X-Amz-Expires | 状態 |
|---|---|---|---|
| src-bubble-1780448083182 | 20260603T005443Z | 600秒 | 署名付き（期限あり） |
| src-bubble-1780448083467 | 20260603T005443Z | 600秒 | 署名付き（期限あり） |

### 確認したいこと

- thumbnailUrlは公開URLで返却される想定か、署名付きURLで返却される想定か

### 補足

サムネイルはBubble DBに保存して一覧表示等で繰り返し利用するため、公開URLでの返却が望ましい。
元画像のダウンロードについては `GET /files/url/{fileKey+}` で都度署名付きURLを発行する設計になっているため、サムネイルが公開URLであってもセキュリティ上の問題はない。
