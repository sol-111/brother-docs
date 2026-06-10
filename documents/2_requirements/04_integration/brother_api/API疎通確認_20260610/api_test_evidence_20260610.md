# API疎通確認エビデンス

- 実施日時: 2026-06-10
- 対象環境: brother-backend-dev-yokinini
- Base URL: `https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod`
- 目的: 生成系API（POST /generated-images, GET /generated-images/{id}, POST /regenerate）の初回疎通確認 + 既存GET系APIの継続確認

---

## 1. GET /source-image-groups

**ステータス: 200 OK**

```json
{
  "items": [
    {
      "sourceImageGroupId": "sig3",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/sig3/src-f366749e-02cb-429b-bddb-e15ca25f6117/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageGroupId": "sig1",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/sig1/src-bubble-1780448083182/thumbnail.png",
      "status": "active"
    }
  ]
}
```

---

## 2. GET /source-image-groups/sig3/source-images

**ステータス: 200 OK**

```json
{
  "items": [
    {
      "sourceImageId": "src-b9bfb2fc-0b50-4137-81d5-797969548444",
      "size": "A3",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/sig3/src-b9bfb2fc-0b50-4137-81d5-797969548444/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageId": "src-f366749e-02cb-429b-bddb-e15ca25f6117",
      "size": "長尺縦",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/sig3/src-f366749e-02cb-429b-bddb-e15ca25f6117/thumbnail.png",
      "status": "active"
    }
  ]
}
```

---

## 3. GET /common-print-groups

**ステータス: 200 OK**

```json
{
  "items": [
    {
      "commonPrintGroupId": "cpg1",
      "name": "サイン・掲示"
    },
    {
      "commonPrintGroupId": "cpg3",
      "name": "ワークショップ"
    }
  ]
}
```

---

## 4. GET /common-print-groups/cpg1/prints

**ステータス: 200 OK**

```json
{
  "items": [
    {
      "commonPrintId": "cpr-635fcd6f-6222-45ba-ada9-f22fbe5bdeec",
      "fileName": "do-not-touch-h1000-p.pptx",
      "fileKey": "common-prints/1780043657150-e7f17ed0-4355-4f1c-9d30-727c64e50b1c-do-not-touch-h1000-p.pptx",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/common-prints/1780907894049-25fe7c33-fbe0-40b1-aaac-ac0a4b459fd1-do-not-touch-h1000-p.jpg",
      "status": "active"
    },
    {
      "commonPrintId": "cpr-de9fcfa6-229a-400e-87c5-f1d89343c6d5",
      "fileName": "do-not-enter-h1000-p.pptx",
      "fileKey": "common-prints/1780043595723-5c2dd902-8e8a-492e-b513-492315dc99d7-do-not-enter-h1000-p.pptx",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/common-prints/1780907894449-82e62491-ad2d-4f03-a72f-038eee1734d1-do-not-enter-h1000-p.jpg",
      "status": "active"
    }
  ]
}
```

---

## 5. GET /files/url/{fileKey+}（共通印刷物）

リクエストパス: `/files/url/common-prints/1780043657150-e7f17ed0-4355-4f1c-9d30-727c64e50b1c-do-not-touch-h1000-p.pptx`

**ステータス: 200 OK**

