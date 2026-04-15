# Sunny プロジェクト概要
日照率算出Webアプリ。単一HTMLファイル（index.html）。
GitHub Pagesでデプロイ済み。
URL: https://kokingbeach-netizen.github.io/sunny-app/

## 技術構成
- Vanilla JS / SunCalc.js / Chart.js
- 単一ファイル構成（index.html）
- サブページ：sunny_how_it_works.html・sunny_privacy.html・sunny_terms.html・sunny_contact.html

## ローカル開発
- 起動：`python3 -m http.server 8080`
- 確認：http://127.0.0.1:8080/index.html
- 停止：`lsof -ti:8080 | xargs kill`
- デプロイ：`git add . → git commit -m "..." → git push`

## 画像ファイル一覧
- sunnylady.png：TOPヒーローセクションの女性イラスト
- hand.png：撮影方法ガイド（横向き45°上向き）
- windowshoot.png：室内からの撮影ガイドイラスト
- balconyshoot.png：バルコニーからの撮影ガイドイラスト
- rooftopshoot.png：ルーフバルコニーからの撮影ガイドイラスト
- landshoot.png：建築前リサーチ（更地）の撮影ガイドイラスト
- sunorbits_2.png：太陽軌道図（how_it_works用）
- sunnyinn_2.png：遮蔽角度説明図（how_it_works用）

## 実装済み機能
- 緯度経度入力（Decimal・DMS両形式対応）
- 現在地取得ボタン（Geolocation API・enableHighAccuracy:true）
- 住所自動入力（Nominatim逆ジオコーディング・町名レベル）
- 窓の向き選択（8方位・コンパス配置）
- 用途選択（室内・バルコニー・ルーフバルコニーの3択）
- 用途選択後に屋根・庇の奥行き選択が表示（〜1m・1〜1.5m・1.5m〜）
- 用途別撮影ガイド（hand.png＋用途別イラストの横並び表示）
- 階数・建物情報の入力
- 写真アップロード（3方向・SVF解析）
- 月別日照率ヒートマップ（窓別表示）
- PROPERTY SCORE（OR計算による総合スコア）
- 季節別スコア（春夏秋冬・カウントアップアニメーション）
- 最も日照が多い月（上位2ヶ月）
- 用途別・向き別・季節性を考慮したコメント
- 窓別詳細表示（季節スコア＋ヒートマップ＋コメント縦並び）
- 樹木補正（常緑樹・落葉樹の季節別補正）
- 必須項目バリデーション（緯度経度・用途・窓の向き）
- 遮蔽物ラベルが窓の向きに連動して自動変更
- 結果URLコピー・スクリーンショット表示
- 建築前リサーチモード（方角別・更地撮影）
- フッター（プライバシーポリシー・利用規約・お問い合わせへのリンク）
- 入力完了オレンジ枠フィードバック（緯度・経度・住所・階数・用途・窓の向き・遮蔽物選択）
- 写真カードの枠ルール改善（未追加：枠なし、追加済み：オレンジ枠）
- 正方形判定カードデザイン（サマリー＋窓別・物件資料への貼り付け対応）
- 窓1枚の場合はサマリーカード1枚のみ表示（総合＋窓コメント統合）
- 窓2枚以上の場合はサマリー＋窓別カードを自動生成
- 判定結果・スクリーンショット画面を同一カードデザインで統一
- 前向きコメント自動生成（スコア・窓の向き別）
- スマホ入力時のiOS自動ズーム防止（font-size:16px統一）

## 用途別計算ロジック
- 室内（living）：obstAngleをそのまま適用
- balcony/veranda：obstAngle × 0.5
- rooftop：obstAngle = 0（遮蔽なし）

## 屋根・庇の補正ロジック
roofDepth（メートル）が0より大きい場合：
roofObstAngle = arctan(roofDepth ÷ 2.3) ※天井高2.3m
obstBase = Math.max(obstBase, roofObstAngle)
ルーフバルコニーは補正なし（roofDepth=0固定）

## PROPERTY SCOREの計算方法
各時間スロットで「いずれかの窓で日照あり」のOR計算。
単純平均ではなく、複数窓の相乗効果を反映。

## 遮蔽物までの距離の選択肢（value値）
- 接近 〜5m：value=3
- 近い 5〜10m：value=7
- やや近い 10〜20m：value=15（デフォルト）
- 普通 20〜35m：value=27
- 遠い 35m〜：value=45

## チェックボックス項目（写っているものを選択）
- 樹木（常緑樹・1年中葉がある）→ 全季節-15%
- 樹木（落葉樹・冬に葉が落ちる）→ 冬-5% 秋-10% 春夏-15%
- 看板・広告板 → 遮蔽として計算（+8°）
- 隣接する建物の壁面（真横）→ 遮蔽として計算（+20°）
- 上記に該当するものは写っていない
※電柱・電線は計算に影響しないため削除済み

## sunny_how_it_works.html の構成
h1：自然界の法則と、あなたの家の周りの遮蔽物から算出しています。
h2①：自然界の法則（緯度・地軸の傾き・南中高度）
h2②：あなたの家の情報（階数・窓の向き/写真）
h2③：遮蔽物（向かいの建物の高さ・距離）
EXAMPLE：計算の流れ（STEP1〜3・判定表）
ENGINE：SunCalc.js
ACCURACY：精度と限界
SOURCE：SunCalc.js・Sky View Factor・Nominatim

## ライセンス確認済み
- SunCalc.js：BSDライセンス・商用利用OK
- Sky View Factor（Oke 1981）：学術概念の参照・問題なし
- Nominatim（OpenStreetMap）：ODbLライセンス・逆ジオコーディングのみ使用

## 直近のTODO
- ドメイン取得（sunnyin.com / sunnyin.jp 候補）
- PWA化（manifest.json + Service Worker）
- FAQページ作成
- 3ページのフッター追加（sunny_privacy・sunny_terms・sunny_contact）
- 日差しアニメーション（1日の日差しの動きを可視化）
- データベース導入検討（Supabase等）
- SUUMOへのBtoB提案準備
- Google Map緯度経度取得方法モーダルの中身を作成（UIのみ実装済み・内容未記入）
- 建築前リサーチモードの常緑樹補正（全季節-15%）を確認・修正
- BtoB向けランディングページの作成

