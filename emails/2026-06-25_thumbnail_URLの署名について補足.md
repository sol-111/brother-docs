# thumbnail_URLの署名について補足

---

**日時:** 2026-06-25 11:36  
**送信者:** "清野奨稀" <seino@ibis.ne.jp>  
**宛先:** kazumi.sudo@brother.co.jp, yuko.miyata2@brother.co.jp, ming.liu@brother.co.jp, 他+2件  
**Cc:** 小野靖史, 山本樹輝  

ブラザー工業株式会社 須藤さま 宮田さま 柳さま 奥野さま 坂野さま

お世話になっております
株式会社アイビスの清野です

先日の定例で会話した「thumbnail_URL」の署名について補足です
渡邉さんにこのまま共有していただきたいです

▼ 結論
署名をつける場合、生成した原稿の「thumbnail_URL」だけでお願いしたい

○ 詳細
お客さんの情報が含まれるのは、生成した原稿のみのため
・行事情報
・連絡先

▼ 補足
○「thumbnail_URL」種類
・デザインテンプレート（SourceImageTable）
・共通印刷物グループ（CommonPrintGroupTable）
・生成した原稿（GeneratedImageTable）

○「thumbnail_URL」利用画面
利用者
・原稿一覧_詳細（新規作成 / 編集）
・原稿一覧_詳細（原稿生成）
・共通印刷物一覧

管理者
・デザインテンプレート
・行事テンプレート
・共通印刷物管理

○ 署名対象「thumbnail_URL」
画面
・原稿一覧_詳細（原稿生成）

API
・POST /generated-images
・GET /generated-images/{generatedImageId}
・POST /generated-images/{generatedImageId}/regenerate

DB
・GeneratedImageTable

引き続きご対応のほどよろしくお願いいたします
