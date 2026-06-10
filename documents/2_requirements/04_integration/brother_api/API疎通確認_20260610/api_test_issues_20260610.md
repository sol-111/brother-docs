# API疎通確認 - 確認事項まとめ

- 実施日時: 2026-06-10
- 対象環境: brother-backend-dev-yokinini
- 対象API: 生成系API（POST /generated-images, GET /generated-images/{id}, POST /regenerate）+ 既存GET系API

---

## 事象1: 再生成時にリクエストに含めていない文言が出力される

再生成（POST /regenerate）の結果画像に、リクエストで送信していない文言が含まれている。初回生成（POST /generated-images）では発生しない。

### 送信したリクエスト内容（再生成時）

```json
{
  "eventName": "ブラザー記念夏祭り工場見学会",
  "eventPeriod": "2026年8月23日〜24日",
  "eventDetails": "工場敷地内にて夏祭りと工場見学会を開催",
  "eventProgram": "10:00 工場見学 / 12:00 昼食 / 14:00 盆踊り / 16:00 閉会",
  "notes": "雨天決行・荒天中止"
}
```

### 実際の出力画像に含まれる文言

| サイズ | リクエストにない文言 |
|---|---|
| A3 | 「盂蘭盆会法要」が中央に大きく表示 |
| A4 | 「兄弟合主幹 堀田 太郎 師 演題:「つながる命、感謝の心」」が追加 |
| 長尺縦 | 「記念会り法見要会」（文字化け的な合成） |

### 補足

- 初回生成では全サイズとも正しい文言で出力されている
- 再生成時のみ全サイズで発生

---

## 事象2: contactName / contactDetails が未実装

仕様書（docs/api_spec.html / docs/flow_mock.html）に記載されている `contactName`（連絡先名）と `contactDetails`（連絡先詳細）が、リリースガイドおよび実際のAPIリクエストパラメータに含まれていない。

2026-06-10の定例会にて、連絡先・連絡先詳細を生成リクエストに含める方針が決定済み。

### 対応依頼

- POST /generated-images および POST /generated-images/{id}/regenerate のリクエストボディに `contactName` / `contactDetails` を追加いただきたい

---

## 事象3: A3サイズの生成結果が長尺縦と同じレイアウトになっている

`src-b9bfb2fc-0b50-4137-81d5-797969548444`（size: A3）で生成した画像が、長尺縦（`src-f366749e-02cb-429b-bddb-e15ca25f6117`）の生成結果とほぼ同じ縦長レイアウトで出力されている。A3（横長もしくはそれに近いアスペクト比）の見た目ではない。

### 生成結果の比較

| サイズ | sourceImageId | 生成結果 |
|---|---|---|
| A3 | src-b9bfb2fc-... | 縦長レイアウト（長尺縦と同様） |
| 長尺縦 | src-f366749e-... | 縦長レイアウト |

### 想定される原因

- dev環境のA3用ソースイメージに、長尺縦と同じ（もしくは類似の）テンプレートが登録されている
- ソースイメージの紐づけ誤り

---

## 事象4: dev環境にソースイメージが3サイズ分しか登録されていない

仕様上は6サイズ（A4, A3, 長尺縦, 長尺横, ハガキ, 封筒 長形3号）が存在するが、dev環境には以下の3サイズしか登録されていない。

| サイズ | グループ | 状態 |
|---|---|---|
| A4 | sig1 | 登録あり |
| A3 | sig3 | 登録あり（ただし事象3の問題あり） |
| 長尺縦 | sig3 | 登録あり |
| 長尺横 | — | **未登録** |
| ハガキ | — | **未登録** |
| 封筒 長形3号 | — | **未登録** |

---

## 事象5: サイズ別入力制約が仕様と異なるフィールド名で実装されている

リリースガイド（bubble-generated-ai-api-pre-release-guide_20260608.md）および仕様書（docs/api_spec.html / docs/flow_mock.html）に記載のリクエストフィールドと、実際のバリデーションエラーメッセージに含まれるフィールド名が異なる。

### 仕様書のフィールド名

| フィールド | 用途 |
|---|---|
| eventName | 行事名 |
| period | 期間 |
| details | 詳細 |
| program | プログラム |
| notes | 備考 |
| contactName | 連絡先名 |
| contactDetails | 連絡先詳細 |

### リリースガイドのフィールド名

| フィールド | 用途 |
|---|---|
| eventName | 行事名 |
| eventPeriod | 期間 |
| eventDetails | 詳細 |
| eventProgram | プログラム |
| notes | 備考 |

### 実際のバリデーションエラー

```json
{
  "errorCode": "INVALID_REQUEST",
  "message": "size=長尺縦 does not allow fields [eventDetails, eventProgram, notes]. Allowed disallowed set: [eventDetails, eventProgram, notes]"
}
```

### 差異まとめ

| 仕様書 | リリースガイド / 実装 |
|---|---|
| period | eventPeriod |
| details | eventDetails |
| program | eventProgram |

---

## 事象6: POSTレスポンスのstatusが仕様と異なる

仕様書（docs/api_spec.html / docs/flow_mock.html）ではPOSTレスポンスのstatusは `PENDING` または `RUNNING` と記載されているが、実際には `done` が返却されている。

### 仕様書

```
POST /generated-images → status: "PENDING" or "RUNNING"
```

### リリースガイド

```
POST /generated-images → status: "done"
※ done は受付完了であり生成完了ではありません
```

### 実際のレスポンス

```json
{
  "generatedImageId": "gen-cc78c82f-7bf4-4984-94db-002e6beacbc8",
  "sourceImageId": "src-f366749e-02cb-429b-bddb-e15ca25f6117",
  "status": "done"
}
```

---

## 事象7: GETポーリングのstatusが仕様と一部異なる

仕様書ではポーリング中のstatusは `RUNNING` と記載されているが、実際には `PROCESSING` が返却されている。

### 仕様書

```
GET /generated-images/{id} → status: "RUNNING" → "COMPLETED" or "FAILED"
```

### リリースガイド

```
GET /generated-images/{id} → status: "PROCESSING" → "COMPLETED" or "FAILED"
```

### 実際のレスポンス

```json
{
  "status": "PROCESSING"
}
```

---

## 確認事項サマリ

| # | 事象 | 重要度 | 対応 |
|---|---|---|---|
| 1 | 再生成時にリクエストにない文言が出力される | 高 | 初回生成では正常。再生成時のみ全サイズで発生 |
| 2 | contactName / contactDetails が未実装 | 中 | 6/10定例で追加決定済み。リクエストパラメータへの追加を依頼 |
| 3 | A3の生成結果が長尺縦と同じレイアウト | 中 | dev環境のソースイメージの紐づけ確認を依頼 |
| 4 | dev環境に3サイズしか登録されていない（長尺横/ハガキ/封筒 長形3号が未登録） | 中 | ソースイメージの追加登録を依頼 |
| 5 | フィールド名が仕様書と異なる（period→eventPeriod等） | 低 | リリースガイドのフィールド名に準拠する |
| 6 | POSTレスポンスのstatusが `done`（仕様は `PENDING`/`RUNNING`） | 低 | リリースガイドに記載あり。`done` を受付完了として扱う |
| 7 | ポーリング中のstatusが `PROCESSING`（仕様は `RUNNING`） | 低 | リリースガイドに記載あり。`PROCESSING` をポーリング継続判定に使用 |
