# API疎通確認コマンド

Base URL: `https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod`

## 1. ソースイメージグループ一覧

```bash
curl -s https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/source-image-groups | jq .
```

## 2. ソースイメージ一覧（グループ指定）

```bash
curl -s https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/source-image-groups/sig1/source-images | jq .
```

## 3. 共通印刷物グループ一覧

```bash
curl -s https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/common-print-groups | jq .
```

## 4. 共通印刷物一覧（グループ指定）

```bash
curl -s https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/common-print-groups/cpg1/prints | jq .
```

## 5. ファイルダウンロードURL取得

```bash
curl -s https://u7ib5ltbdd.execute-api.ap-northeast-1.amazonaws.com/Prod/files/url/common-prints/1780043657150-e7f17ed0-4355-4f1c-9d30-727c64e50b1c-do-not-touch-h1000-p.pptx | jq .
```