```json
{
  "url": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/common-prints/1780043657150-e7f17ed0-4355-4f1c-9d30-727c64e50b1c-do-not-touch-h1000-p.pptx?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3G5WLETR3%2F20260610%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260610T074528Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaDmFwLW5vcnRoZWFzdC0xIkgwRgIhALQjXQAwXNycEfMeTE6Sd4aNFPt5QSOLI3XB%2BPzSVYBVAiEAlzBpUf54s1GAT4z8MtUHVgGwB9hM%2BIt5zmHS2hP7Y%2B4qgAUI4f%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARAAGgwzMzI4OTY5Mzg2MTQiDBEjdUGxmDqRqAwWOCrUBPFouHnsMAe4mYjy0wYELb8V%2F%2FkMNqxb4tRp7wv9tWxCpb625uFl7Du8MIJ5xxj8SlLGR4H1fuvlc5iH646R%2F5%2FTPSbj88%2BZaLl5XTZCVUatSWafqJCm4ybXcqDSdlHQr%2FzysdhRN7GM1J%2BqBTQN2p9KpRfqNBbAqp%2Fd4%2Fw4J81FcC2VPWeiJQ4NFqAcE2p0VENu4fE6dV63EoHbB00zih8uSOC2FvVtNxV85nK9MvQWDMTdDxhM7KzX0dwEMHOt1ELNEmfigJlEdAg%2BCDjFciBYswYzxCqnZOXkTAOh1u6CJgkOqYZ0DQINvE80xQsr1HDEvVOZZkY%2BbYx0OOUPS04YP9W0UNkIWb2%2Fs6KsOSXi%2FDnYpVj00zY8wScZu5g8xEcU%2FyG6a1phQOYt2V1ynMFgw6rsoCVzPN7km2wSTwakyQMToEBM1PQOtMK2BtQNFToF0L6BBhyxV7MZGnL5iTY52XBOC7d3d0ZlICneQORF5y%2BdY3FKouCOViNHUqKBMGPJP%2BjM%2BbTH33RQQnha%2BgTyjLZQ4iJPb6ohWHwhZSrwncqAhu40LRaDtaK5vVr%2BDbbhOcO8aqzIDXAvTIAsf66xpCyFj92k%2FR0N0Gh9W%2FcrBqbjJ2vmwdL%2FIVJo9nDP8g3uyjIXCBwRF0Gbw8StM5%2BXYZVNFb36t2om%2FVNtnE8BDs6YSK6MjZgY%2FNS4gcTqNNrnazBOtrFHtdCKVNjJvCwkYJrAGPDnTK%2BE6A85Dhdn6rTl1Q0InQ2Xgzf5eOLZ8HRE9U4IarbUsDmK7%2Bvc5shXWxQtMJespNEGOqIBPGThu%2B7GAQT2Yw61fcNL7pEEDxv86cOgsrt9clKBmXJkDg2DJH72gP48w9w27Oyjd3bGbhBU4MFDAjPgmU%2Bjm0c5V1Yjulq%2F4gD8gjjSn3YL4%2BkOCyNGk2d9LKpyUSZianvoiI2XmXRpJLBOgDXiliHv3UgY3yzJ8EWIHUYop5diMgnllYk4nD%2Fd6GL5E9XADDEDW4AxEVnQwwbB%2B4fg26Kc&X-Amz-Signature=6a35482f9eb5201e7d7e06c91522af1ae7733200522df24de8d4e85b0194a67f&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
  "expiresAt": "2026-06-10T07:55:28.442Z"
}
```

---

## 6. POST /generated-images（長尺縦サイズ）

使用sourceImageId: `src-f366749e-02cb-429b-bddb-e15ca25f6117`（size: 長尺縦）

### 6-a. サイズ別入力制約の確認（全フィールド送信 → エラー）

リクエスト:
```json
{
  "sourceImageId": "src-f366749e-02cb-429b-bddb-e15ca25f6117",
  "eventName": "ブラザー夏祭り盆踊り会",
  "eventPeriod": "2026年8月23日",
  "eventDetails": "境内にて夏祭りを開催いたします",
  "eventProgram": "10:00 開会 / 12:00 盆踊り / 15:00 閉会",
  "notes": "雨天時は中止となります"
}
```

**ステータス: 400 Bad Request**

```json
{
  "errorCode": "INVALID_REQUEST",
  "message": "size=長尺縦 does not allow fields [eventDetails, eventProgram, notes]. Allowed disallowed set: [eventDetails, eventProgram, notes]"
}
```

→ サイズ別入力制約のバリデーションが正しく動作していることを確認。

### 6-b. 許可フィールドのみで再送信 → 成功

リクエスト:
```json
{
  "sourceImageId": "src-f366749e-02cb-429b-bddb-e15ca25f6117",
  "eventName": "ブラザー夏祭り盆踊り会",
  "eventPeriod": "2026年8月23日"
}
```

**ステータス: 200 OK**

```json
{
  "generatedImageId": "gen-cc78c82f-7bf4-4984-94db-002e6beacbc8",
  "sourceImageId": "src-f366749e-02cb-429b-bddb-e15ca25f6117",
  "status": "done"
}
```

---

## 7. GET /generated-images/{generatedImageId}（ポーリング）

