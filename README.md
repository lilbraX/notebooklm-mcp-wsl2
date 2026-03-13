# NotebookLM MCP — WSL2 セットアップ & YouTube 動画解析環境

このリポジトリは WSL2（Ubuntu）上で YouTube 動画をAI解析するための環境設定を記録しています。

---

## 概要

YouTube URL を渡すだけで `/youtube-summary` スキルが動画の内容を抽出・整理します。
トランスクリプトは `youtube-transcript-api` で直接取得するため、**クッキーや認証は不要**です。

### できること
- YouTube 動画の内容を表・箇条書きで網羅的に抽出
- 概要・トピック・詳細・結論の構造化出力
- NotebookLM MCP による追加質問・ノートブック管理

---

## インストール済みコンポーネント

```
~/.local/bin/uv                               # パッケージマネージャー
~/.local/bin/nlm                              # NotebookLM CLI ツール
~/.local/bin/notebooklm-mcp                  # MCP サーバーバイナリ
~/.local/share/uv/tools/notebooklm-mcp-cli/  # パッケージ本体
~/.claude/commands/youtube-summary.md         # youtube-summary スキル
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

### 3. youtube-transcript-api

追加インストール不要。スキル実行時に `uv run --with youtube-transcript-api` で自動取得されます。

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

**出力例：**
- 概要（3〜5文）
- 主なトピック（箇条書き）
- 詳細内容（見出しと箇条書きで構造化）
- 重要なポイント・結論

---

## 内部フロー

1. `uv run --with youtube-transcript-api` でトランスクリプトを直接取得
2. `noembed.com` から動画タイトルを取得
3. Claude が直接要約・整理して出力
