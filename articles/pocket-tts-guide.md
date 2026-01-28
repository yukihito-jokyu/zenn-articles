---
title: "Pocket TTSでボイスクローニングを試す"
emoji: "🔊"
type: "tech"
topics: ["tts", "音声合成", "ボイスクローニング", "python"]
published: false
---

Pocket TTSは、Kyutai Labsが開発した軽量なText-to-Speech（テキスト読み上げ）アプリケーションです。CPUで動作し、リアルタイム処理が可能で、ボイスクローニング機能も備えています。

## Pocket TTSの特徴

| 特徴 | 詳細 |
|------|------|
| **CPUで動作** | GPU不要、MacBook Air M4で約6倍速のリアルタイム処理 |
| **小さなモデル** | 100Mパラメータ、軽量設計 |
| **低レイテンシ** | 最初の音声チャンクまで約200ms |
| **オーディオストリーミング** | リアルタイムで音声を生成 |
| **ボイスクローニング** | WAVファイルから声を複製可能 |
| **CLI & Python API** | コマンドラインとPythonライブラリの両方で利用可能 |

## 注意点

- **英語のみ対応**（日本語は未サポート）
- GPU使用時のスピードアップは限定的（バッチサイズ1のため）

## インストール

`uvx` を使用してインストールします（依存関係は自動でインストールされます）。

```bash
uvx pocket-tts generate
```

## 使い方

### 基本的な使い方

デフォルト設定で音声を生成します。

```bash
uvx pocket-tts generate
```

**結果**: `./tts_output.wav` が出力されます。

### 別のボイスで生成

```bash
uvx pocket-tts generate \
  --voice marius \
  --text "Hello, this is a voice cloning test with Pocket TTS." \
  --output-path ./voice_marius.wav
```

### ボイスクローニング

Hugging Faceから音声ファイルを指定し、ボイスクローニングを行います。

```bash
uvx pocket-tts generate \
  --voice "hf://kyutai/tts-voices/expresso/ex01-ex02_default_001_channel2_198s.wav" \
  --text "This is a voice cloning demonstration. Pocket TTS can clone any voice from a short audio sample." \
  --output-path ./voice_cloned_expresso.wav
```

**結果**: 指定した音声の特徴を模倣した読み上げが生成されます。

## トラブルシューティング

### ボイスクローニングができない

**エラー**:
```
ValueError: We could not download the weights for the model with voice cloning, but you're trying to use voice cloning.
```

**解決策**:
1. https://huggingface.co/kyutai/pocket-tts で利用規約に同意
2. Hugging Faceにログイン
   ```bash
   uvx --from huggingface-hub hf auth login
   ```

### 日本語が正しく出力されない

Pocket TTSは現時点で英語のみ対応しています。日本語を入力すると以下のような警告が出ます。

```
Maximum generation length reached without EOS
```

意味のない音声が生成されるため、英語での使用を推奨します。

## 代替案

日本語での音声合成が必要な場合は、以下のサービスを検討してください。

| サービス/ツール | 特徴 | 日本語対応 |
|----------------|------|-----------|
| **Pocket TTS** | CPU動作、軽量、ボイスクローニング可 | ❌ |
| **Coqui TTS** | オープンソース、多言語対応 | ⚠️ 限定的 |
| **OpenAI TTS** | 高品質、API経由 | ✅ |
| **VOICEVOX** | 日本語特化、無料 | ✅ |

## 今後の活用

- **Python APIの活用**: アプリケーションへの組み込み
- **サーバーモード**: `pocket-tts serve` でWebインターフェースを立ち上げ
- **自分の声でクローニング**: 実際に自分の音声を録音して試す
- **日本語対応の動向をウォッチ**: 公式の言語拡張を待つ

## 参考リンク

- [Pocket TTS 公式リポジトリ](https://github.com/kyutai-labs/pocket-tts)
- [Hugging Face モデルカード](https://huggingface.co/kyutai/pocket-tts)
- [Kyutai 公式デモ](https://kyutai.org/pocket-tts)
