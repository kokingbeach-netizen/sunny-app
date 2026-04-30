# Sunny — 開発リファレンス

## 1. プロジェクト概要

| 項目 | 内容 |
|------|------|
| アプリ名 | SUNNY — 日当たり算出ツール |
| 種別 | 日照率算出 Web アプリ（無料・ブラウザのみ） |
| 構成 | 単一 HTML ファイル（index.html）+ サブページ |
| ライブラリ | SunCalc.js 1.9.0（BSD） / Chart.js 4.4.0 / Noto Sans JP |
| デプロイ | GitHub Pages |
| 本番 URL | https://kokingbeach-netizen.github.io/sunny-app/ |
| 将来目標 | 不動産プラットフォーム（SUUMO 等）への BtoB 組み込み |

---

## 2. ファイル構成

```
index.html              メインアプリ（全ロジック・UI を含む単一ファイル）
sunny_lp.html           BtoB 向けランディングページ
sunny_how_it_works.html 計算手法の解説ページ
sunny_contact.html      お問い合わせページ
sunny_privacy.html      プライバシーポリシー
sunny_terms.html        利用規約

sunnylady.png           ヒーローセクション女性イラスト
hand.png                撮影ガイド（横向き45°上向き）
windowshoot.png         室内からの撮影ガイド
balconyshoot.png        バルコニーからの撮影ガイド
skybalconyshoot.png     ルーフバルコニーからの撮影ガイド
landshoot.png           更地・土地からの撮影ガイド
sunorbits_2.png         太陽軌道図（how_it_works 用）
sunnyinn_2.png          遮蔽角度説明図（how_it_works 用）
```

---

## 3. ローカル開発環境

```bash
# 起動
python3 -m http.server 8080

# 確認
open http://127.0.0.1:8080/index.html

# 停止
lsof -ti:8080 | xargs kill

# デプロイ
git add . && git commit -m "..." && git push
```

---

## 4. 主要ステート変数

`index.html` の `<script>` 冒頭（948行目付近）で `var` 宣言されている。

| 変数名 | 型 | 説明 |
|--------|----|------|
| `windowStates` | Array | 各窓オブジェクトの配列。`makeDefaultWindow(i)` で生成 |
| `selectedYoto` | string | 用途（`'living'` / `'balcony'` / `'rooftop'`） |
| `roofDepth` | number | 屋根・庇の奥行き（m）。`selRoofDepth()` で更新 |
| `selectedMode` | string | 現在未使用（将来用。`'A'` 固定） |
| `selectedUsecase` | string | `'物件・室内'` または `'土地・屋外'`（初期値：`'物件・室内'`） |
| `lastSeasonScores` | object\|null | 最後に計算した季節スコア `{spring, summer, autumn, winter}` |
| `constructionResults` | Array | 土地・屋外モードの計算結果 `[{floor: N, data: {south,east,west,north}}]` |
| `selectedConstructionFloor` | number | 現在表示中の階数タブ（土地・屋外モード） |
| `storedHeatmaps` | Array | 物件・室内モードの計算済みヒートマップ（スクショ再描画用） |
| `cState` | object | 土地・屋外モードの方角別入力 `{south, southeast, southwest, east, west, north}` 各要素: `{bldH, bldDist, surroundings[], svf?}` |
| `cAddedDirs` | Array | 追加済みの方角キーリスト（土地・屋外モード） |

### windowState オブジェクト構造

```js
{
  name: '窓1',          // 自動生成: autoNameWindow() で「南の窓・大サイズ」形式に更新
  dir: '',              // '南'|'東'|'西'|'北'|'南東'|'南西'|'北東'|'北西'
  size: '中',           // '小'|'中'|'大'
  surroundings: [],     // ['evergreen'|'deciduous'|'signboard'|'wall'|'none']
  bldH: 0,              // 正面建物高さ (m)
  bldDist: 15,          // 正面建物距離 (m)
  bldDistSelected: false,
  photos: { left:null, center:null, right:null },  // base64 or null
  svf:    { left:null, center:null, right:null },  // 0〜1 or null
  activeTab: 'center'
}
```

---

## 5. 主要関数一覧

### 計算系

