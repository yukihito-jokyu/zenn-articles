---
title: "MacでQwen3-TTSを使ってみよう：音声合成環境のセットアップガイド"
emoji: "🎤"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["qwen3", "tts", "mac", "音声合成", "python"]
published: false
---

## はじめに：Qwen3-TTSとは？

**Qwen3-TTS**は、Alibabaが公開している音声合成（Text-to-Speech）のライブラリです。

文章を入力するだけで、自然な音声を生成してくれます。AIによるボイスオーバー作成や、ナレーション生成など、幅広い用途で活用できます。

この記事では、Mac（Apple Silicon）環境でQwen3-TTSを動かし、日本語を音声に変換するまでの手順を解説します。

### この記事でできること

- Qwen3-TTSをMacにインストール
- 日本語テキストから音声ファイルを生成

## 準備編：必要な環境

### 対象環境

- **Mac**（Apple Silicon搭載、M1/M2/M3など）
- **Python 3.12**
- **ターミナル操作ができる**

### 事前に必要なもの

- Homebrewがインストールされていること
- インターネット接続（モデルダウンロードに使用）

### 用語解説

| 用語 | 意味 |
|------|------|
| **TTS** | Text-to-Speechの略。文章を音声に変換する技術 |
| **uv** | Pythonのパッケージ管理ツール。pipよりも高速 |
| **MPS** | Metal Performance Shadersの略。Apple SiliconのGPUを活用する仕組み |
| **ModelScope** | Alibabaが提供するモデル共有サービス。HuggingFaceの中国版のようなもの |

## ステップ1：環境構築

### 1.1 必要なツールのインストール

まず、音声処理に必要な`sox`をインストールします。

```bash
brew install sox
```

**sox（Sound eXchange）**は、音声ファイルを扱うためのツールです。Qwen3-TTSが内部で利用しています。

### 1.2 作業ディレクトリの作成

作業用のディレクトリを作成し、移動します。

```bash
mkdir -p work/qwen3-tts-demo
cd work/qwen3-tts-demo
```

### 1.3 Python環境のセットアップ

`uv`を使ってPython環境を初期化します。

```bash
uv init --python 3.12
```

### 1.4 依存ライブラリのインストール

必要なライブラリをインストールします。

```bash
uv add qwen-tts modelscope soundfile
```

| ライブラリ | 用途 |
|-----------|------|
| `qwen-tts` | 音声合成の本体ライブラリ |
| `modelscope` | モデルをダウンロードするためのライブラリ |
| `soundfile` | 音声ファイルを扱うためのライブラリ |

## ステップ2：モデルのダウンロード

### モデルとは？

Qwen3-TTSは「学習済みのニューラルネットワークモデル」を使って音声を生成します。

モデルは事前に学習済みで、それをダウンロードして使う形になります。モデルサイズは**約1GB**以上あるため、ダウンロードに時間がかかります。

### ModelScopeを使う理由

モデルはModelScopeからダウンロードします。HuggingFaceでも公開されていますが、環境によってはダウンロードが非常に遅くなる場合があります。

ModelScope（Alibaba提供）を使うことで、より安定したダウンロードが期待できます。

### ダウンロードスクリプトの作成

以下の内容で`download_script.py`を作成します。

```python
from modelscope import snapshot_download

# モデルのダウンロード
model_dir = snapshot_download(
    "Qwen/Qwen3-TTS-12Hz-0.6B-CustomVoice",
    cache_dir="./models"
)

print(f"Model downloaded to: {model_dir}")
```

### ダウンロードの実行

```bash
uv run python -u download_script.py
```

`-u`オプションをつけることで、進捗状況をリアルタイムで確認できます。

**⏱️ 時間の目安**: 回線環境にもよりますが、10〜30分程度かかる場合があります。

## ステップ3：音声を生成してみる

### テストスクリプトの作成

以下の内容で`run_tts_test.py`を作成します。

```python
import torch
from qwen_tts import QwenTTS

# デバイスの設定（Mac GPUを活用）
device = "mps" if torch.backends.mps.is_available() else "cpu"
print(f"Using device: {device}")

# モデルのロード
print("Loading model...")
tts = QwenTTS(
    model_path="./models/Qwen/Qwen3-TTS-12Hz-0.6B-CustomVoice",
    device=device
)

# 音声生成
text = "こんにちは、これはテストです。Qwen3-TTSを使って、日本語を音声に変換します。"
print(f"Generating speech for: {text}")

audio = tts.generate(text)

# ファイルに保存
output_path = "output_test.wav"
tts.save(audio, output_path)
print(f"Audio saved to: {output_path}")
```

### 音声生成の実行

```bash
uv run run_tts_test.py
```

### 出力の確認

成功すると、`output_test.wav`ファイルが生成されます。

```bash
open output_test.wav
```

または、ターミナルで直接再生します。

```bash
play output_test.wav
```

`play`コマンドは`sox`に含まれている機能です。

## トラブルシューティング

### ダウンロードが遅い、途中で止まる

**原因**: モデルサイズが大きいため、回線状況やサーバー混雑で遅くなることがあります。

**対処法**:
- 安定したネットワーク環境で行う
- 止まった場合は`Ctrl+C`で中断し、再実行する
- 時間をずらして再試行する

### Lockファイルの問題

**現象**: 再実行するとエラーが出る。

**原因**: 途中中断の際、ロックファイルが残っている場合があります。

**対処法**: ロックファイルを削除します。

```bash
rm -f ~/.cache/modelscope/hub/.lock
```

### Sox関連のエラー

**現象**: 実行時に`sox: command not found`などのエラー。

**対処法**: Homebrewでsoxをインストールします。

```bash
brew install sox
```

### MPS関連の警告

**現象**: `flash-attn is not supported on this device`のような警告が出る。

**説明**: MacではFlash Attentionがサポートされていないため、通常のAttention実装が使われます。

**影響**: 推論速度はCUDA環境より遅くなりますが、実用範囲内です。問題なく動作します。

## まとめ

この記事では、Mac環境でQwen3-TTSをセットアップし、日本語音声を生成する手順を解説しました。

### 次にできること

- **高品質モデル**: 0.6Bモデル以外に、より高品質な1.7Bモデルも試せます
- **ボイスデザイン**: 自分の音声を学習させたカスタムボイスの作成
- **WebUI**: ブラウザから操作できるインターフェースの構築

モデルIDを変更するだけで、別のモデルを試すことができます。

### 参考リンク

- [Qwen3-TTS 公式リポジトリ](https://github.com/QwenLM/Qwen3-TTS)
- [ModelScope Hub - Qwen3-TTS](https://www.modelscope.cn/models/Qwen/Qwen3-TTS-12Hz-0.6B-CustomVoice)
- [QwenLM 公式サイト](https://qwenlm.github.io/)
