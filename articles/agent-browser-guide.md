---
title: "agent-browserでブラウザ操作を自動化する方法"
emoji: "🌐"
type: "tech"
topics: ["playwright", "browser", "automation", "ai"]
published: false
---

agent-browserは、AIエージェントでの利用を想定して設計されたブラウザ自動化CLIツールです。内部的にはMicrosoft Playwrightを使用しており、自然言語プロンプトからの変換や、アクセシビリティツリーに基づいた一意な要素IDを提供することで、LLMがブラウザ操作を行う際の複雑性を低減します。

## agent-browserの特徴

- **Semantic Locators**: アクセシビリティ情報をベースに要素を特定できるため、DOM構造の変化に強い
- **JSON Output**: 機械可読な形式（JSON）でレスポンスを返すオプションがあり、AIエージェントとの統合が容易
- **Persistent Profiles**: Cookieやログインセッションを永続化・再利用できる

## インストールとセットアップ

```bash
npm install -g agent-browser
agent-browser install
```

これでツール本体とブラウザ（Chromium）がインストールされます。

## 使い方

### 基本的な使い方

まず、ブラウザを起動します。

```bash
agent-browser open https://example.com --profile ./profile --headed
```

- `--profile`: セッションを保存するディレクトリ
- `--headed`: ヘッドレスモード（UIなし）をオフにする

### 要素の操作

agent-browserはアクセシビリティツリーに基づいて要素を識別し、一意なID（`@e1`, `@e2`）を割り当てます。

```bash
# ページスナップショットを取得
agent-browser snapshot

# 要素をクリック
agent-browser click @e3

# テキストを入力
agent-browser type @e5 "検索キーワード"
```

## 認証に関する注意点

Googleなどのサービスでは、自動化ツールによるログインが「安全でないアプリ」としてブロックされる場合があります。

### 回避策

**方法1**: 既存のログイン済みブラウザを使用する

普段使いのブラウザを `--remote-debugging-port=9222` で起動し、CDP（Chrome DevTools Protocol）経由で接続します。

```bash
# 既存のChromeを起動（別ターミナル）
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222

# agent-browserから接続
agent-browser open https://gemini.google.com/app --cdp http://localhost:9222
```

**方法2**: 手動でログインする

一度手動でログインした後、プロファイルを再利用することで認証をスキップできます。

## 実用例: Geminiで画像を生成する

Google Geminiに自動でプロンプトを送信し、画像を生成する例を示します。

### 手順

1. **認証済みのセッションを使用する**
   - あらかじめブラウザでGeminiにログインしておく

2. **ページを開く**
   ```bash
   agent-browser open https://gemini.google.com/app --cdp http://localhost:9222
   ```

3. **プロンプトを入力して送信**
   ```bash
   agent-browser type @e5 "猫の画像を生成してください"
   agent-browser click @e8  # 送信ボタン
   ```

4. **生成された画像をダウンロード**
   ```bash
   # 画像のURLを取得し、curlで保存
   curl -o cat.webp "https://..."
   ```

## トラブルシューティング

| 問題 | 詳細 | 解決策 |
| :--- | :--- | :--- |
| **ログインブロック** | Googleなどで「安全でないアプリ」としてログインが拒否される | 既存のログイン済みブラウザを利用するか、CDP接続を行う |
| **日本語入力** | CLI経由での日本語入力が正常に動作しない場合がある | クリップボード経由の貼り付けや、JavaScriptによる `value` 設定で回避 |
| **要素が見つからない** | ページの読み込みが完了していない | `sleep` で待機するか、明示的に読み込みを待つ |

## まとめ

agent-browserは、AIエージェントでの利用を想定したブラウザ自動化ツールです。Semantic LocatorsやJSON Outputなど、プログラムからの利用を考慮した機能が備わっています。

認証の厳しいサイトを操作する場合は、CDP接続を活用することでスムーズに操作できます。

## 参考リンク

- [GitHub: vercel-labs/agent-browser](https://github.com/vercel-labs/agent-browser)
- [Microsoft Playwright](https://playwright.dev/)