| 関数名 | 説明 |
|--------|------|
| `calcWindowHeatmap(w, floorNum)` | 1窓の12×6ヒートマップ（月×時間帯）を計算。遮蔽仰角・樹木補正を含む |
| `calcConstructionHeatmap(floorNum)` | 土地・屋外モード用。4方角（南東西北）× 1階分のヒートマップを返す |
| `buildObstructionMap(floorNum, bldH, bldDist)` | 南向き仰角（±45°）を Float32Array で返す（旧ロジック・一部残存） |
| `applyWindowObstruction(obstMap, ...)` | 窓方向中心 ±60° に仰角をセット |
| `applySVFtoMap(obstMap, svfVal)` | SVF 値から全方位に仰角を設定 |
| `computeOrHeatmap(heatmaps)` | OR 結合: `1 - ∏(1 - p_i)` で物件スコアを算出 |
| `computeAggHeatmap(heatmaps)` | 単純平均ヒートマップ（現在 OR に置き換え済み、参照のみ） |
| `getSeasonScoresFromMatrix(matrix)` | 12×6 matrix から春夏秋冬スコアを返す |
| `calcSeasonScores(matrix)` | `getSeasonScoresFromMatrix` + カウントアップアニメーション起動 |
| `calcPeakMonth(matrix)` | 日照最大の上位2ヶ月を返す |
| `analyzeSVF(img)` | 写真（Image 要素）のピクセル解析で SVF 値（0〜1）を返す |
| `cAnalyzeSky(dataUrl, dir)` | 土地・屋外モード用の空ピクセル割合解析（`cState[dir].svf` に保存） |

### UI系

| 関数名 | 説明 |
|--------|------|
| `renderWindowCard(w, i)` | 1窓カードの HTML 文字列を返す |
| `renderAllWindows()` | 全窓カードを `#windowsContainer` に描画 |
| `rerenderWindow(i)` | 指定インデックスの窓カードのみ再描画 |
| `renderPhotoCards(i, w)` | 3方向写真カード + 遮蔽物入力 HTML を返す |
| `renderSurroundingCheckboxes(i, selected)` | チェックボックスリスト HTML を返す |
| `autoNameWindow(i)` | 方向・サイズから「南の窓・大サイズ」形式で窓名を自動更新 |
| `setWinDir(i, dir)` | 方向セット → autoNameWindow → rerenderWindow → updatePrecision |
| `setWinSize(i, size)` | サイズセット → autoNameWindow → rerenderWindow |
| `addWindow()` | 窓を最大4枚まで追加 |
| `deleteWindow(i)` | 指定窓を削除し全再描画 |
| `selYoto(el, val)` | 用途ボタン選択（`selectedYoto` 更新・屋根庇UIの表示切替） |
| `selRoofDepth(el, val)` | 屋根庇奥行き選択（`roofDepth` 更新） |
| `updatePrecision()` | 入力完了判定 → オレンジ枠フィードバック + `checkCalcReady()` |
| `checkCalcReady()` | 計算ボタンの活性/非活性を判定 |
| `handlePhoto(i, tabKey, input)` | 写真アップロード → `analyzeSVF()` → `updatePrecision()` |
| `deletePhoto(i, tabKey)` | 写真削除 |
| `switchPhotoTab(i, tabKey)` | 写真タブ切替 |
| `displayResults(score, winScores, heatmaps)` | 物件・室内モードの結果表示（#results に innerHTML） |
| `countUp(el, target, duration, suffix)` | カウントアップアニメーション |
| `buildHeatmapHtml(matrix)` | インタラクティブなヒートマップ HTML（タブ切替あり）を生成 |
| `buildHeatmapForCard(matrix)` | スクショカード用の簡易ヒートマップテーブル HTML を生成 |
| `renderHeatmapTable(matrix)` | ヒートマップの table 要素 HTML を生成 |
| `switchHeatmapTab(tabId)` | 結果タブ（総合/窓別）の表示切替 |
| `openScreenshot()` / `closeScreenshot()` | スクショオーバーレイの開閉 |
| `buildSummaryCard(score, seasons, matrix, addr, comment, floor)` | 正方形サマリーカード HTML を生成（floor あり → 「N階」プレフィックス） |
| `buildWindowCard(w, i, score, seasons, matrix, comment)` | 窓別カード HTML を生成 |
| `resetAll()` | 全入力リセット |
| `copyResultUrl()` | 入力状態を URL hash に保存してクリップボードにコピー |
| `restoreFromHash()` | URL hash から入力状態を復元（初期化時に呼ばれる） |
| `showCopyToast()` | 「URLをコピーしました」トースト表示 |
| `showError(msg)` / `hideError()` | エラーボックスの表示/非表示 |
| `getGeolocation()` | Geolocation API で現在地を取得 → Nominatim で住所を逆引き |
| `openHowModal()` / `closeHowModal()` | 仕組みモーダル（iframe）の開閉 |
| `openMapGuideModal()` / `closeMapGuideModal()` | Google Map 取得方法モーダルの開閉（中身は未実装） |
| `openPhotoLightbox(src)` / `closePhotoLightbox()` | 写真拡大表示 |

