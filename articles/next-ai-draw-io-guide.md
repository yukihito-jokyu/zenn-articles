---
title: "Next AI Draw.ioでGLMモデルを使って図を自動生成する"
emoji: "📊"
type: "tech"
topics: ["nextjs", "ai", "drawio", "glm", "図解"]
published: false
---

Next AI Draw.ioは、人気のある作図ツール「draw.io」にAI生成機能を統合したNext.jsベースのWebアプリケーションです。自然言語による指示でフローチャートやアーキテクチャ図を自動生成・修正できます。

多数のAIプロバイダー（OpenAI、Anthropic、Google Geminiなど）に対応しており、OpenAI互換のエンドポイントを持つモデル（GLMなど）も利用可能です。

## 特徴

- 自然言語でフローチャートやアーキテクチャ図を自動生成
- 既存の画像やPDFからの図解生成をサポート
- 複数のAIプロバイダーに対応
- OpenAI互換APIを使用したカスタムモデルの利用が可能

## 環境構築

### 1. リポジトリのクローン

```bash
git clone https://github.com/DayuanJiang/next-ai-draw-io
cd next-ai-draw-io
```

### 2. 依存ライブラリのインストール

```bash
npm install
```

### 3. 環境変数の設定

`.env.local` ファイルを作成し、以下の設定を追加します。

#### OpenAIの場合

```bash
AI_PROVIDER="openai"
AI_MODEL="gpt-4"
OPENAI_API_KEY="sk-..."
```

#### GLM (Zhipu AI) の場合

OpenAI互換モードを利用して設定します。

```bash
AI_PROVIDER="openai"
AI_MODEL="zai/glm-4.7"
OPENAI_API_KEY="********"  # GLM API Key
OPENAI_BASE_URL="https://domain.name/v1"  # LiteLLMを使用
```

### 4. アプリケーションの起動

```bash
npm run dev
```

ブラウザで `http://localhost:6002` にアクセスします。

## 使い方

### 1. 図の生成

チャットボットに自然言語で指示を入力します。

**プロンプト例**:
```
Draw a simple flowchart for a login process
```

AIが構造化データを生成し、キャンバス上にフローチャートが自動で描画されます。

### 2. 既存の画像から図を生成

画像やPDFをアップロードすると、そこから図解を生成できます。

## 動作検証

GLM-4.7モデルを使用して、シンプルなフローチャートの自動生成を試しました。

**結果**:
- チャットボットがプロンプトを受け取り、GLM-4が構造化データを生成
- キャンバス上にフローチャート（Start → Input → Logic Check → End）が正しく描画された

GLM-4は本アプリケーションのバックエンドモデルとして十分に機能し、複雑なセットアップなしでOpenAI互換モード経由で利用可能です。

## 注意点

`AI_PROVIDER=openai` を指定しつつ、`OPENAI_BASE_URL` を上書きすることで、多くのOpenAI互換AIサービスを利用できます。この汎用性の高さは大きな利点です。

## 今後の活用

- **Docker運用**: チームで共有する場合はDockerコンテナ化を検討
- **MCP活用**: Claude Desktopなどとの連携を検証し、IDE連携を強化
- **カスタムモデル**: ローカルLLM（Ollama）との連携検証

## 参考リンク

- [GitHub - DayuanJiang/next-ai-draw-io](https://github.com/DayuanJiang/next-ai-draw-io)
- [Zhipu AI (GLM) API Documentation](https://open.bigmodel.cn/dev/api)
- [draw.io](https://www.draw.io/)
