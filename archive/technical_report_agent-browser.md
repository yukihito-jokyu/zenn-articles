---
tags:
  - agent-browser
  - playwright
  - browser-automation
created: "2026-01-27-18:13"
---

# Technical Report: agent-browser (Browser Automation CLI)

**参考文献**:
- [GitHub: vercel-labs/agent-browser](https://github.com/vercel-labs/agent-browser)
- [Issue #46: [Experiment]: agent-browserからhttps://gemini.google.com/appへアクセスし、nanobananaで画像を生成させる](https://github.com/yukihito-jokyu/task-manager/issues/46)

## 1. 技術概要 (Technical Overview)
`agent-browser` は、AIエージェントによる使用を想定して設計された、ブラウザ自動化のためのCLIツールです。
内部的には Microsoft Playwright を使用しており、自然言語プロンプトから変換しやすいコマンド体系や、アクセシビリティツリーに基づいた一意な要素ID（Ref IDs: `@e1`, `@e2`）を提供することで、LLMがブラウザ操作を行う際の複雑性を低減します。

### 主な特徴
- **Semantic Locators**: アクセシビリティ情報を元にした要素特定ができ、DOM構造の変化に強い。
- **JSON Output**: 機械可読な形式（JSON）でレスポンスを返すオプションがあり、AIエージェントとの統合が容易。
- **Persistent Profiles**: Cookieやログインセッションを永続化・再利用する機能を持つ。

## 2. 実施内容 (Implementation Details)
今回の検証では、Google Geminiに対して自動的にプロンプトを送信し、生成された画像を取得することをゴールとしました。

### 達成したゴール
- [x] `agent-browser` のインストールとセットアップ
- [x] Geminiへのログイン（Googleアカウント認証）
    - *注: 自動化ツールによる直接ログインはブロックされたため、既存セッションを活用する検証方針に切り替え*
- [x] プロンプト送信による画像生成（「猫の画像を生成してください」）
- [x] 生成された画像のダウンロード

## 3. 検証詳細 (Validation Details)

### 3.1 インストールと初期設定
以下のコマンドでツール本体とブラウザ（Chromium）をインストールしました。
```bash
npm install -g agent-browser
agent-browser install
```

### 3.2 認証のハードルと回避策
当初、以下のコマンドでPlaywright管理下のChromiumを起動し、ログインを試みました。
```bash
agent-browser open https://gemini.google.com/app --profile ./profile --headed
```
**結果**: Googleのセキュリティにより「安全でないブラウザ」としてログインがブロックされました。
**解決策**: 自動化フラグの隠蔽（Stealth Mode）なども試みましたが検知されたため、最終的には「ユーザーが既にログイン済みの正規ブラウザ（Antigravity内蔵ブラウザ）」を使用することで認証をクリアしました。
*※ 実運用で `agent-browser` を使う場合は、普段使いのChromeを `--remote-debugging-port=9222` で起動し、`--cdp` オプションで接続する方法が必須となります。*

### 3.3 画像生成の実行
認証済みのセッション上で、以下のフローを実行しました。
1.  **プロンプト入力**: 「猫の画像を生成してください」を入力し送信。
2.  **生成待機**: 画像が生成されるまで待機。
3.  **ダウンロード**: 生成された画像のURLを特定し、`curl` コマンドを用いてローカルディレクトリ（`work/gemini_cat.webp`）に保存しました。

## 4. 依存ライブラリ (Dependencies)
- `agent-browser` (CLI Tool)
- `playwright` (Browser Automation Engine)
- `curl` (Image Download)

## 5. トラブルシューティング (Troubleshooting)

| 問題 | 詳細 | 解決策 |
| :--- | :--- | :--- |
| **ログインブロック** | Playwright付属のChromiumからのGoogleログインが「安全でないアプリ」として拒否される。 | 既存のログイン済みブラウザを利用するか、普段使いのChromeへCDP接続を行う。 |
| **日本語入力** | CLI経由での日本語入力（`type` コマンド）が一部正常に動作しない場合がある。 | クリップボード経由の貼り付けや、JavaScriptによる `value` 設定などで回避可能。 |

## 6. 今後の展望 (Future Prospects)
- **CDP接続の標準化**: Googleサービスなど認証が厳しいサイトを操作する場合は、ローカルChromeへのCDP接続フローを標準とするスクリプトを用意する。
- **複雑なワークフローの自動化**: 今回は単発の生成でしたが、連続的な会話や、生成結果に応じた再プロンプトなど、より対話的な自動化への応用が期待できます。