### コメント・テキスト生成系

| 関数名 | 説明 |
|--------|------|
| `getScoreText(score)` | スコアと selectedUsecase に応じた総評テキストを返す |
| `getSolarAltitudeComment()` | 用途別の太陽軌道コメントを返す |
| `getDirComment(dir)` | 方向別の日照特性コメントを返す |
| `getYotoComment(level)` | 用途 × スコアレベル別コメントを返す |
| `getScreenshotComment(score, windows)` | スクショカード用コメントを生成 |
| `getWindowComment(dir)` | 窓別カード用コメントを返す |

### 土地・屋外モード専用

| 関数名 | 説明 |
|--------|------|
| `displayConstructionResults(maxFloor)` | 階数タブUI + 方角別スコアを `#results` に描画 |
| `switchConstructionFloor(floor)` | タブクリック時に `selectedConstructionFloor` を更新し再描画 |
| `renderConstructionFloor(floor)` | 選択中の階のスコアバー + サマリーカードを `#c-floor-result` に描画 |
| `getConstructionAdvice(floor, dirScores)` | 階 + 最良方角に応じたアドバイス文を生成 |
| `cHandlePhoto(input, dir)` | 写真アップロード → DOM 更新 → `cAnalyzeSky()` |
| `cToggleCheck(key, val, checked)` | チェックボックス変更 → `cState[key].surroundings` を更新 |
| `cSelect(el, groupId)` | グループ内の選択切替（`.c-sel`） |
| `cToggle(dir)` | 方角ブロックのアコーディオン開閉 |
| `cToggleAddPanel()` | 「方角を追加」パネルの表示切替 |
| `cAddSpecificDir(key, arrow, name, el)` | 追加方角ブロックを動的に生成 |
| `cRemoveDir(key)` | 追加した方角ブロックを削除 |

### 設定・初期化系

| 関数名 | 説明 |
|--------|------|
| `makeDefaultWindow(i)` | 窓オブジェクトのデフォルト値を返す |
| `getCurrentConfig()` | `usecaseConfig[selectedUsecase]` を返す |
| `updateStep2()` | ユースケース切替時に STEP2 の UI を更新 |
| `updateStep3()` | ユースケース切替時に STEP3 の UI を更新 |
| `selectCard(el, group)` | カード選択の active クラス切替 |
| `startWithUsecase()` | STEP1 にスクロール |
| `toggleBldDist(selectEl, distId)` | 建物高さ選択時に距離選択を表示/非表示 |
| `runCalculation()` | 計算ボタン押下時のエントリポイント |

---

## 6. 計算ロジック概要

### サンプリング方法

```
月（1〜12） × 時間帯（6区間） × 分（0,10,20...110 = 12点）
= 月の15日を代表日とし、各2時間スロットを10分刻みでサンプリング

TIME_SLOTS = [6, 8, 10, 12, 14, 16]  // 各スロットの開始時刻
```

### 方位角フィルタ

- 太陽方位角（azDeg）と窓方向（windowAz）の差が 90° 以内のときのみ日照とみなす
- ルーフバルコニー（`rooftop`）は全方位受光するためフィルタをスキップ

### 遮蔽仰角の合成

```
formulaObst = atan2(建物高さ - 自分の高さ, 距離)  // 数式入力
photoObst   = (1 - meanSVF) × 30°               // 写真SVF
両方ある場合 → 平均値
片方のみ    → その値を使用

edgeFactor = cos(方位角差)  // 窓正面ほど遮蔽が強く、窓縁は遮蔽なし
obstAngle  = obstBase × edgeFactor
```

### 用途別補正

| 用途 | obstAngle への係数 |
|------|--------------------|
| `living` / `bedroom` 等 | ×1.0 |
| `balcony` / `veranda` | ×0.5 |
| `rooftop` | 0（遮蔽なし） |

### 屋根・庇の補正

```
roofObstAngle = atan2(roofDepth, 2.3)  // 天井高 2.3m
obstBase = max(obstBase, roofObstAngle)
※ ルーフバルコニーは補正なし（roofDepth = 0 固定）
```

### チェックボックス補正

| チェック項目 | 効果 |
|------------|------|
| 常緑樹 | 全季節 -15% |
| 落葉樹 | 冬 -5% / 秋 -10% / 春夏 -15% |
| 看板・広告板 | `obstBase = max(obstBase, 8°)` |
| 隣接壁（真横） | `obstBase = max(obstBase, 20°)` |

