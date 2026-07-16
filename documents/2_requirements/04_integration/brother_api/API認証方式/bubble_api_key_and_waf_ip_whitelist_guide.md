# Bubble側担当者向け API Key設定 / WAF IP WhiteList 連携ガイド

作成日: 2026-07-06
対象環境: PoC (API Gateway + API Key必須)

## 1. 目的

この資料は、Bubble側で必要となる以下をまとめたものです。

- API呼び出し時のヘッダー設定
- API Keyの管理運用
- WAFのIP set (WhiteList) 運用時に必要な Bubble 側送信元IPの共有方法

---

## 2. API Keyヘッダー設定

### 2.1 設定内容

Bubble側の全対象APIリクエストに、以下のヘッダーを付与してください。

- 送信先ヘッダー: x-api-key
- 値: QB2A3vrPXQ8TCYWalX54j4ApbZjg1o5K248YICbW

設定形式:

- x-api-key: QB2A3vrPXQ8TCYWalX54j4ApbZjg1o5K248YICbW

### 2.2 実装上の注意

- クエリ文字列ではなく、必ずヘッダーで送信すること
- API Keyが未設定/誤設定の場合、403が返る
- API Keyは秘密情報として扱い、Bubble側の安全な設定領域で管理すること

---

## 3. API応答の想定

- 200: 正常
- 403: API Key未送信 or 不正なAPI Key

PoC検証済み:

- API Keyなし: 403
- API Keyあり: 200

---

## 4. WAFのIP set (WhiteList) 運用について

### 4.1 Bubble側サーバIPについて

重要: 本バックエンド側だけでは Bubble 側送信元IPを確定できません。

理由:

- Bubbleの実行方式 (サーバー側呼び出し / ブラウザ直接呼び出し) により送信元IPが変わる
- Bubbleプランや構成により、送信元IPが固定でない場合がある

そのため、WhiteList運用には Bubble側担当者から以下情報の提供が必要です。

- API呼び出しが Bubble サーバー経由であること
- 固定送信元IP (またはCIDR) 一覧
- 変更時の連絡手順

### 4.2 Bubble側で実施してほしい確認

1. 呼び出し方式の確認
- BubbleのServer-side action/API Connector経由で呼んでいるか
- ブラウザ直接呼び出しになっていないか

2. 送信元IPの取得
- Bubble側情報またはサポート窓口から、固定送信元IP/CIDRを取得
- 必要に応じて複数リージョン/冗長経路のIPを確認

3. バックエンド側へ共有
- 許可対象IP/CIDRを正式連絡
- 適用開始日時とテスト日時を明記

---

## 5. 共有テンプレート (Bubble担当者 -> バックエンド担当者)

件名: WAF IP WhiteList登録依頼 (PoC)

共有項目:

- 環境: PoC
- 呼び出し方式: Bubbleサーバー経由
- 許可希望IP/CIDR:
  - 203.0.113.10/32
  - 198.51.100.0/24
- 適用希望日時: 2026-07-07 10:00 JST
- 疎通確認担当

---

## 6. セキュリティ運用メモ

- WhiteListは最小CIDRで登録すること
- 将来の本番運用ではユーザー単位認証 (JWT) への移行を検討すること