対象: `gen-cc78c82f-7bf4-4984-94db-002e6beacbc8`

### ポーリング1回目（即時）

**ステータス: 200 OK**

```json
{
  "generatedImageId": "gen-cc78c82f-7bf4-4984-94db-002e6beacbc8",
  "sourceImageId": "src-f366749e-02cb-429b-bddb-e15ca25f6117",
  "status": "PROCESSING"
}
```

### ポーリング2回目（5秒後）

**ステータス: 200 OK**

```json
{
  "generatedImageId": "gen-cc78c82f-7bf4-4984-94db-002e6beacbc8",
  "sourceImageId": "src-f366749e-02cb-429b-bddb-e15ca25f6117",
  "status": "PROCESSING"
}
```

### ポーリング3回目（さらに10秒後）→ 完了

**ステータス: 200 OK**

```json
{
  "generatedImageId": "gen-cc78c82f-7bf4-4984-94db-002e6beacbc8",
  "sourceImageId": "src-f366749e-02cb-429b-bddb-e15ca25f6117",
  "status": "COMPLETED",
  "fileKey": "generated-images/bubble-test-user/gen-cc78c82f-7bf4-4984-94db-002e6beacbc8/output.png",
  "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-cc78c82f-7bf4-4984-94db-002e6beacbc8/thumbnail.png"
}
```

---

## 8. GET /files/url/{fileKey+}（生成画像のDL URL）

リクエストパス: `/files/url/generated-images/bubble-test-user/gen-cc78c82f-7bf4-4984-94db-002e6beacbc8/output.png`

**ステータス: 200 OK**

```json
{
  "url": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/generated-images/bubble-test-user/gen-cc78c82f-7bf4-4984-94db-002e6beacbc8/output.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3G5WLETR3%2F20260610%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260610T074635Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaDmFwLW5vcnRoZWFzdC0xIkgwRgIhALQjXQAwXNycEfMeTE6Sd4aNFPt5QSOLI3XB%2BPzSVYBVAiEAlzBpUf54s1GAT4z8MtUHVgGwB9hM%2BIt5zmHS2hP7Y%2B4qgAUI4f%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARAAGgwzMzI4OTY5Mzg2MTQiDBEjdUGxmDqRqAwWOCrUBPFouHnsMAe4mYjy0wYELb8V%2F%2FkMNqxb4tRp7wv9tWxCpb625uFl7Du8MIJ5xxj8SlLGR4H1fuvlc5iH646R%2F5%2FTPSbj88%2BZaLl5XTZCVUatSWafqJCm4ybXcqDSdlHQr%2FzysdhRN7GM1J%2BqBTQN2p9KpRfqNBbAqp%2Fd4%2Fw4J81FcC2VPWeiJQ4NFqAcE2p0VENu4fE6dV63EoHbB00zih8uSOC2FvVtNxV85nK9MvQWDMTdDxhM7KzX0dwEMHOt1ELNEmfigJlEdAg%2BCDjFciBYswYzxCqnZOXkTAOh1u6CJgkOqYZ0DQINvE80xQsr1HDEvVOZZkY%2BbYx0OOUPS04YP9W0UNkIWb2%2Fs6KsOSXi%2FDnYpVj00zY8wScZu5g8xEcU%2FyG6a1phQOYt2V1ynMFgw6rsoCVzPN7km2wSTwakyQMToEBM1PQOtMK2BtQNFToF0L6BBhyxV7MZGnL5iTY52XBOC7d3d0ZlICneQORF5y%2BdY3FKouCOViNHUqKBMGPJP%2BjM%2BbTH33RQQnha%2BgTyjLZQ4iJPb6ohWHwhZSrwncqAhu40LRaDtaK5vVr%2BDbbhOcO8aqzIDXAvTIAsf66xpCyFj92k%2FR0N0Gh9W%2FcrBqbjJ2vmwdL%2FIVJo9nDP8g3uyjIXCBwRF0Gbw8StM5%2BXYZVNFb36t2om%2FVNtnE8BDs6YSK6MjZgY%2FNS4gcTqNNrnazBOtrFHtdCKVNjJvCwkYJrAGPDnTK%2BE6A85Dhdn6rTl1Q0InQ2Xgzf5eOLZ8HRE9U4IarbUsDmK7%2Bvc5shXWxQtMJespNEGOqIBPGThu%2B7GAQT2Yw61fcNL7pEEDxv86cOgsrt9clKBmXJkDg2DJH72gP48w9w27Oyjd3bGbhBU4MFDAjPgmU%2Bjm0c5V1Yjulq%2F4gD8gjjSn3YL4%2BkOCyNGk2d9LKpyUSZianvoiI2XmXRpJLBOgDXiliHv3UgY3yzJ8EWIHUYop5diMgnllYk4nD%2Fd6GL5E9XADDEDW4AxEVnQwwbB%2B4fg26Kc&X-Amz-Signature=390cc3a000e993f009e4b4a8e926d8291ba58e9c845133242a3d0b5b527f1d69&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
  "expiresAt": "2026-06-10T07:56:35.783Z"
}
```