### PROPERTY SCORE（OR 結合）

```
物件スコア = 各時間スロットで「いずれかの窓が日照あり」の確率
= 1 - ∏(1 - p_i)   // 複数窓の相乗効果を反映
```

### SVF 画像解析（`analyzeSVF` / `cAnalyzeSky`）

ピクセルを走査して「空」と判定する条件：
- **晴天**: `b > 100 && b > r+10 && b > g×0.85`
- **曇天**: 上半分 60% かつグレー系（R,G,B 均等 150〜245）
- **除外**: 下部 40% の白壁ピクセル（R,G,B > 220 かつ均等）

---

## 7. UI 定数

```js
DIR_AZ = { '南':180, '南東':135, '南西':225, '東':90, '西':270, '北':0, '北東':45, '北西':315 }
SIZE_MULT = { '小':0.85, '中':1.0, '大':1.1 }
TIME_SLOTS  = [6, 8, 10, 12, 14, 16]
TIME_LABELS = ['6-8時','8-10時','10-12時','12-14時','14-16時','16-18時']
MONTH_LABELS = ['1月',...,'12月']

// 窓の方角に応じた3方向写真タブのラベルマップ
PHOTO_TAB_MAP = {
  '南': { left:'南東', center:'南', right:'南西' },
  // ... 8方位すべて定義
}

// usecaseConfig（5ユースケース定義）
// 現在 UI で選択可能: '物件・室内' / '土地・屋外'
// 定義済みだが未実装: '農業・園芸' / 'ソーラー・エネルギー' / '商業・施設'
```

---

## 8. カラーパレット（CSS 変数）

| 変数 | 値 | 用途 |
|------|----|------|
| `--bg` | `#F5F4F0` | ページ背景 |
| `--white` | `#FFFFFF` | カード背景 |
| `--black` | `#1A1A1A` | 主テキスト・ヘッダー |
| `--amber` | `#F5A623` | ブランドカラー（アクティブ・スコア） |
| `--amber-dk` | `#D4880F` | ラベル・リンク |
| `--gray` | `#6B7280` | サブテキスト |
| `--gray-lt` | `#EFEFEC` | 入力欄背景 |
| `--border` | `rgba(0,0,0,0.08)` | 通常の枠線 |
| `--border-md` | `rgba(0,0,0,0.12)` | やや強い枠線 |
| `--red` | `#EF4444` | エラー・必須ラベル |
| `--green` | `#10B981` | 完了バッジ |

入力完了フィードバック：`borderColor: '#F5A623'`, `background: 'rgba(245,166,35,0.06)'`

---

## 9. 2つのモード

### 物件・室内モード（selectedUsecase = '物件・室内'）

1. 窓ごとに `calcWindowHeatmap(w, floorNum)` → 12×6 の確率行列
2. `computeOrHeatmap(heatmaps)` → PROPERTY SCORE
3. `displayResults()` → サマリーカード + 窓別カード（2枚以上の場合）
4. `storedHeatmaps` に保存 → スクショ用に再利用

### 土地・屋外モード（selectedUsecase = '土地・屋外'）

1. `runCalculation()` → 1〜N 階ループ
2. 各階で `calcConstructionHeatmap(f)` → `{south, east, west, north}` の4方角 matrix
3. `constructionResults = [{floor: N, data: {...}}]` に保存
4. `displayConstructionResults(maxFloor)` → 階数タブ UI を生成
5. `renderConstructionFloor(floor)` → 選択階の方角別スコアバー + サマリーカード

---

## 10. 実装済み機能（2026-04 時点）

