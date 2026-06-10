# API疎通確認コマンド

Base URL: `https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod`

## 既存API（前回テスト済み）

### 1. ソースイメージグループ一覧

```bash
curl -s https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/source-image-groups | jq .
```

### 2. ソースイメージ一覧（グループ指定）

```bash
curl -s https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/source-image-groups/sig3/source-images | jq .
```

### 3. 共通印刷物グループ一覧

```bash
curl -s https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/common-print-groups | jq .
```

### 4. 共通印刷物一覧（グループ指定）

```bash
curl -s https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/common-print-groups/cpg1/prints | jq .
```

### 5. ファイルダウンロードURL取得

```bash
curl -s https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/files/url/common-prints/1780043657150-e7f17ed0-4355-4f1c-9d30-727c64e50b1c-do-not-touch-h1000-p.pptx | jq .
```

## 生成系API（今回新規テスト）

### 6. 画像生成（長尺縦サイズ）

```bash
curl -s -X POST https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/generated-images \
  -H "Content-Type: application/json" \
  -d '{
    "sourceImageId": "src-f366749e-02cb-429b-bddb-e15ca25f6117",
    "eventName": "ブラザー夏祭り盆踊り会",
    "eventPeriod": "2026年8月23日"
  }' | jq .
```

### 7. 画像生成（A3サイズ - 全フィールド）

```bash
curl -s -X POST https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/generated-images \
  -H "Content-Type: application/json" \
  -d '{
    "sourceImageId": "src-b9bfb2fc-0b50-4137-81d5-797969548444",
    "eventName": "ブラザー夏祭り盆踊り会",
    "eventPeriod": "2026年8月23日",
    "eventDetails": "境内にて夏祭りを開催いたします",
    "eventProgram": "10:00 開会 / 12:00 盆踊り / 15:00 閉会",
    "notes": "雨天時は中止となります"
  }' | jq .
```

### 8. 画像生成（A4サイズ - 全フィールド）

```bash
curl -s -X POST https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/generated-images \
  -H "Content-Type: application/json" \
  -d '{
    "sourceImageId": "src-bubble-1780448083182",
    "eventName": "ブラザー夏祭り盆踊り会",
    "eventPeriod": "2026年8月23日",
    "eventDetails": "境内にて夏祭りを開催いたします",
    "eventProgram": "10:00 開会 / 12:00 盆踊り / 15:00 閉会",
    "notes": "雨天時は中止となります"
  }' | jq .
```

### 9. 生成状態確認（ポーリング）

```bash
curl -s https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/generated-images/{generatedImageId} | jq .
```

### 10. 生成画像のDL URL取得

```bash
curl -s https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/files/url/{fileKey+} | jq .
```

### 11. 再生成（長尺縦 - eventName/eventPeriodのみ）

```bash
curl -s -X POST https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/generated-images/{generatedImageId}/regenerate \
  -H "Content-Type: application/json" \
  -d '{
    "eventName": "ブラザー記念夏祭り工場見学会",
    "eventPeriod": "2026年8月23日〜24日"
  }' | jq .
```

### 12. 再生成（A3 / A4 - 全フィールド）

```bash
curl -s -X POST https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/generated-images/{generatedImageId}/regenerate \
  -H "Content-Type: application/json" \
  -d '{
    "eventName": "ブラザー記念夏祭り工場見学会",
    "eventPeriod": "2026年8月23日〜24日",
    "eventDetails": "工場敷地内にて夏祭りと工場見学会を開催",
    "eventProgram": "10:00 工場見学 / 12:00 昼食 / 14:00 盆踊り / 16:00 閉会",
    "notes": "雨天決行・荒天中止"
  }' | jq .
```
