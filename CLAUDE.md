# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Sunny プロジェクト概要

日照率算出Webアプリ。単一HTMLファイル構成。GitHub Pagesでデプロイ済み。

- **本番URL**: https://kokingbeach-netizen.github.io/sunny-app/
- **ローカル確認**: http://192.168.2.107:8080/phase1.html（`phase1.html` が開発版、`index.html` が本番と同期）

## デプロイ

ビルドステップなし。`git push origin main` でGitHub Pagesに自動デプロイ（数分かかる場合あり）。

## ファイル構成

| ファイル | 役割 |
|---|---|
| `index.html` | 本番ファイル（GitHub Pages用） |
| `phase1.html` | 開発ファイル（ローカル確認用）。`index.html` と常に同内容に保つ |
| `sunny_how_it_works.html` | 算出方法の説明ページ（モーダルで表示） |

`phase1.html` と `index.html` は実質同一。修正時は**必ず両方に適用**する。

## 技術構成

- **Vanilla JS**（フレームワークなし）
- **SunCalc.js**（npm package、node_modules内）：緯度経度から太陽位置を計算
- **Chart.js**（CDN）：ヒートマップ表示
- ライブラリはすべてCDN or インライン読み込み

## コアロジック

### 日照計算（`calcWindowHeatmap(w, floorNum)`）

月×時間スロット（12×6）のmatrixを返す。各セルは日照率（0〜1）。

1. 遮蔽仰角 `obstBase` を算出（建物高さ・距離・SVF写真・周辺環境から合成）
2. 各月の15日を代表日に、10分刻みでSunCalc.getPosition()を呼ぶ
3. 判定条件：`altDeg > 0`（地平線上）、`azDiff <= 90`（窓の視野内）、`altDeg > effectiveObstAngle`

### 用途別の遮蔽補正（`effectiveObstAngle`）

| 用途 | 補正 |
|---|---|
| living / bedroom / study / kids / kitchen / sunroom | obstAngle × 1.0（変更なし） |
| balcony / veranda | obstAngle × 0.5 |
| rooftop | 0（完全開放） |

### スコア集計

- `windowScores`：各窓のmatrix全要素の平均 × 100
- `propertyScore`：全窓スコアの平均（メインの「81%」表示）
- `lastSeasonScores`：`calcSeasonScores(matrix)` で春夏秋冬の平均を保持（グローバル変数）

### AIコメント生成

`getScoreText(score)` → `base` + `getSolarAltitudeComment()` + `getYotoComment(level)`

- `getSolarAltitudeComment()`：`selectedYoto`（グローバル）に基づいて用途別の太陽高度解説を返す
- `getYotoComment(level)`：high/mid/low/vlowの4段階 × 用途のマトリクスで文言決定

### 緯度経度パーサー（`parseLatLng` / `dmsToDecimal`）

- Decimal形式（カンマ・スペース区切り）と DMS形式（`35°41'35.2"N`）の両方に対応
- ペーストボックス（`#pasteInput`）へのpaste/inputイベントで自動判定・分割

## グローバル変数（重要）

| 変数 | 内容 |
|---|---|
| `selectedYoto` | 用途選択値（living/bedroom/balcony/rooftop etc.） |
| `selectedUsecase` | 居住用 or 建築前リサーチ |
| `windowStates` | 窓の入力状態配列 |
| `lastSeasonScores` | 最新の季節スコア `{spring, summer, autumn, winter}` |
| `storedHeatmaps` | 計算済みmatrix（スクリーンショット生成等で再利用） |

## sunny_how_it_works.html の構成

ページの主軸は黒背景の3つのh2セクション（`hw-prologue` スタイル）：

- **h2①「自然界の法則」**：緯度・地軸の傾き・南中高度の説明（law-cardsスタイル）
- **h2②「あなたの家の情報」**：階数・窓の向き/写真
- **h2③「遮蔽物」**：向かいの建物の高さ・距離
- **EXAMPLE**：計算の流れ（STEP1〜3・判定テーブル）
- **ENGINE / USE CASE / SEASON / ACCURACY / SOURCE**：各種補足セクション

カード内テキスト色は `hw-prologue` クラス内でCSSオーバーライドが必要（`.hw-prologue .input-title` など）。

## 直近のTODO

- 実機テストの積み上げ（精度向上）
- SUUMOへのBtoB提案準備
- Google Maps API統合