---

## 9. POST /generated-images/{generatedImageId}/regenerate

対象: `gen-cc78c82f-7bf4-4984-94db-002e6beacbc8`

リクエスト:
```json
{
  "eventName": "ブラザー記念夏祭り工場見学会",
  "eventPeriod": "2026年8月23日〜24日"
}
```

**ステータス: 200 OK**

```json
{
  "generatedImageId": "gen-101d4d57-788d-45fe-984b-b198f69442dc",
  "parentGeneratedImageId": "gen-cc78c82f-7bf4-4984-94db-002e6beacbc8",
  "sourceImageId": "src-f366749e-02cb-429b-bddb-e15ca25f6117",
  "status": "done"
}
```

### 再生成のポーリング（10秒後）→ 完了

**ステータス: 200 OK**

```json
{
  "generatedImageId": "gen-101d4d57-788d-45fe-984b-b198f69442dc",
  "sourceImageId": "src-f366749e-02cb-429b-bddb-e15ca25f6117",
  "status": "COMPLETED",
  "fileKey": "generated-images/bubble-test-user/gen-101d4d57-788d-45fe-984b-b198f69442dc/output.png",
  "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-101d4d57-788d-45fe-984b-b198f69442dc/thumbnail.png"
}
```

---

## 10. POST /generated-images（A3サイズ - 全フィールド送信）

使用sourceImageId: `src-b9bfb2fc-0b50-4137-81d5-797969548444`（size: A3）

リクエスト:
```json
{
  "sourceImageId": "src-b9bfb2fc-0b50-4137-81d5-797969548444",
  "eventName": "ブラザー夏祭り盆踊り会",
  "eventPeriod": "2026年8月23日",
  "eventDetails": "境内にて夏祭りを開催いたします",
  "eventProgram": "10:00 開会 / 12:00 盆踊り / 15:00 閉会",
  "notes": "雨天時は中止となります"
}
```

**ステータス: 200 OK**

```json
{
  "generatedImageId": "gen-4e8cbfa4-c53d-41b6-b358-58f31e34c528",
  "sourceImageId": "src-b9bfb2fc-0b50-4137-81d5-797969548444",
  "status": "done"
}
```

### ポーリング（15秒後）→ 完了

**ステータス: 200 OK**

```json
{
  "generatedImageId": "gen-4e8cbfa4-c53d-41b6-b358-58f31e34c528",
  "sourceImageId": "src-b9bfb2fc-0b50-4137-81d5-797969548444",
  "status": "COMPLETED",
  "fileKey": "generated-images/bubble-test-user/gen-4e8cbfa4-c53d-41b6-b358-58f31e34c528/output.png",
  "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-4e8cbfa4-c53d-41b6-b358-58f31e34c528/thumbnail.png"
}
```

---

## 11. POST /generated-images（A4サイズ - 全フィールド送信）

使用sourceImageId: `src-bubble-1780448083182`（size: A4、グループ: sig1）

リクエスト:
```json
{
  "sourceImageId": "src-bubble-1780448083182",
  "eventName": "ブラザー夏祭り盆踊り会",
  "eventPeriod": "2026年8月23日",
  "eventDetails": "境内にて夏祭りを開催いたします",
  "eventProgram": "10:00 開会 / 12:00 盆踊り / 15:00 閉会",
  "notes": "雨天時は中止となります"
}
```

