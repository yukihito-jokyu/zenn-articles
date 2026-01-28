---
tags:
  - Next.js
  - AI
  - Draw.io
  - GLM
  - Visualization
created: "2026-01-27-23:48"
---

# Technical Report: Next AI Draw.io

## 技術概要
**Next AI Draw.io** は、人気のある作図ツール [draw.io](https://www.draw.io/) にAI生成機能を統合したNext.jsベースのWebアプリケーションです。自然言語による指示でフローチャートやアーキテクチャ図を自動生成・修正できるほか、既存の画像やPDFからの図解生成もサポートしています。
多数のAIプロバイダー（OpenAI, Anthropic, Google Gemini等）に対応しており、OpenAI互換のエンドポイントを持つモデル（GLMなど）も利用可能です。

## 実施内容
*   **目的**: ローカル環境でNext AI Draw.ioを構築し、**GLM (Zhipu AI)** モデルを用いてAI作図が可能か検証する。
*   **成果**:
    *   ローカル開発環境 (`npm run dev`) の構築完了。
    *   GLM-4モデル (OpenAI互換モード) による作図機能の動作確認。
    *   「シンプルなログインプロセスのフローチャート」の自動生成に成功。

## 検証詳細

### 1. 環境構築
*   リポジトリ: `https://github.com/DayuanJiang/next-ai-draw-io`
*   セットアップ手順:
    1.  `git clone` および `npm install`
    2.  `.env.local` の作成
    3.  `npm run dev` で起動 (Port 6002)

### 2. LiteLLM (GLM-4.7) の設定
公式ドキュメントには明示されていませんが、OpenAI互換設定を利用してLiteLLM(GLM-4.7)を設定しました。

```bash
# .env.local
AI_PROVIDER="openai"
AI_MODEL="zai/glm-4.7"
OPENAI_API_KEY="********" # GLM API Key
OPENAI_BASE_URL="https://domain.name/v1" # LiteLLM使用
```

### 3. 動作検証
*   **プロンプト**: "Draw a simple flowchart for a login process"
*   **挙動**:
    *   チャットボットがプロンプトを受け付け、GLM-4が構造化データを生成。
    *   キャンバス上にフローチャート（Start -> Input -> Logic Check -> End）が描画された。
*   **結論**: GLM-4は本アプリケーションのバックエンドモデルとして十分に機能し、複雑なセットアップなしでOpenAI互換モード経由で利用可能である。

## 依存ライブラリ
*   **Node.js**: ランタイム
*   **Next.js**: アプリケーションフレームワーク
*   **draw.io**: 作図コア機能
*   **LangChain / AI SDK**: （推測）AIプロバイダーとの通信

## トラブルシューティング
*   **特になし**: 今回の検証ではスムーズに動作した。
*   **注意点**: `AI_PROVIDER=openai` を指定しつつ、`OPENAI_BASE_URL` を上書きすることで、多くのOpenAI互換AIサービスが利用できる汎用性の高さが確認できた。

## 今後の展望
*   **Docker運用**: チームで共有する場合はDockerコンテナ化（Plan B）を検討。
*   **MCP活用**: Claude Desktop等との連携（MCPサーバー機能）を検証し、IDE連携を強化する。
*   **カスタムモデル**: ローカルLLM (Ollama) との連携検証。

## 参考文献
*   [GitHub - DayuanJiang/next-ai-draw-io](https://github.com/DayuanJiang/next-ai-draw-io)
*   [Zhipu AI (GLM) API Documentation](https://open.bigmodel.cn/dev/api)
