# Bubble向け 生成AI公開API リリースガイド

最終更新日: 2026-06-08
対象: Bubble側開発・QA・運用担当

## 1. 目的

本ドキュメントは、Bubble側へ以下の生成AI公開APIをリリースするための実装・試験・運用手順をまとめたものです。

- POST /generated-images
- GET /generated-images/{generatedImageId}
- POST /generated-images/{generatedImageId}/regenerate

## 2. 前提

- 認証: 現在は無認証呼び出しを許可（AllowUnauthenticatedGeneratedApi=true）
- 将来方針: API Gateway + Authorizer を有効化し、生成系APIを認証必須へ切替可能
- 生成処理: 非同期（即時に画像は返らない）
- Bubble側はポーリングで状態監視する


## 3. API仕様（Bubble実装用）

### 3.1 POST /generated-images

用途: 新規生成ジョブ受付

リクエスト例:

{
  "sourceImageId": "src-f366749e-02cb-429b-bddb-e15ca25f6117",
  "eventName": "ブラザー夏祭り盆踊り会",
  "eventPeriod": "2026年8月23日",
  "eventDetails": "任意",
  "eventProgram": "任意",
  "notes": "任意"
}

成功レスポンス例:

{
  "generatedImageId": "gen-xxxx",
  "sourceImageId": "src-xxxx",
  "status": "done"
}

注意:
- done は受付完了であり生成完了ではありません。
- 直後に GET /generated-images/{generatedImageId} をポーリングしてください。

### 3.2 GET /generated-images/{generatedImageId}

用途: 生成状態・結果確認

レスポンス例（処理中）:

{
  "generatedImageId": "gen-xxxx",
  "sourceImageId": "src-xxxx",
  "status": "PROCESSING"
}

レスポンス例（完了）:

{
  "generatedImageId": "gen-xxxx",
  "sourceImageId": "src-xxxx",
  "status": "COMPLETED",
  "fileKey": "generated-images/user-xxx/gen-xxxx/output.png",
  "thumbnailUrl": "https://.../thumbnail.png"
}

レスポンス例（失敗）:

{
  "generatedImageId": "gen-xxxx",
  "sourceImageId": "src-xxxx",
  "status": "FAILED",
  "errorCode": "AI_GENERATION_FAILED",
  "errorMessage": "AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."
}

### 3.3 POST /generated-images/{generatedImageId}/regenerate

用途: 既存生成結果を親にして再生成

リクエスト例:

{
  "eventName": "ブラザー記念夏祭り工場見学会",
  "eventPeriod": "2026年8月23日",
  "eventDetails": "任意",
  "eventProgram": "任意",
  "notes": "任意"
}

成功レスポンス例:

{
  "generatedImageId": "gen-child-xxxx",
  "parentGeneratedImageId": "gen-parent-xxxx",
  "sourceImageId": "src-xxxx",
  "status": "done"
}

## 4. Bubble側実装ルール

### 4.1 ポーリング

推奨値:
- 間隔: 3-5秒
- 最大待機: 2-5分
- 最大回数: 40-60回

終了条件:
- COMPLETED: 成果物を保存して完了
- FAILED: エラー表示して完了
- タイムアウト: 一時エラー表示（再確認導線を出す）

### 4.2 サイズ別入力制約

sourceImage.size によって許可される event フィールドが異なります。

重要:
- 許可されないフィールドを送ると 400 INVALID_REQUEST
- 生成時だけでなく再生成時も同じ制約

実装推奨:
- Bubble側フォームで size ごとに入力項目を出し分ける
- API送信前にクライアント側でも必須・許可チェックを行う

### 4.3 エラー処理

主に想定するエラー:
- 400 INVALID_REQUEST: 入力項目不正（size制約違反含む）
- 401 UNAUTHORIZED: 認証不足
- 403 FORBIDDEN: 所有者不一致
- 404 NOT_FOUND: ID不正
- 429 RATE_LIMIT_EXCEEDED / AI_REQUEST_FAILED:429: AI側クォータ制限
- 500 INTERNAL_ERROR: サーバ内部障害

Bubble側表示方針（推奨）:
- 400: 入力修正を促す
- 401/403: 再ログインまたは権限確認
- 429: 少し待って再実行を促す
- 500: 一時障害として再試行導線を表示


