---
title: "RemotionとReactでプログラマティックに動画を作成する"
emoji: "🎬"
type: "tech"
topics: ["remotion", "react", "video", "tailwindcss"]
published: false
---

Remotionは、Reactを使ってプログラマティックに動画を作成できるフレームワークです。Web技術（HTML、CSS、JavaScript）を駆使して、mp4やWebMなどの動画ファイルを生成したり、動的なプレビューを行ったりすることができます。

## Remotionの特徴

- **Reactコンポーネントベース**: 再利用可能なコンポーネントで動画を構築
- **Web技術の活用**: CSSアニメーション、SVG、Canvasなどがそのまま使える
- **プログラマティック**: 変数やAPIデータに基づいた動的な動画生成が可能
- **高速レンダリング**: 並列処理による高速な動画書き出し

## 環境構築

### プロジェクトの作成

```bash
npx create-video@latest
```

対話モードでテンプレートを選択し、プロジェクトを作成します。今回は「Blank」テンプレートを使用しました。

### Tailwind CSS v4 の導入

Remotion v4系では、Tailwind CSS v4が利用可能です。

1. `@remotion/tailwind-v4` をインストール
   ```bash
   npm install @remotion/tailwind-v4
   ```

2. `remotion.config.ts` で設定
   ```ts
   import {Config} from '@remotion/cli/config';
   import {enableTailwind} from '@remotion/tailwind-v4';

   Config.overrideWebpackConfig(enableTailwind);
   ```

3. `src/index.css` にインポート
   ```css
   @import "tailwindcss";
   ```

## 動画の構成

今回は「Pocket TTSの紹介動画（約2分30秒）」を作成しました。以下のセクションで構成されています。

1. **Intro**: タイトルアニメーション
2. **Overview**: 3つの主要機能（CPU最適化、軽量、リアルタイム）
3. **Features**: 詳細機能リスト
4. **Demo**: ターミナル風の実行デモ
5. **Comparison**: 他社サービスとの比較表
6. **Outro**: まとめとURL

## アニメーションの実装

Remotionのフックを使用してアニメーションを実装します。

### Springアニメーション

弾むようなリッチな動きを実装します。

```tsx
import {spring} from 'remotion';

const scale = spring({
  frame,
  fps: 30,
  config: {
    damping: 200,
    stiffness: 100,
    mass: 0.5,
  },
});
```

### Interpolateアニメーション

フレーム数に応じた値の補間を行います。

```tsx
import {interpolate} from 'remotion';

const opacity = interpolate(frame, [0, 20], [0, 1]);
const translateY = interpolate(frame, [0, 20], [50, 0]);
```

### Sequenceによるタイムライン制御

コンポーネントの表示タイミングを制御します。

```tsx
import {Sequence} from 'remotion';

<Sequence from={0} durationInFrames={100}>
  <IntroSection />
</Sequence>
<Sequence from={100} durationInFrames={200}>
  <OverviewSection />
</Sequence>
```

## 音声との同期

Remotionはプログラムによる完全制御が可能なため、以下の実装ができます。

### 音声ファイルの長さに合わせた動画尺の自動調整

```ts
import {getAudioDurationInSeconds} from '@remotion/media-utils';

const audioDuration = await getAudioDurationInSeconds('audio.mp3');
```

ナレーションの長さに応じてシーンの長さを動的に決定できます。

### 字幕の自動同期

JSONなどのスクリプトデータに基づき、音声とテキストを完全に同期させて表示できます。

```tsx
const subtitles = [
  {start: 0, end: 3, text: "Pocket TTSの紹介"},
  {start: 3, end: 6, text: "軽量で高速な音声合成"},
];

// 現在のフレームに対応する字幕を表示
```

### 波形アニメーション

`useAudioData()` フックを使用して、実際の音声波形をリアルタイムに視覚化できます。

```tsx
import {useAudioData} from '@remotion/media-utils';

const {audioData} = useAudioData('audio.mp3');
```

## トラブルシューティング

### 対話モードの自動化

`npx create-video` は対話モードが必須であり、完全自動化が難しい場合があります。CI/CDでの自動実行を希望する場合は、引数を調査するか、手動セットアップを行ってください。

### 日本語フォント

デフォルトでは日本語フォントが指定されていないため、OS依存になります。Google Fonts（Noto Sans JPなど）を `@remotion/google-fonts` で読み込むのがベストプラクティスです。

```bash
npm install @remotion/google-fonts
```

## 今後の活用

- **完全自動生成パイプライン**: テキストを入力すると、TTSで音声を生成し、その長さに合わせてRemotionで動画を自動生成するツールを作成
- **動的データ連携**: GitHub APIから最新のスター数を取得して動画内に表示するなど、Remotionの強みを活かす
- **レンダリング自動化**: GitHub Actionsで `npx remotion render` を実行し、コードが更新されるたびに動画を自動生成するワークフローの構築

## 参考リンク

- [Remotion 公式ドキュメント](https://www.remotion.dev/)
- [Tailwind CSS v4 with Remotion](https://www.remotion.dev/docs/tailwind)
