---
tags:
  - TTS
  - 音声合成
  - ボイスクローニング
created: "2026-01-26-23:25"
---

# Pocket TTS 技術検証レポート

## 参考文献
- [GitHub Issue #39](https://github.com/yukihito-jokyu/task-manager/issues/39)
- [Pocket TTS 公式リポジトリ](https://github.com/kyutai-labs/pocket-tts)
- [Hugging Face モデルカード](https://huggingface.co/kyutai/pocket-tts)
- [Kyutai 公式デモ](https://kyutai.org/pocket-tts)

---

## 技術概要

**Pocket TTS** は、Kyutai Labs が開発した**軽量なText-to-Speech（テキスト読み上げ）アプリケーション**。

### 主な特徴
| 特徴 | 詳細 |
|------|------|
| **CPUで動作** | GPU不要、MacBook Air M4で約6倍速のリアルタイム処理 |
| **小さなモデル** | 100Mパラメータ、軽量設計 |
| **低レイテンシ** | 最初の音声チャンクまで約200ms |
| **オーディオストリーミング** | リアルタイムで音声を生成 |
| **ボイスクローニング** | WAVファイルから声を複製可能 |
| **CLI & Python API** | コマンドラインとPythonライブラリの両方で利用可能 |

### 制限事項
- **英語のみ対応**（日本語は未サポート）
- GPU使用時のスピードアップは限定的（バッチサイズ1のため）

---

## 実施内容

### ゴール
- Pocket TTSを使って、任意の音声ファイルからボイスクローニングを行い、テキストをその声で読み上げる

### 達成した項目
1. ✅ Pocket TTSのインストールと動作確認（MVP）
2. ✅ 複数のプリセットボイスでの音声生成
3. ✅ Hugging Faceへのログインとボイスクローニング用モデルの取得
4. ✅ 外部音声ファイルを使ったボイスクローニング
5. ✅ 日本語対応の検証（結果：未対応）

---

## 検証詳細

### 1. 動作確認（MVP）
```bash
uvx pocket-tts generate
```
**結果**: 成功
- 生成時間: 940 ms で 6320 ms の音声を生成
- 速度: リアルタイムの 6.72倍
- 出力: `./tts_output.wav`

### 2. 別のボイスでの生成
```bash
uvx pocket-tts generate --voice marius --text "Hello, this is a voice cloning test with Pocket TTS." --output-path ./voice_marius.wav
```
**結果**: 成功
- 速度: リアルタイムの 6.77倍

### 3. ボイスクローニング
```bash
uvx pocket-tts generate \
  --voice "hf://kyutai/tts-voices/expresso/ex01-ex02_default_001_channel2_198s.wav" \
  --text "This is a voice cloning demonstration. Pocket TTS can clone any voice from a short audio sample." \
  --output-path ./voice_cloned_expresso.wav
```
**結果**: 成功
- エンコーディング時間: 1031 ms
- 生成時間: 2214 ms で 7120 ms の音声を生成
- 速度: リアルタイムの 3.22倍
- ※プリセットボイスより遅いのは、音声エンコーディングが必要なため

### 4. 日本語テスト
```bash
uvx pocket-tts generate \
  --voice "hf://kyutai/tts-voices/expresso/ex01-ex02_default_001_channel2_198s.wav" \
  --text "こんにちは、これは日本語のテストです。" \
  --output-path ./voice_japanese_test.wav
```
**結果**: 失敗（予想通り）
- 警告: `Maximum generation length reached without EOS`
- 日本語は正しく処理されず、意味のない音声が生成された

---

## 依存ライブラリ

`uvx` を使用したため、依存関係は自動でインストールされた。主なパッケージ：

| パッケージ | 用途 |
|-----------|------|
| `pocket-tts` | メインパッケージ |
| `torch` | PyTorch（CPU版で動作） |
| `scipy` | WAVファイル処理 |
| `huggingface_hub` | モデルダウンロード |

---

## トラブルシューティング

### 問題1: ボイスクローニングができない
**エラー内容**:
```
ValueError: We could not download the weights for the model with voice cloning, but you're trying to use voice cloning.
```

**解決策**:
1. https://huggingface.co/kyutai/pocket-tts で利用規約に同意
2. `uvx --from huggingface-hub hf auth login` でHugging Faceにログイン

### 問題2: コマンドオプションの間違い
**エラー内容**: `--output` ではなく `--output-path` が正しいオプション

**解決策**: `uvx pocket-tts generate --help` でオプションを確認

---

## 代替案・比較

| サービス/ツール | 特徴 | 日本語対応 |
|----------------|------|-----------|
| **Pocket TTS** | CPU動作、軽量、ボイスクローニング可 | ❌ |
| **Coqui TTS** | オープンソース、多言語対応 | ⚠️ 限定的 |
| **OpenAI TTS** | 高品質、API経由 | ✅ |
| **VOICEVOX** | 日本語特化、無料 | ✅ |

---

## 今後の展望

1. **Python APIの活用**: アプリケーションへの組み込み
2. **サーバーモード**: `pocket-tts serve` でWebインターフェースを立ち上げ
3. **自分の声でクローニング**: 実際に自分の音声を録音して試す
4. **日本語対応の動向をウォッチ**: 公式の言語拡張を待つ

---

## 生成ファイル一覧

| ファイル名 | 説明 |
|-----------|------|
| `tts_output.wav` | デフォルト設定での生成（alba ボイス） |
| `voice_marius.wav` | marius ボイスでの生成 |
| `voice_cloned_expresso.wav` | ボイスクローニング（expressoサンプル） |
| `voice_japanese_test.wav` | 日本語テスト（失敗） |
