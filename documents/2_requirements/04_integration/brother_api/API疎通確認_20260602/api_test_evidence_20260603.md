# API疎通確認エビデンス

- 実施日時: 2026-06-03
- 対象環境: brother-backend-dev-yokinini
- Base URL: `https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod`

---

## 1. GET /source-image-groups

**ステータス: 200 OK**

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

---

## 2. GET /source-image-groups/sig1/source-images

**ステータス: 200 OK**

```json
{
  "items": [
    {
      "sourceImageId": "src-bubble-1780448083182",
      "size": "A4",
      "thumbnailUrl": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/source-images/sig1/src-bubble-1780448083182/thumbnail.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3NNNLTKZ2%2F20260603%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260603T005443Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEGkaDmFwLW5vcnRoZWFzdC0xIkYwRAIgQeM01rcUF%2Frc1%2FVn8NOaDN9yFWVSa%2Bx37dBKhJHKyD8CIDY6oO7GM%2FHlGIhOWMihhYETGbWacF0FZzNH4JZ7gotxKoMECDIQABoMMzMyODk2OTM4NjE0Igz6XGhvZQZaqdlhRP0q4ANE1mp1CMigpu%2FgbThKu5ukSC52WVFGUHEQaHlozCeB7KAnhWAnyI1HT%2FGmyDhGThdUHLbd9%2BzQFmw3HYidK5JO2%2BCB5U2VnMvemPqHGi6A%2Bi3cbU7XembQm5Th7UUzkg76Nn1lGIJGIrgaL16HlLVul3Qk2chdEy8dYYE2JM20myrhIA5gTxBb8gz%2BrH5I%2B17Z%2F4HI%2B6Rq6DpY2LvWJIFMYXPnaacy39HJwOEwQnljSbp7f8GVHpopy7VDnOZmUforxW0qWchzZO4WAOegHNo%2FJk5OfLNkgIw8QOiKg7eFzUxfoH0JVrADAxq4d0VX9bASyzqFvvm%2BkDHpX%2BqlOIQKSuAr4xXCuFR4THsRTgYtz%2BMEBtdTuBojuY2CVeL4%2FK7v4VQZhJIOZQsjuz%2FNDeACwaoappgz7K9bbJg2djJ%2FjwH04DeOGRge8AQDTj0ls4ospT0RDyhBOGG39fsDWtQwPgytaFZTHXQx%2B6kbuKitXn%2Bjy87LcVTCpZh54RaQGSPypwHJuICnM%2FvCKiPs1xowAJEsQ%2FgjUS0DARr8JzPaUc6WMlG5CJi%2FG%2BMLwG9NpdLNaiorwePj8ZEKPMpXiQMPobG6GdcJMtNjznZj00gDaxNMHrL%2FPPy387ZygZW8bOkwzvX90AY6qQH1r8tiycORfFG2aJxt7dvxVDO5%2BdyhnygryqvKOn9YuMBOjpvtq73SeMzun%2FFqNK9KlyR2D5z0OA5hnfrysgQdcHVweaJ7Nv8SmMEzbzGUuUaVOM0PbvHKJ6vpM4%2BZkNdnKQmI1J%2FiniUHJW7wPjfCfbHEl4eyjCSUlDwJRdQ1ChSKBfUIzpprPemf6%2Fc0CCIvbSqKigNFr5AjLR4tJHN%2FqFdWboHWnzNn&X-Amz-Signature=5af20057fd0337d78a0483dabde2cb36878d056b411ad0c35ef2b311b63d4cde&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
      "status": "active",
      "allowedEventFields": ["eventName", "eventPeriod", "eventDetails", "eventProgram", "notes"],
      "disallowedEventFields": [],
      "allowedUploadExtensions": [".jpg"]
    },
    {
      "sourceImageId": "src-bubble-1780448083467",
      "size": "A4",
      "thumbnailUrl": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/source-images/sig1/src-bubble-1780448083467/thumbnail.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3NNNLTKZ2%2F20260603%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260603T005443Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEGkaDmFwLW5vcnRoZWFzdC0xIkYwRAIgQeM01rcUF%2Frc1%2FVn8NOaDN9yFWVSa%2Bx37dBKhJHKyD8CIDY6oO7GM%2FHlGIhOWMihhYETGbWacF0FZzNH4JZ7gotxKoMECDIQABoMMzMyODk2OTM4NjE0Igz6XGhvZQZaqdlhRP0q4ANE1mp1CMigpu%2FgbThKu5ukSC52WVFGUHEQaHlozCeB7KAnhWAnyI1HT%2FGmyDhGThdUHLbd9%2BzQFmw3HYidK5JO2%2BCB5U2VnMvemPqHGi6A%2Bi3cbU7XembQm5Th7UUzkg76Nn1lGIJGIrgaL16HlLVul3Qk2chdEy8dYYE2JM20myrhIA5gTxBb8gz%2BrH5I%2B17Z%2F4HI%2B6Rq6DpY2LvWJIFMYXPnaacy39HJwOEwQnljSbp7f8GVHpopy7VDnOZmUforxW0qWchzZO4WAOegHNo%2FJk5OfLNkgIw8QOiKg7eFzUxfoH0JVrADAxq4d0VX9bASyzqFvvm%2BkDHpX%2BqlOIQKSuAr4xXCuFR4THsRTgYtz%2BMEBtdTuBojuY2CVeL4%2FK7v4VQZhJIOZQsjuz%2FNDeACwaoappgz7K9bbJg2djJ%2FjwH04DeOGRge8AQDTj0ls4ospT0RDyhBOGG39fsDWtQwPgytaFZTHXQx%2B6kbuKitXn%2Bjy87LcVTCpZh54RaQGSPypwHJuICnM%2FvCKiPs1xowAJEsQ%2FgjUS0DARr8JzPaUc6WMlG5CJi%2FG%2BMLwG9NpdLNaiorwePj8ZEKPMpXiQMPobG6GdcJMtNjznZj00gDaxNMHrL%2FPPy387ZygZW8bOkwzvX90AY6qQH1r8tiycORfFG2aJxt7dvxVDO5%2BdyhnygryqvKOn9YuMBOjpvtq73SeMzun%2FFqNK9KlyR2D5z0OA5hnfrysgQdcHVweaJ7Nv8SmMEzbzGUuUaVOM0PbvHKJ6vpM4%2BZkNdnKQmI1J%2FiniUHJW7wPjfCfbHEl4eyjCSUlDwJRdQ1ChSKBfUIzpprPemf6%2Fc0CCIvbSqKigNFr5AjLR4tJHN%2FqFdWboHWnzNn&X-Amz-Signature=81b6af09a0c76953eac3ad5637623b0ce07ec3988d53b3011f28c21286291431&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
      "status": "active",
      "allowedEventFields": ["eventName", "eventPeriod", "eventDetails", "eventProgram", "notes"],
      "disallowedEventFields": [],
      "allowedUploadExtensions": [".jpg"]
    },
    {
      "sourceImageId": "src-9612d898-76e8-4e37-ba86-7eb6a36d3708",
      "size": "A4",
      "thumbnailUrl": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/source-images/sig1/src-9612d898-76e8-4e37-ba86-7eb6a36d3708/thumbnail.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3FNY6AKWX%2F20260529%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260529T083609Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEPj%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLW5vcnRoZWFzdC0xIkcwRQIgLV5lit%2FfkvESmAqKPQpRHIUXWNFwMR9Lh5GnEb7Hw3cCIQDRpNwr%2BzhCaQI4ETtrKV%2FNlh%2FhbGPXbIkxQrplC7dL9iqJBQjC%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAAaDDMzMjg5NjkzODYxNCIM4ns56cMq8yiq69P4Kt0EfH7j1FW76vU96d%2BgA5rwMi0TW6yfSQcvW0UpWKFFknJ4Q6SClFVWQB1BV5lRARm%2BdUU3IHJzOLLl7pcCctZoGhwvDqh25%2BHkE1xWQMOZUDGYVAqX%2BtuFcY6yCjn%2BtBthec16dQJ3AZl8dympwt5V1ABH0YQ6D3nesSDf5UO9DwmCIxg27kdrYp8WRPUONVEBVF3V2YLxwy6ntAw%2FkZaD15UW6QHlwM7GmVTSUJ6obcnoqtajyLnWSZGuV40sPEBp1USC5IQTisSKxdW1DnvrSCg5BXZ%2B6DFwo4ILczP19dbhNbxTxGZnISbXGFcMX76M6xyckmNcWQogyQGt01wzCFMovMVdI%2FcNFEGTSlMOUs1UdTRc1sZOZswoJUU7JInYnYZ3Oik0NOmF2CsWNpv8SQU9kBpyFcnlQTuurQhwE4yzw%2BEuC8SGQVhVMD3s0vggDegldqoS7B%2FGrt4STelit9hAexgjHck8gULJFf86nSwoV0kxt6suPmv%2FSnQnAZO6ZRaHKM68BV0d%2BZ3jpAUXFZfOvKTOhV3BNYxJxXEpE3jFZWpl3xPNmmpovASwSwvrt3fzUYcToc%2BNfMti1Hj0wOeesPET1QSR5TfCeOout4NgnUt2BVToSFxHBW2dIYSsPW7agblu6dUtrrnT81%2Fd7tUJnay4rAh859lHD6aSFLU9abWVe%2B0up7yHq5bI%2BCc2yRegfKCqBgzcCNSksvJiwU8Gu8NPj6Xthd7mri44hKFwcQMtG816eo2hSJOe06WCeZeo7GjQtZRMWRmIwR7OcTgP0UhTsffVhdL7nLAw95%2Fl0AY6owEZy9Vocakx7OQcGw9XNIUZgLerNNnB9KGPoQbK6enKvuuwkk0vuPD5idRBb039XQ7O75zFVYmG4sb37l4sDzh%2BV07FDd%2BkHYA44nFihYNFO%2BL8MLr3A5UcMOBRtpyLHuXLLq2XYsKE2DG2mHRz24Hw1GpYFcrajGYUNPtseVfWmTo5akKQVPISXSi08V6DPo2vHqIVEJk4nXkLLQWs6pz2yH3c&X-Amz-Signature=6d6ab461d50f7f24507ddf4ab1af593a38378507e4cf3335839ad88a2b894576&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
      "status": "active",
      "allowedEventFields": ["eventName", "eventPeriod", "eventDetails", "eventProgram", "notes"],
      "disallowedEventFields": [],
      "allowedUploadExtensions": [".jpg"]
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
  "url": "https://brother-backend-dev-yokinini-imagebucket-gghw7nev1auv.s3.ap-northeast-1.amazonaws.com/common-prints/1780043657150-e7f17ed0-4355-4f1c-9d30-727c64e50b1c-do-not-touch-h1000-p.pptx?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAU3ARUFJ3IE6GYSO3%2F20260603%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20260603T062846Z&X-Amz-Expires=600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEG4aDmFwLW5vcnRoZWFzdC0xIkYwRAIgBfmXDtol1MoF99yDpAje29B7iCfZbOI1blJoy%2B7Gir4CIHq9Bp2p8GpWcJ%2FXaD8blD78eYVPHjyBX2mNE%2BB7YlJTKvcECDgQABoMMzMyODk2OTM4NjE0IgwBLFtzsZGtM%2Bf27Koq1ARx%2BI4h%2BHv%2FqB%2B8i622BhPkyc17PMY%2B9oeQXBBrzuPP2fR4q8ieCINc7RWWf7TC3%2FOe3cUuyAABTizaU1sjRtCksFbU%2BhttLhSaMxmbfC%2BzXmRWiPrHBl7W%2BwXbKZiowkFHmrhJ%2Fcl57LlHX3vzwkpFFXw1ALQkQhZ%2Fh443ENGgZMyeN2JTs1Hcdq%2FOpCRDa6ZpyTV9mvKdZRi7C0LalltHn1Mmm92L0ZAomP5YlEv6IspvPQZbkIDbv%2F4XggkPIQarOz7GY1dDLWydl2cf33cVx6%2F%2ByZWUmh4F9XCcxV6%2FOd2FRTjTTLuUc9AiDbjUC5KvatUb7ncemvt9Z9Ia0tCv6GrcfWmkqspO3wKiuNhKRoEaxvS1H7TM71Uf3NnC7Q3sgh3oyrsPbKTm1ysyKpQBhD%2Fkvj2D6zxvW9Yu8gi8ye57xe2IVZ9qg%2BAKZRAMGJ8XjhsK5kRs6hPLDFgisPe5GX0%2Fj%2BtcyRoBpQMGjbkHPJMYlq6n1ncgbzfPDoBzYNx2eeGKUaAMuBP7IsJnHhRrThRBzdcxoWYlJwwSAvclDtMM%2BAyEGE35kxox3XkOq%2BuSQ%2Fm4cJzCSPeXNOwb9w7r8340s0h8t7OVqKxTZF2uV1b7jt8aIA2qmCTMfzhSwA3GOrumVvyx0NJNDQJS6yFXqpy%2F3wnmXmpKjM2SfoPXWYT4hP%2B%2Fx%2Fb0sWuBF34%2FRkDqZ%2Fii0B2jrsmkybsub5HmBYPeb9LAkVY0AHhA4UvBLFkzmTbj0tc%2FzNHCzDxV0%2Fl5A8KwViLOIdvnkfIiPbnAqEIizDCdk%2F%2FQBjqkAbakxHsPNtl%2BPNdJ0PMIetiOHxStWT239QkXnVwZYVgEO3uxznWH4TsAKVtqw1anhr8U%2FaBA5OaeVL5tzVV8YnYcUmSFM7L7LA2wdGZjlXHnZQDV9b84VVqDJh93yfhR0XLX1%2F06mHR6x2hTQWImBb82mhdv7iWM9%2F4zgMysjMru5kUME%2FxomQB%2FxdyyvvFyGFg%2FEWu3v9T7mXi5pLhC5GMA7UM3&X-Amz-Signature=0899fefccd3445f614bfbd06902bdb5b114423ca543b56b6f8ff64e7e99ba93a&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
  "expiresAt": "2026-06-03T06:38:46.361Z"
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
