# Sunny プロジェクト概要
日照率算出Webアプリ。単一HTMLファイル（index.html）。
GitHub Pagesでデプロイ済み。
URL: https://kokingbeach-netizen.github.io/sunny-app/

## 技術構成
- Vanilla JS / SunCalc.js / Chart.js
- 単一ファイル構成（index.html）
- サブページ：sunny_how_it_works.html

## 実装済み機能
- 緯度経度入力（Decimal・DMS両形式対応）
- 窓の向き選択（8方位・コンパス配置）
- 用途選択（室内・バルコニー・ルーフバルコニーの3択）
- 階数・建物情報の入力
- 写真アップロード（3方向・SVF解析）
- 月別日照率ヒートマップ（窓別表示）
- PROPERTY SCORE（OR計算による総合スコア）
- 季節別スコア（春夏秋冬・カウントアップアニメーション）
- 最も日照が多い月（上位2ヶ月）
- 用途別・向き別・季節性を考慮したAIコメント
- 窓別詳細表示（季節スコア＋ヒートマップ＋コメント縦並び）
- 樹木補正（常緑樹・落葉樹の季節別補正）
- 必須項目バリデーション（緯度経度・用途・窓の向き）
- 遮蔽物ラベルが窓の向きに連動して自動変更

## 用途別計算ロジック
- 室内（living）：obstAngleをそのまま適用
- balcony/veranda：obstAngle × 0.5
- rooftop：obstAngle = 0（遮蔽なし）

## PROPERTY SCOREの計算方法
各時間スロットで「いずれかの窓で日照あり」のOR計算。
単純平均ではなく、複数窓の相乗効果を反映。

## sunny_how_it_works.html の構成
h1：自然界の法則と、あなたの家の周りの遮蔽物から算出しています。
h2①：自然界の法則（緯度・地軸の傾き・南中高度）
h2②：あなたの家の情報（階数・窓の向き/写真）
h2③：遮蔽物（向かいの建物の高さ・距離）
EXAMPLE：計算の流れ（STEP1〜3・判定表）
ENGINE：SunCalc.js
ACCURACY：精度と限界
SOURCE：使用技術・出典

## 直近のTODO
- 実機テストの積み上げ（精度向上）
- 日差しアニメーション（1日の日差しの動きを可視化）
- SUUMOへのBtoB提案準備
