# Brother - 行事デザイン自動生成サービス（POC）

業界（お寺・宗派等）ごとの行事に合わせた印刷物デザインを AI で自動生成する Web サービスの POC 開発ドキュメントです。

## ディレクトリ構成

```
documents/
├── 0_sales/                            # 営業・内部MTG
│   ├── internal_meeting_20260413.txt
│   └── summary.md
│
├── 1_meeting_minutes/                  # 定例議事録
│   └── 議事録_ゼロイチスタート_定例会_YYYYMMDD.pdf  (9件)
│
├── 2_requirements/                     # 要件定義
│   ├── 00_overview/                    #   プロジェクト概要
│   │   ├── project_summary.md
│   │   └── glossary.md
│   │
│   ├── 01_functional/                  #   機能要件
│   │   ├── *.csv                       #     画面別 機能一覧・詳細 (15件)
│   │   ├── 行事情報バリデーション.md      #     行事情報の必須/任意
│   │   └── 企業情報バリデーション.md      #     企業情報の必須/任意
│   │
│   ├── 02_flow/                        #   フロー図（予約）
│   │
│   ├── 03_email/                       #   メール設計（予約）
│   │
│   ├── 04_integration/                 #   外部連携
│   │   └── brother_api/
│   │       ├── api_specification_summary.md
│   │       ├── api_questions_20260416.md
│   │       └── API仕様_シーケンス_20260417-selected/
│   │           ├── API_20260417.pdf     #     ブラザー提供API仕様
│   │           ├── sequenceDiagram.md   #     シーケンス図
│   │           ├── genAISequenceDiagram.md
│   │           └── 論点一覧.yaml         #     API設計の論点
│   │
│   └── 05_db/                          #   DB設計・マスタデータ
│       ├── 業界一覧.md                   #     業界マスタ（大分類・小分類）
│       └── デザインサイズ一覧.md           #     デザインサイズ一覧（7サイズ）
│
├── 3_test/                             # テスト
│   ├── テスト一覧.csv                   #   テストケース一覧
│   └── IT-01 ~ IT-13_*.csv            #   結合テストケース (13件)
│
└── 99_receives/                        # クライアント向け確認資料
    └── page/                           #   確認用HTML
        ├── index.html
        ├── design_template_questions.html
        └── search_engine_elements.html

docs/                                   # GitHub Pages 公開用
├── index.html                          #   タブナビゲーション（TOP）
└── api_spec.html                       #   API仕様書（サイドバーナビ）
```

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| フロントエンド | Bubble（ノーコード） |
| 画像生成 | ブラザー AI チーム API |
| 連携方式 | Bubble → トリガーAPI → 画像生成API |

## 主要機能

- 企業アカウント管理（メール + パスワード認証）
- 行事管理（作成・編集・複製・削除）
- デザイン自動生成（A3 / A4 / 長尺縦横、各3パターン）
- デザイン評価（五段階）
- パワーポイント形式ダウンロード
- 操作ログ CSV 出力
- 共通印刷物・ノベルティ紹介
- メールリマインダー（2ヶ月前・1ヶ月前）

## スケジュール

| フェーズ | 期間 |
|---------|------|
| 要件定義 | 2月中旬 〜 3月中 |
| 開発 | 3月中旬 〜 4月末 |
| 結合テスト | 5月（3週間） |
| 受け入れテスト | 5月（2週間） |
| リリース | 5月末目標 |

## クライアント向け確認ページ

https://sol-111.github.io/brother-docs/

## 関係者

- **クライアント**: ゼロイチスタート（小野、清野、山本）
- **BIL**: SPDEV 柳 / PA 奥野 / DG 宮田・須藤
