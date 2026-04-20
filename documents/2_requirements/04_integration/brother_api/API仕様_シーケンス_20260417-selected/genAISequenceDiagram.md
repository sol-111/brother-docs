```mermaid
sequenceDiagram
    autonumber

    actor User as ユーザ
    participant FE as Frontend
    participant API as Backend API
    participant SIDB as Source Image DB
    participant GIDB as Generated Image DB
    participant AI as Generative AI
    participant FS as File Storage

    User ->> FE: 特定原稿詳細を開く
    FE ->> API: GET /source-images/{sourceImageId}
    API ->> SIDB: 原稿詳細を取得
    SIDB -->> API: 原稿メタ情報を返却
    API ->> FS: 原稿画像URLを取得
    FS -->> API: sourceImageUrl
    API -->> FE: 原稿詳細 + sourceImageUrl
    FE -->> User: 原稿詳細を表示
、
    User ->> FE: ターゲットテキストと差し替えテキストを入力し「生成」を実行
    FE ->> API: POST /generated-images\n{ sourceImageId, originalText、replaceText }
    API -->> FE: generatedImageId, status = RUNNING
    FE -->> User: 生成受付/生成中を表示    

    API ->> SIDB: sourceImageId の存在確認
    SIDB -->> API: 原稿情報を返却

    API ->> GIDB: 生成レコード作成\nstatus = PENDING
    GIDB -->> API: generatedImageId を返却

    API ->> AI: 元原稿 + replaceText で画像生成依頼
    AI -->> API: 生成画像データを返却

    API ->> FS: 生成画像を保存
    FS -->> API: generatedFileKey を返却

    API ->> GIDB: status = COMPLETED,\nfileKey = generatedFileKey を更新
    GIDB -->> API: 更新完了

    API -->> FE: generatedImageId, status = COMPLETED
    FE -->> User: 生成受付/生成完了を表示

    User ->> FE: 生成結果を表示
    FE ->> API: GET /generated-images/{generatedImageId}

    API ->> GIDB: generatedImageId で生成結果取得
    GIDB -->> API: status, fileKey を返却

    API ->> FS: fileKey から表示用URLを取得
    FS -->> API: generatedImageUrl

    API -->> FE: generatedImageId,\nstatus = COMPLETED,\ngeneratedImageUrl
    FE -->> User: 生成画像を表示
```