**ステータス: 200 OK**

```json
{
  "generatedImageId": "gen-59f290a2-d188-4e33-a4c2-a69d6e1825c7",
  "sourceImageId": "src-bubble-1780448083182",
  "status": "done"
}
```

### ポーリング（20秒後）→ 完了

**ステータス: 200 OK**

```json
{
  "generatedImageId": "gen-59f290a2-d188-4e33-a4c2-a69d6e1825c7",
  "sourceImageId": "src-bubble-1780448083182",
  "status": "COMPLETED",
  "fileKey": "generated-images/bubble-test-user/gen-59f290a2-d188-4e33-a4c2-a69d6e1825c7/output.png",
  "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-59f290a2-d188-4e33-a4c2-a69d6e1825c7/thumbnail.png"
}
```

---

## 12. 初回生成まとめ（全サイズ）

| サイズ | sourceImageId | generatedImageId | 結果 |
|---|---|---|---|
| A3 | src-b9bfb2fc-0b50-4137-81d5-797969548444 | gen-2e009946-0e67-49c6-be91-5cf83cd60571 | COMPLETED |
| 長尺縦 | src-f366749e-02cb-429b-bddb-e15ca25f6117 | gen-37e187bb-d6ee-4c88-9936-872c0b8a1d0b | COMPLETED |
| A4 | src-bubble-1780448083182 | gen-59f290a2-d188-4e33-a4c2-a69d6e1825c7 | COMPLETED |

全サイズ GET /files/url/{fileKey+} でDL URL取得→ダウンロード成功。01_初回生成_印刷物/ に保存済み。

---

## 13. POST /generated-images/{id}/regenerate（全サイズ）

### 13-a. A3再生成

対象: `gen-2e009946-0e67-49c6-be91-5cf83cd60571`

リクエスト:
```json
{
  "eventName": "ブラザー記念夏祭り工場見学会",
  "eventPeriod": "2026年8月23日〜24日",
  "eventDetails": "工場敷地内にて夏祭りと工場見学会を開催",
  "eventProgram": "10:00 工場見学 / 12:00 昼食 / 14:00 盆踊り / 16:00 閉会",
  "notes": "雨天決行・荒天中止"
}
```

**1回目: FAILED（429）**

```json
{
  "generatedImageId": "gen-be260cf8-333a-4b36-8589-2a248fb6823b",
  "parentGeneratedImageId": "gen-2e009946-0e67-49c6-be91-5cf83cd60571",
  "sourceImageId": "src-b9bfb2fc-0b50-4137-81d5-797969548444",
  "status": "done"
}
```

ポーリング結果:
```json
{
  "generatedImageId": "gen-be260cf8-333a-4b36-8589-2a248fb6823b",
  "sourceImageId": "src-b9bfb2fc-0b50-4137-81d5-797969548444",
  "status": "FAILED",
  "errorCode": "AI_GENERATION_FAILED",
  "errorMessage": "AI_REQUEST_FAILED:429:Resource has been exhausted (e.g. check quota)."
}
```

**2回目（リトライ）: COMPLETED**

```json
{
  "generatedImageId": "gen-b310de71-8892-4351-8814-6e0c8a7b82d7",
  "parentGeneratedImageId": "gen-2e009946-0e67-49c6-be91-5cf83cd60571",
  "sourceImageId": "src-b9bfb2fc-0b50-4137-81d5-797969548444",
  "status": "done"
}
```

ポーリング結果:
```json
{
  "generatedImageId": "gen-b310de71-8892-4351-8814-6e0c8a7b82d7",
  "sourceImageId": "src-b9bfb2fc-0b50-4137-81d5-797969548444",
  "status": "COMPLETED",
  "fileKey": "generated-images/bubble-test-user/gen-b310de71-8892-4351-8814-6e0c8a7b82d7/output.png",
  "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-b310de71-8892-4351-8814-6e0c8a7b82d7/thumbnail.png"
}
```

### 13-b. 長尺縦再生成

対象: `gen-37e187bb-d6ee-4c88-9936-872c0b8a1d0b`

リクエスト:
```json
{
  "eventName": "ブラザー記念夏祭り工場見学会",
  "eventPeriod": "2026年8月23日〜24日"
}
```

