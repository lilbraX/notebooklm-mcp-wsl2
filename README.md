# NotebookLM MCP — WSL2 セットアップ & YouTube 動画解析環境

このリポジトリは WSL2（Ubuntu）上で NotebookLM MCP を使い、YouTube 動画をAI解析するための環境設定を記録しています。

---

## 概要

[notebooklm-mcp-cli](https://github.com/szeider/notebooklm-mcp-cli) を Claude Code の MCP サーバーとして接続し、YouTube URL を渡すだけで動画の内容を NotebookLM 経由で抽出・整理できるワークフローです。

### できること
- YouTube 動画の内容を表・箇条書きで網羅的に抽出
- NotebookLM ノートブックの作成・管理
- 抽出した内容への追加質問（フォローアップ）

---

## インストール済みコンポーネント

```
~/.local/bin/nlm                        # CLI ツール
~/.local/bin/notebooklm-mcp            # MCP サーバーバイナリ
~/.local/share/uv/tools/notebooklm-mcp-cli/   # パッケージ本体
```

---

## 初回セットアップ手順

### 1. パッケージインストール

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

### 3. 認証（WSL2 の場合）

WSL2 ではブラウザが起動しないため、手動でクッキーを取得する。

**手順：**

1. Windows 側の Chrome で `https://notebooklm.google.com` にログイン
2. F12 → Network タブ → ページをリロード
3. リストの先頭のリクエストをクリック → Request Headers → `Cookie:` の値を全コピー
4. Claude Code 上で以下を実行（または直接 MCP ツールを呼び出す）：

```bash
nlm login --manual
# プロンプトが出たら ~/.nlm/cookies.txt のパスを入力
# または Ctrl+C して下記の手動方法へ
```

**推奨：save_auth_tokens を使う方法**

Claude Code に Cookie 文字列を渡して `save_auth_tokens` MCP ツールで保存する。
その後、プロファイルへの同期スクリプトを実行：

```bash
python3 << 'EOF'
import json, datetime
with open('/home/yugo2/.notebooklm-mcp-cli/auth.json') as f:
    auth = json.load(f)
with open('/home/yugo2/.notebooklm-mcp-cli/profiles/default/cookies.json', 'w') as f:
    json.dump(auth['cookies'], f, indent=2)
metadata = {"csrf_token": None, "session_id": None, "email": None, "build_label": None,
            "last_validated": datetime.datetime.now().isoformat()}
with open('/home/yugo2/.notebooklm-mcp-cli/profiles/default/metadata.json', 'w') as f:
    json.dump(metadata, f, indent=2)
print("Done")
EOF
```

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
- 動画の構成（章立て）
- 主要な主張・論点（箇条書き）
- 比較表・まとめ表
- 結論

---

## セッション切れ時の対処

「Authentication expired」エラーが出た場合：

1. Chrome の NotebookLM ページで Cookie を再取得
2. Claude Code に Cookie 文字列を渡して `save_auth_tokens` で更新
3. プロファイル同期スクリプトを実行
4. `refresh_auth` で再読み込み

詳細は [CLAUDE.md](./CLAUDE.md) を参照。

---

## 認証ファイルの場所

| ファイル | 説明 |
|---|---|
| `~/.notebooklm-mcp-cli/auth.json` | MCP が参照するメインの認証キャッシュ |
| `~/.notebooklm-mcp-cli/profiles/default/cookies.json` | プロファイル別クッキー（JSON 形式） |
| `~/.notebooklm-mcp-cli/profiles/default/metadata.json` | CSRF トークン・セッション ID 等 |
| `~/.nlm/cookies.txt` | nlm CLI 用 Netscape 形式クッキー |

**注意：** これらの認証ファイルはコミットしない（`.gitignore` で除外）。