- 緯度経度入力（Decimal・DMS・カッコ形式に対応）
- 現在地取得（Geolocation API・`enableHighAccuracy:true`）
- Nominatim 逆ジオコーディング（町名レベル）
- 用途選択見出し「どこの日当たりを調べる？」
- 物件カード「お部屋の日当たりを調べる」/ 土地カード「土地や屋外の日当たりを調べる」
- 用途ボタンのアイコン削除（室内 / ベランダ・バルコニー / ルーフバルコニー）
- STEP2 タイトル「物件・お部屋の情報」
- 窓の向きコンパス選択（8方位グリッド配置）
- 窓名の自動生成（`autoNameWindow`）: 「南の窓・大サイズ」形式
- 用途別撮影ガイド（hand.png + 用途別イラスト）
- 写真アップロード（3方向）+ SVF 自動解析
- 月別日照率ヒートマップ（窓別タブ表示）
- 季節別スコア（カウントアップアニメーション）
- PROPERTY SCORE（OR 計算）
- 正面の遮蔽物入力を「正面に見える遮蔽物について」に統一
- 樹木補正・看板補正・隣接壁補正
- 正方形判定カードデザイン（サマリー + 窓別）
- 土地・屋外モード：階数タブ切替 + 方角別スコアバー + 設計アドバイス
- スクショカードに階数表示（floor 引数）
- URL コピー（物件・室内・土地・屋外 両モード対応）
- URL hash からの入力復元
- iOS 自動ズーム防止（`font-size: 16px` 統一）
- BtoB LP（sunny_lp.html）
- ナビゲーション「Sunnyとは？」リンク
- Google Map から緯度経度を取得する方法モーダルを実装（スマホ・PC 別の手順＋画像表示）
- sunny_hiatari.html：「日当たりって大切？」コラムページを追加（エビデンス・データ・ユーザーの声・CTA含む）
- ナビゲーションに「日当たりって大切？」リンクを追加（index.html・sunny_lp.html）
- TOP ファーストビューの CTA ボタンを「日当たりって大切？」へ変更
- sunny_lp.html に「詳しい算出方法をみる」リンクを追加
- sunny_hiatari.html：各セクションに出典リンクを追加（01〜04全セクション）
- sunny_hiatari.html：参考データ・記事セクションを削除（各セクションに出典を統合）
- 全ページのトップバー左側ロゴ・「日当たり算出ツール」テキストを統一
- 全ページのフッターに「日当たりって大切？」「算出方法」リンクを追加・統一
- sunny_lp.html：PROBLEMセクションの見出し・本文を修正
- sunny_lp.html：引用ブロック・DATAセクションを削除
- sunny_lp.html：信頼性セクションに「詳しい算出方法をみる」リンクを追加
- sunny_hiatari.html：エディトリアルデザインにフルリニューアル（白背景・グリッドレイアウト・DM Mono書体）
- sunny_lp.html：エディトリアルデザインにフルリニューアル（hiatari.html と同一デザインシステム）
- 両ページ共通：本文テキスト黒文字化（#888→#333）・グリッドアイテムにカード背景（#f8f8f6・角丸）追加
- 両ページ共通：h1/h2/h3 のサイズ・色・ウェイトを統一（sec-h2: 39px/500、sec-h3: 25px/400、hero-h1: 62px/300）
- 各ページヘッダーのテキストナビリンクを削除（「ツールを使う」CTAボタンのみ残す）
- 全ページ「Sunny」→「SUNNY」大文字統一
- 日当たりシミュレーション機能追加（`buildHeatmapHtml(matrix, winIdx)` + `showLightSim` + `switchSimWin`）
  - ヒートマップの各セルをタップ → 断面図で日差しの入り方をシミュレーション
  - SunCalc で太陽高度・方位を計算、庇（`roofDepth`）の影響を反映
  - アンバーグラデーションの光ビーム + 「差し込みXm」矢印表示
  - チップ色はヒートマップの `getHeatInfo()` と完全統一
  - 複数窓対応：窓選択タブ切替（`switchSimWin`）
- 全ページのフッター著作権を「日照率チェッカー」→「日当たり算出ツール」に統一
- 全ページのフッターナビ「SUNNYとは？」→「日当たり算出ツールSUNNYとは？」、「算出方法」→「算出計算方法」に統一

---

## 11. 今後の TODO

### フェーズ1（今すぐ・進行中）
- FAQ ページ作成
- サブページ（sunny_privacy・sunny_terms・sunny_contact）のフッター追加

### フェーズ2（オープン化のタイミング）
- ドメイン取得（sunnyin.com / sunnyin.jp 候補）
- URL 構成整理（LP → トップ、ツール → /app）
- PWA 化（manifest.json + Service Worker）
- Supabase 導入・データ蓄積（算出日時・緯度経度・用途・スコア）
- LP「導入実績」セクション追加（営業活動後）
- SUUMO 等への BtoB 提案

---

## 12. ビジネス戦略

**現在の方針**：クローズドβ。営業した不動産・建築会社に使ってもらい実績を積む → LP「導入実績」掲載 → SUUMO 等へ API 連携・BtoB 営業。

**ターゲット**：
- 主：不動産仲介・売買、建築・設計会社、賃貸管理会社
- 副：一般ユーザー（購入検討者・テレワーカー・ベランダ菜園等）

**権威性確保**（検討中）：X・note で発信している都内建築士への監修依頼を直接コンタクトで進める方針。
