# YouTube 動画解析 & Google ドキュメント自動保存環境

このリポジトリは WSL2（Ubuntu）上で YouTube 動画をAI解析し、要約を Google ドキュメントに自動保存するための環境設定を記録しています。

---

## 概要

YouTube URL を渡すだけで `/youtube-summary` スキルが動画の内容を抽出・整理し、**Google ドキュメントに自動保存**します。
トランスクリプトは `youtube-transcript-api` で直接取得するため、**クッキーや認証は不要**です。

### できること
- YouTube 動画のトランスクリプトを自動取得
- 概要・トピック・詳細・結論の構造化出力
- 要約を Google ドキュメント（YouTube-Summary）に自動追記

---

## インストール済みコンポーネント

```
~/.local/bin/uv                               # パッケージマネージャー
~/.local/bin/nlm                              # NotebookLM CLI ツール
~/.local/bin/notebooklm-mcp                  # MCP サーバーバイナリ
~/.local/share/uv/tools/notebooklm-mcp-cli/  # パッケージ本体
~/.claude/commands/youtube-summary.md         # youtube-summary スキル
~/.google-credentials.json                    # Google サービスアカウントキー（非公開）
```

---

## 初回セットアップ手順

### 1. NotebookLM MCP インストール

```bash
uv tool install notebooklm-mcp-cli
```

### 2. Claude Code の MCP 設定

`~/.claude.json` の `mcpServers` に以下を追加：

```json
{
  "mcpServers": {
    "notebooklm-mcp": {
      "command": "/home/yugo2/.local/bin/notebooklm-mcp"
    }
  }
}
```

### 3. youtube-transcript-api / Google API ライブラリ

追加インストール不要。スキル実行時に `uv run --with` で自動取得されます。

### 4. Google ドキュメント連携のセットアップ

1. [Google Cloud Console](https://console.cloud.google.com) でプロジェクトを作成
2. **Google Docs API** と **Google Drive API** を有効化
3. サービスアカウントを作成し、JSON キーをダウンロード
4. JSON キーを `~/.google-credentials.json` に配置
5. Google ドキュメント（YouTube-Summary）をサービスアカウントのメールアドレスと **編集者** で共有

---

## 使い方：YouTube 動画を解析する

Claude Code に YouTube URL を貼り付けるだけで `/youtube-summary` スキルが自動起動します。

```
https://youtu.be/XXXXXXXXXXX
```

または明示的に：

```
/youtube-summary https://youtu.be/XXXXXXXXXXX
```

**出力：**
- チャットに要約を表示（概要・トピック・詳細・結論）
- Google ドキュメント（YouTube-Summary）に自動追記

---

## 内部フロー

1. `uv run --with youtube-transcript-api` でトランスクリプトを直接取得
2. `noembed.com` から動画タイトルを取得
3. Claude が要約・整理してチャットに出力
4. `uv run --with google-api-python-client --with google-auth` で Google ドキュメントに保存

---

## ファイル構成

| ファイル | 説明 |
|---|---|
| `~/.google-credentials.json` | Google サービスアカウントキー（`.gitignore` で除外） |
| `~/.claude/commands/youtube-summary.md` | スキル定義ファイル |
| `~/watched.md` | 視聴済み動画リンク集（タイトル・URL・日付） |

`watched.md` はスキル実行時に手動で更新する視聴履歴ファイルです。Google ドキュメントには要約全文が、`watched.md` にはリンクと概要が保存されます。
