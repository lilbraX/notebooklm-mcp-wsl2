# CLAUDE.md — NotebookLM MCP セットアップガイド

このファイルは Claude Code（AI）向けの環境情報です。
このホームディレクトリ（WSL2 / Ubuntu）で NotebookLM MCP と youtube-transcript-api を使って YouTube 動画を解析するセットアップが完了しています。

---

## 環境構成

| 項目 | 値 |
|---|---|
| OS | WSL2 (Ubuntu on Windows) |
| Shell | bash |
| nlm CLI バイナリ | `/home/yugo2/.local/bin/nlm` |
| notebooklm-mcp バイナリ | `/home/yugo2/.local/bin/notebooklm-mcp` |
| パッケージ本体 | `/home/yugo2/.local/share/uv/tools/notebooklm-mcp-cli/` |

---

## `/youtube-summary` スキルの使い方

YouTube URL を Claude Code に貼り付けると自動的にスキルが起動する。
または明示的に：

```
/youtube-summary https://youtu.be/XXXX
```

### 内部フロー
1. `uv run --with youtube-transcript-api` で YouTube トランスクリプトを直接取得（クッキー不要）
2. `WebFetch` で `noembed.com` から動画タイトルを取得
3. Claude が直接トランスクリプトを要約・整理して出力

### 前提条件
- `uv` がインストール済みであること（`~/.local/bin/uv`）
- インターネット接続があること（youtube-transcript-api が自動インストールされる）
