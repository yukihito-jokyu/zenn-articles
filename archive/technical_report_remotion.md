---
tags:
  - Remotion
  - React
  - VideoCreation
  - TailwindCSS
created: "2026-01-27-12:00"
---

# Remotion 技術検証レポート

## 参考文献
- [GitHub Issue #40](https://github.com/yukihito-jokyu/task-manager/issues/40)
- [Remotion 公式ドキュメント](https://www.remotion.dev/)
- [Tailwind CSS v4 with Remotion](https://www.remotion.dev/docs/tailwind)

---

## 技術概要

**Remotion** は、**Reactを使ってプログラマティックに動画を作成できるフレームワーク**です。Web技術（HTML, CSS, JavaScript）を駆使して、mp4やWebMなどの動画ファイルを生成したり、動的なプレビューを行ったりすることができます。

### 主な特徴
- **Reactコンポーネントベース**: 再利用可能なコンポーネントで動画を構築
- **Web技術の活用**: CSSアニメーション、SVG、Canvasなどがそのまま使える
- **プログラマティック**: 変数やAPIデータに基づいた動的な動画生成が可能
- **高速レンダリング**: 並列処理による高速な動画書き出し

---

## 実施内容

### ゴール
- **Pocket TTSの紹介動画（約2分30秒）をRemotionで作成する**
- Tailwind CSS v4 を使用してスタイリングを行う
- 映像のみのMVPを作成

### 成果物
- **プロジェクト**: `work/remotion-pocket-tts`
- **動画構成**:
  1. **Intro**: タイトルアニメーション
  2. **Overview**: 3つの主要機能（CPU最適化、軽量、リアルタイム）
  3. **Features**: 詳細機能リスト
  4. **Demo**: ターミナル風の実行デモ
  5. **Comparison**: 他社サービスとの比較表
  6. **Outro**: まとめとURL

### 達成した項目
1. ✅ Remotionプロジェクトのセットアップ（Blankテンプレート）
2. ✅ Tailwind CSS v4 の連携設定
3. ✅ `src/components` 配下に6つのセクションコンポーネント作成
4. ✅ `Sequence` を使ったタイムライン構築（全4500フレーム / 150秒）
5. ✅ 日本語テキストによる解説動画化

---

## 検証詳細

### 1. プロジェクト作成と課題
`npx create-video@latest` コマンドを使用しましたが、対話モード（テンプレート選択やGit設定）が必須であり、完全自動化（`--yes`オプションなど）が難しかったです。
**解決策**: ユーザーによる手動操作でプロジェクトを作成しました。

### 2. Tailwind CSS v4 の導入
Remotion v4系では、Tailwind CSS v4 (`@remotion/tailwind-v4`) が利用可能です。
- **設定**: `remotion.config.ts` で `Config.overrideWebpackConfig(enableTailwind)` を記述。
- **CSS**: `src/index.css` に `@import "tailwindcss";` を記述するだけで動作しました。

### 3. アニメーション実装
Remotionのフック `useCurrentFrame`, `interpolate`, `spring` を使用してアニメーションを実装しました。
- **Spring**: 弾むようなリッチな動き（タイトルの拡大など）
- **Interpolate**: フレーム数に応じた値の保管（透明度や移動）
- **Sequence**: コンポーネントの表示タイミング制御

```tsx
// 例: フェードイン + スライドイン
const opacity = interpolate(frame, [0, 20], [0, 1]);
const translateY = interpolate(frame, [0, 20], [50, 0]);
```

### 4. 音声との同期（可能性の検証）
今回は映像のみの実装でしたが、Remotionは**プログラムによる完全制御**が可能なため、以下の実装が可能です。
- **音声ファイルの長さに合わせた動画尺の自動調整**: `getAudioDurationInSeconds()` などを用いて、ナレーションの長さに応じてシーンの長さを動的に決定する。
- **字幕の自動同期**: JSONなどのスクリプトデータに基づき、音声とテキストを完全に同期させて表示する。
- **波形アニメーション**: `useAudioData()` フックを使用して、実際の音声波形をリアルタイムに視覚化する（オーディオビジュアライザー）。

---

## 依存ライブラリ

| パッケージ | バージョン | 用途 |
|-----------|------------|------|
| `remotion` | 4.0.410 | コアライブラリ |
| `react` | 19.2.3 | UIライブラリ |
| `tailwindcss` | 4.0.0 | スタイリング |
| `@remotion/tailwind-v4` | 4.0.410 | Tailwind連携 |
| `@remotion/google-fonts` | 最新 | フォント読み込み |

---

## トラブルシューティング

### 問題: 対話モードの自動化不能
**状況**: `npx create-video` がCI/CDやエージェント環境での自動実行に対応しきれていない（テンプレート引数が効かない場合がある等）。
**対処**: 初回セットアップのみ手動で行うか、`npm init video` の引数を調査して完全に指定する必要がある。今回は手動実行で回避。

### 問題: 日本語フォント
**状況**: デフォルトでは日本語フォントが指定されていないため、OS依存になる。
**対処**: Google Fonts（Noto Sans JPなど）を `@remotion/google-fonts` で読み込むのがベストプラクティス。今回はデフォルトフォントを使用したが、表示は問題なし。

---

## 今後の展望

1. **完全自動生成パイプライン**: テキストを入力すると、TTSで音声を生成し、その長さに合わせてRemotionで動画を自動生成するツールを作成する。
2. **動的データ連携**: GitHub APIから最新のスター数を取得して動画内に表示するなど、Remotionの強みを活かす。
3. **レンダリング自動化**: GitHub Actionsで `npx remotion render` を実行し、コードが更新されるたびに動画を自動生成するワークフローの構築。
