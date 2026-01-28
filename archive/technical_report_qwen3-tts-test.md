---
tags:
  - Qwen3-TTS
  - AI
  - VoiceSynthesis
created: "2026-01-28-19:30"
---

# Technical Report: Qwen3-TTS Setup & Verification

## 1. 概要
Github Issue #48 「Qwen3-TTSを試す」に基づき、Mac (Apple Silicon) 環境でのローカル実行環境を構築・検証しました。
ターゲットOSS: [QwenLM/Qwen3-TTS](https://github.com/QwenLM/Qwen3-TTS)

主な成果:
- `uv` を用いた環境構築手順の確立。
- Mac (MPS) 環境での動作確認用コードの作成。
- `modelscope` を用いたモデルダウンロードの最適化（HuggingFaceの転送速度問題への対応）。

## 2. 環境構築手順 (Setup)

以下の手順で再現可能です。

### 2.1 必須ツールと依存関係
- `uv` (Python Package Manager)
- `brew` (Homebrew)
- `sox` (音声処理ライブラリ)

```bash
# soxのインストール（必須）
brew install sox

# 作業ディレクトリ作成と初期化
mkdir -p work/qwen3-tts-demo
cd work/qwen3-tts-demo
uv init --python 3.12
uv add qwen-tts modelscope soundfile
```

### 2.2 モデルダウンロード
モデルサイズが大きいため（0.6Bモデルで約1GB+）、回線環境によってはHuggingFaceからのダウンロードが非常に遅くなります。
Alibaba提供の `modelscope` を利用することで改善が見込まれますが、それでも時間を要する場合があります。

以下のスクリプトを実行してモデルをダウンロードしてください。途中で止まった場合はリトライ（`ctrl+c` で停止して再実行）が可能です。

```bash
# ダウンロード用スクリプトの実行 (-u オプションでログをリアルタイム表示)
uv run python -u download_script.py
```
※ `download_script.py` は既に作成済みです。

### 2.3 音声生成テスト
モデルダウンロード完了後、以下の検証用スクリプトを実行してください。

```bash
uv run run_tts_test.py
```

このスクリプトは以下の処理を行います:
1. MPS (Apple Silicon GPU) の利用可否を自動判定。
2. ローカルにダウンロードされたモデルをロード。
3. 日本語テキスト「こんにちは、これはテストです...」を音声(wav)に変換し、`output_test.wav` として保存。

## 3. 検証結果とトラブルシューティング

### MPS (Metal) 対応
Code上で `device="mps"` を指定することで、MacのGPUを活用可能であることを確認しました。
ただし、`flash-attn` はMacではサポートされていないため、`sdpa` (Scale Dot Product Attention) または通常のAttention実装が使用されます。これにより推論速度はCUDA環境より劣る可能性がありますが、実用範囲内です。

### ダウンロードの課題
- **現象**: HuggingFaceからのダウンロードが極めて低速、またはタイムアウトする。
- **対応**: `modelscope` ライブラリを採用。また、ダウンロードプロセスが中断・競合した場合にLockファイル (`~/.cache/modelscope/hub/.lock` 等) が残留し、再実行できなくなる現象を確認しました。
- **解決策**: Lockファイルの削除を行い、再実行することでダウンロードが進行することを確認済です。

### Soxの依存性
`qwen-tts` は内部で `sox` コマンドを利用しているため、Mac環境では別途インストールが必要でした。これがないと実行時にエラーとなります。

## 4. 今後の展望
今回は `0.6B` (軽量版) モデルを使用しましたが、より高品質な `1.7B` モデルや `Voice Design` モデルへの切り替えも、スクリプト内のモデルIDを変更するだけで可能です。
WebUI (`qwen-tts-demo`) については、HTTPS証明書の生成やポート設定が必要なため、まずは上記のPythonスクリプトでの基本動作確認を推奨します。

---
**参考文献**:
- [Qwen3-TTS 公式リポジトリ](https://github.com/QwenLM/Qwen3-TTS)
- [ModelScope Hub](https://www.modelscope.cn/models/Qwen/Qwen3-TTS-12Hz-0.6B-CustomVoice)
