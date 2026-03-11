# CLAUDE.md — NotebookLM MCP セットアップガイド

このファイルは Claude Code（AI）向けの環境情報です。
このホームディレクトリ（WSL2 / Ubuntu）で NotebookLM MCP を使って YouTube 動画を解析するセットアップが完了しています。

---

## 環境構成

| 項目 | 値 |
|---|---|
| OS | WSL2 (Ubuntu on Windows) |
| Shell | bash |
| nlm CLI バイナリ | `/home/yugo2/.local/bin/nlm` |
| notebooklm-mcp バイナリ | `/home/yugo2/.local/bin/notebooklm-mcp` |
| パッケージ本体 | `/home/yugo2/.local/share/uv/tools/notebooklm-mcp-cli/` |
| クッキーファイル (nlm 用) | `~/.nlm/cookies.txt` |
| 認証キャッシュ (MCP 用) | `~/.notebooklm-mcp-cli/auth.json` |
| プロファイル cookies | `~/.notebooklm-mcp-cli/profiles/default/cookies.json` |
| プロファイル metadata | `~/.notebooklm-mcp-cli/profiles/default/metadata.json` |

---

## 認証フロー（WSL2 での重要な注意点）

### 問題
WSL2 環境ではブラウザが起動しないため、`nlm login`（自動認証）は使えない。

### 解決策：手動クッキー配置

1. **Windows 側の Chrome** で `https://notebooklm.google.com` にログイン
2. Chrome DevTools を開く（F12）→ **Network** タブ
3. ページをリロードし、リストの先頭のリクエストをクリック
4. **Request Headers** から `Cookie:` の値を全コピー（`key=value; key2=value2; ...` 形式）
5. WSL2 ターミナルで `save_auth_tokens` MCP ツールに渡す（下記参照）

### cookies.txt の手動作成（nlm CLI 用）
`~/.nlm/cookies.txt` は Netscape 形式で作成する：
```
# Netscape HTTP Cookie File
.google.com	TRUE	/	TRUE	<expiry>	<name>	<value>
```
ただし、**MCP ツール経由の `save_auth_tokens` で自動生成される `auth.json` と `profiles/default/cookies.json` が正であり、こちらを優先して使うこと。**

---

## クッキーの更新手順（セッション切れ時）

セッションが切れると以下のエラーが出る：
```
Authentication expired. Run 'nlm login' in your terminal to re-authenticate.
```

### 手順

1. Chrome で `notebooklm.google.com` にアクセス（必要なら再ログイン）
2. F12 → Network タブ → ページリロード → 先頭リクエスト → Request Headers → `Cookie:` の値をコピー
3. Claude Code から MCP ツールを呼び出す：

```
mcp__notebooklm-mcp__save_auth_tokens(
  cookies="<コピーした Cookie: ヘッダーの値>"
)
```

4. `profiles/default/cookies.json` と `metadata.json` を同期（save_auth_tokens は auth.json に書くが、プロファイルには自動反映されない場合がある）：

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

5. MCP の認証をリフレッシュ：

```
mcp__notebooklm-mcp__refresh_auth()
```

### よくあるトラブル：cookies.json の破損
`Illegal header value b"# Netscape HTTP Cookie File..."` というエラーが出た場合、
`profiles/default/cookies.json` にクッキーファイルの内容が丸ごと入ってしまっている。
上記の Python スクリプトで `~/.nlm/cookies.txt` から再パースして修正できる。

---

## `/youtube-summary` スキルの使い方

YouTube URL を Claude Code に貼り付けると自動的にスキルが起動する。
または明示的に：

```
/youtube-summary https://youtu.be/XXXX
```

### 内部フロー
1. `WebFetch` で `noembed.com` から動画タイトルを取得
2. `notebook_create` でそのタイトルのノートブックを作成
3. `source_add(source_type="url", url=<YouTube URL>, wait=True, wait_timeout=180)` でソース追加
4. `notebook_query` で「動画の内容をすべて網羅的に抽出し、表と箇条書きで整理してください」と質問

### 前提条件
- NotebookLM MCP が Claude Code の MCP サーバーとして設定済みであること
- 有効なセッションクッキーが `auth.json` および `profiles/default/cookies.json` に存在すること
