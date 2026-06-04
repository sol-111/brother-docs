# API疎通確認エビデンス（再確認）

- 実施日時: 2026-06-04
- 対象環境: brother-backend-dev-yokinini
- Base URL: `https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod`
- 目的: 2026-06-03に報告した事象1〜3の修正確認

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
      "thumbnailUrl": "https://d111111abcdef8.cloudfront.net/source-images/sig1/src-bubble-1780448083182/thumbnail.png",
      "status": "active"
    }
  ]
}
```

---

## 2. GET /source-image-groups/sig1/source-images

**ステータス: 200 OK**

```json
{
  "items": [
    {
      "sourceImageId": "src-bubble-1780448083182",
      "size": "A4",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/sig1/src-bubble-1780448083182/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageId": "src-bubble-1780448083467",
      "size": "A4",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/sig1/src-bubble-1780448083467/thumbnail.png",
      "status": "active"
    },
    {
      "sourceImageId": "src-9612d898-76e8-4e37-ba86-7eb6a36d3708",
      "size": "A4",
      "thumbnailUrl": "https://d3p6nztoczy84t.cloudfront.net/source-images/sig1/src-9612d898-76e8-4e37-ba86-7eb6a36d3708/thumbnail.png",
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
      "thumbnailUrl": "https://commonPrint.com",
      "status": "active"
    },
    {
      "commonPrintId": "cpr-de9fcfa6-229a-400e-87c5-f1d89343c6d5",
      "fileName": "do-not-enter-h1000-p.pptx",
      "fileKey": "common-prints/1780043595723-5c2dd902-8e8a-492e-b513-492315dc99d7-do-not-enter-h1000-p.pptx",
      "thumbnailUrl": "https://commonPrint.com",
      "status": "active"
    }
  ]
}
```

---

## 5. GET /files/url/{fileKey+}

リクエストパス: `/files/url/common-prints/1780043657150-e7f17ed0-4355-4f1c-9d30-727c64e50b1c-do-not-touch-h1000-p.pptx`

**ステータス: 200 OK**

```json
{
  "url": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/common-prints/1780043657150-e7f17ed0-4355-4f1c-9d30-727c64e50b1c-do-not-touch-h1000-p.pptx?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3MBHGSJOC%2F20260604%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260604T050855Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEIX%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLW5vcnRoZWFzdC0xIkcwRQIgRjRNHflnMJm6aPaqX3oGprqCTavFCl3kpAcPiFe6%2FDcCIQCBVTkhPvZ3h3YsRZCN2wDh4AjtaJ9zgfUyAiQ3oFqd3Cr3BAhOEAAaDDMzMjg5NjkzODYxNCIMsfymipxgGgzqVGOcKtQETa5JwtGZs%2By%2FhdtkitszlYOTIJrsp2kT3HEm37owN5xuATu6oYAlrpD%2FjIBe7rtf0%2FCep%2BpmPxSvMS7%2BOpQsXRqjde27h84V1BDHmjyGdvYRp3Uek9KLgHFlSf4G4r7nIVWm%2FeXnpyQVyCDd1QuSa7Eqavk3sBNPZRo5BUorhZad6Mhxw8Psh6Vpx8oQsEJRhvOIYQVAl1HivicNkvVGDvP0DNvkyKUJxWsa06ezkyN2j2Q8A89R39WvG18A%2BTgMWHB1mJ3YaXXSSUqT%2BBOgvsbtChJwhdaiO2PF6JMOBIDmYyDZfuYZ%2F9SR1J0SJXfFyidJMBOMYmZB8ZjlXC0yPONIbR8jt%2FGldnQxkjrdjwzJEnD3E9w4GFbbkkCQjzTgIR%2F97A%2FQqEaAcSI5s1F38tteVxonerXi4wuclT2QDotIgaaO%2FeWVVWil0aGNQh8gEjBdgJRtWKvksT%2FM7YruEzU%2FZmZb3tXyu2ZpO7fD%2F7afQZeqjQfz%2BGr4pANH3pH6oPWY3yBCbL%2F0ns8ACS1K%2FqcpEN%2FrRNPw85SVUbyQbF57EO%2BectQc0fVPPjt0oO%2BFhGJDjMCd6UmyUL%2FYKIwu%2FMbK3K0iUp3YS8N10QnN2CNF%2FQlqhsBFgLqKZPvfCwYIAFEDWzYfywCJ8plmuKUaV5nLkKcV3D1JVhJiK23bhJ1kJX0OplxJBAR%2FzZAAKv2onxnAiAwYmLuCXRA%2FrP8384AhktyndVvzguBSm%2F5bKMx9CtXNb2H9bcwLrHpXOKKBJ7eKD4id2UyOpXRzYQB2to2p4Hsw5pCE0QY6owHMbCTX6FgLM8IY%2BHZhBYU5LD8QfcHL4M3qgb8RvfNX99CS5l0PVgS4nJXr4QzNS7em5ePAbqNZE7hoyX%2FKzkAE6b8PtIUpVrwIaKTuqOvsSG0PXvp6nbBXSNM9UA12V85rVahHEo1UQ35OxR62Lsz0K%2Fvn%2FfWAuvh%2BMYwE%2FVHXHJg%2B1QFopvsRSq5B90S6RlsypvxhrBEeeahjcKLAnrz7WcMu&X-Amz-Signature=2cc8e4bcd70e649e75be470f7db1336bae7d08185774c7999d662e566c5e9e8f&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
  "expiresAt": "2026-06-04T05:18:55.293Z"
}
```

---

## 結果サマリ

| # | エンドポイント | ステータス |
|---|---|---|
| 1 | GET /source-image-groups | 200 OK |
| 2 | GET /source-image-groups/sig1/source-images | 200 OK |
| 3 | GET /common-print-groups | 200 OK |
| 4 | GET /common-print-groups/cpg1/prints | 200 OK |
| 5 | GET /files/url/{fileKey+} | 200 OK |

全API正常応答を確認。

---

## 事象の修正確認

| 事象 | 内容 | 結果 |
|---|---|---|
| 事象1 | 仕様書に記載のないフィールドが返却されている | **解消** — `allowedEventFields` / `disallowedEventFields` / `allowedUploadExtensions` が返却されなくなった |
| 事象2 | 既に期限切れの署名付きURLが返却されている | **解消** — 全件CloudFront公開URL（期限なし）で返却されている |
| 事象3 | thumbnailUrlが署名付き一時URLになっている | **解消** — CloudFront公開URL（`https://d3p6nztoczy84t.cloudfront.net/...`）で返却されており、Bubble DBに保存して再利用可能 |