**ステータス: 200 OK → COMPLETED**

```json
{
  "generatedImageId": "gen-41bc2dcb-8043-4a9c-8a1f-67a54d5c371c",
  "parentGeneratedImageId": "gen-37e187bb-d6ee-4c88-9936-872c0b8a1d0b",
  "sourceImageId": "src-f366749e-02cb-429b-bddb-e15ca25f6117",
  "status": "done"
}
```

ポーリング結果:
```json
{
  "generatedImageId": "gen-41bc2dcb-8043-4a9c-8a1f-67a54d5c371c",
  "sourceImageId": "src-f366749e-02cb-429b-bddb-e15ca25f6117",
  "status": "COMPLETED",
  "fileKey": "generated-images/bubble-test-user/gen-41bc2dcb-8043-4a9c-8a1f-67a54d5c371c/output.png",
  "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-41bc2dcb-8043-4a9c-8a1f-67a54d5c371c/thumbnail.png"
}
```

### 13-c. A4再生成

対象: `gen-59f290a2-d188-4e33-a4c2-a69d6e1825c7`

リクエスト:
```json
{
  "eventName": "ブラザー記念夏祭り工場見学会",
  "eventPeriod": "2026年8月23日〜24日",
  "eventDetails": "工場敷地内にて夏祭りと工場見学会を開催",
  "eventProgram": "10:00 工場見学 / 12:00 昼食 / 14:00 盆踊り / 16:00 閉会",
  "notes": "雨天決行・荒天中止"
}
```

**ステータス: 200 OK → COMPLETED**

```json
{
  "generatedImageId": "gen-fd2d8c6c-1634-4eac-897e-353e317762a9",
  "parentGeneratedImageId": "gen-59f290a2-d188-4e33-a4c2-a69d6e1825c7",
  "sourceImageId": "src-bubble-1780448083182",
  "status": "done"
}
```

ポーリング結果:
```json
{
  "generatedImageId": "gen-fd2d8c6c-1634-4eac-897e-353e317762a9",
  "sourceImageId": "src-bubble-1780448083182",
  "status": "COMPLETED",
  "fileKey": "generated-images/bubble-test-user/gen-fd2d8c6c-1634-4eac-897e-353e317762a9/output.png",
  "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/generated-images/bubble-test-user/gen-fd2d8c6c-1634-4eac-897e-353e317762a9/thumbnail.png"
}
```

---

## 14. 再生成まとめ（全サイズ）

| サイズ | 元generatedImageId | 再生成generatedImageId | 結果 | 備考 |
|---|---|---|---|---|
| A3 | gen-2e009946-... | gen-b310de71-... | COMPLETED | 1回目429失敗→リトライで成功 |
| 長尺縦 | gen-37e187bb-... | gen-41bc2dcb-... | COMPLETED | |
| A4 | gen-59f290a2-... | gen-fd2d8c6c-... | COMPLETED | |

全サイズ GET /files/url/{fileKey+} でDL URL取得→ダウンロード成功。02_再生成_印刷物/ に保存済み。

---

## 結果サマリ

### GET系API（既存）

| # | エンドポイント | ステータス |
|---|---|---|
| 1 | GET /source-image-groups | 200 OK |
| 2 | GET /source-image-groups/sig3/source-images | 200 OK |
| 3 | GET /common-print-groups | 200 OK |
| 4 | GET /common-print-groups/cpg1/prints | 200 OK |
| 5 | GET /files/url/{fileKey+}（共通印刷物） | 200 OK |

### 初回生成（今回新規）

| サイズ | POST /generated-images | GET ポーリング | GET /files/url DL |
|---|---|---|---|
| A3 | 200 OK | COMPLETED | 200 OK |
| 長尺縦 | 200 OK | COMPLETED | 200 OK |
| A4 | 200 OK | COMPLETED | 200 OK |

### 再生成（今回新規）

| サイズ | POST /regenerate | GET ポーリング | GET /files/url DL | 備考 |
|---|---|---|---|---|
| A3 | 200 OK | COMPLETED | 200 OK | 1回目429失敗→リトライで成功 |
| 長尺縦 | 200 OK | COMPLETED | 200 OK | |
| A4 | 200 OK | COMPLETED | 200 OK | |

全API正常応答を確認。
