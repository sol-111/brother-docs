# WAF IP WhiteList運用に関する回答（Bubble側送信元IPについて）

- 作成日: 2026-07-08
- 作成: 株式会社アイビス
- 対象: 「Bubble側担当者向け API Key設定 / WAF IP WhiteList 連携ガイド」4章（Bubble側で実施してほしい確認）への回答

## 1. 要約

Bubbleの通常プランでは**固定の送信元IP（またはCIDR）の提供は困難です**。
現状のままではWAFのIP WhiteList運用に必要なIP一覧の提供が難しいため、本書にて調査結果と代替となる構成のご提案を共有いたします。

## 2. ご依頼事項への回答

### 2.1 呼び出し方式の確認

- API呼び出しはBubbleの**API Connector（サーバーサイド）経由**で行います。
- ブラウザからの直接呼び出しは行いません。

### 2.2 送信元IPの取得

調査の結果、以下のとおり**固定送信元IPのご提供は困難**です。

- Bubbleの通常プラン（Free / Starter / Growth / Team）では、外向きAPIリクエストの送信元はAWS上の**動的なIPプール**から割り当てられるため、予告なく変動します
- Bubbleが公式に静的IPを提供するのは **Dedicated instance（専用インスタンス、参考価格 $3,000/月〜）** のみです

参考情報:

- [Bubble公式ドキュメント: Dedicated instance](https://manual.bubble.io/help-guides/bubble-for-enterprise/hosting-and-infrastructure/dedicated-instance)（"A dedicated instance provides your apps with a static IP"）
- [Bubbleフォーラム: How to get Bubble's IP(s) to whitelist](https://forum.bubble.io/t/how-to-get-bubbles-ip-s-to-whitelist/255684)（静的IPの取得手段はDedicatedのみとの回答）

## 3. 代替構成のご提案

IP WhiteListの目的（意図しない第三者からのAPI呼び出しの防止）を、送信元IPに依存しない方法で実現する案として、以下をご検討いただけますと幸いです。

### 比較サマリ

| | 案1: API Key＋標準機能 | 案2: 固定IPプロキシ | 案3: 中継サーバー（BFF） |
|---|---|---|---|
| IP WhiteList | 適用しない | 適用可能 | 中継→貴社API間で適用可能 |
| 追加費用（目安） | なし | $19/月〜 | $35〜40/月＋構築費 |
| 導入時期 | 即時（現行構成のまま） | 数日 | 要設計・構築（本番向け） |
| 通信経路 | Bubble→貴社API（直接） | **第三者プロキシを経由** | Bubble→中継→貴社API |
| API Keyの配置 | Bubble側 | Bubble側 | 中継サーバー側（Bubbleに置かない） |

### 案1: API Key認証＋WAF/API Gatewayの標準機能で運用（PoC期間の推奨案）

| 対策 | 内容 |
|---|---|
| API Key認証 | 現行どおり（ガイド1〜3章） |
| レートリミット / 使用量プラン | API Gatewayの標準機能。万一のキー漏洩時も大量呼び出しを制限可能 |
| API Keyの定期ローテーション | キー漏洩時の影響期間を限定。ローテーション手順・頻度は別途ご相談 |

### 案2: 固定IPプロキシの導入

QuotaGuard等の静的IPプロキシサービス（参考価格 $19/月〜）をBubbleのAPI Connectorに設定し、送信元を固定IP化する方式です。APIリクエスト（API Keyを含む）が外部事業者のプロキシサーバーを経由するため、貴社セキュリティ基準での評価が必要です。

### 案3: 中継サーバー（BFF）方式

将来のJWT移行（ガイド6章）とあわせた本番向けの構成案です。弊社または貴社のAWS上に中継サーバーを構築します（VPC内のLambda等＋NAT Gateway＋Elastic IPで送信元を固定IP化する構成を想定）。

- Bubble → 中継サーバー: ユーザー単位認証（JWT）で保護
- 中継サーバー → 貴社API: 中継サーバーは固定IPを持つため、**この区間でWAF IP WhiteListが機能**
- API Keyは中継サーバー側で保持し、Bubble側にキーを配置しない構成が可能

補足: 貴社API GatewayにJWT Authorizerを実装いただける場合、中継サーバーを介さずユーザー単位認証を実現する構成も考えられます。本番設計の際にあわせてご相談させてください。

## 4. 各案のメリット・デメリット一覧

| 案 | メリット | デメリット |
|---|---|---|
| **案1: API Key＋標準機能**（PoC推奨） | ・追加費用・追加構築なしで即時運用可能（貴社側のWAF/API Gateway設定のみ）<br>・通信経路に第三者が入らない | ・送信元IPによる制限は行わないため、API Key漏洩時はレートリミットの範囲でしか防げない（ローテーションで影響期間を限定） |
| **案2: 固定IPプロキシ** | ・インフラ構築なし・設定のみで固定IPを実現でき、IP WhiteListをそのまま適用可能<br>・導入が最も早い（数日程度） | ・API Keyを含む通信が外部事業者のプロキシを経由（HTTPSのまま転送されるが、経路に第三者が介在）<br>・プロキシ事業者の障害がAPI連携全体の障害になる<br>・転送量に応じた従量課金 |
| **案3: 中継サーバー（BFF）**（本番向け） | ・API Keyを中継側で保持でき、Bubbleにキーを置かない<br>・IP WhiteListとユーザー単位認証（JWT）を両立でき、本番のセキュリティ要件に最も適合<br>・通信経路がすべて自社・貴社管理下に収まる | ・設計・構築の工数と期間が必要（PoC期間中の導入には不向き）<br>・インフラ費用（NAT Gateway等で$35〜40/月程度）と運用・保守体制が必要 |

## 5. お願い事項

- 上記を踏まえ、PoC期間中のWAF IP WhiteList適用可否についてご判断・ご教示ください
- なお、弊社からの疎通確認（開発・検証作業）は弊社オフィスから直接APIを呼び出しています。
- WhiteListを有効化される場合は、**弊社の固定IPの登録についても別途ご相談させてください**（適用日時の事前共有をお願いいたします）
