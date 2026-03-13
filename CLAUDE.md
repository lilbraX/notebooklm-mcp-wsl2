# CLAUDE.md — YouTube 動画解析環境ガイド

このファイルは Claude Code（AI）向けの環境情報です。
このホームディレクトリ（WSL2 / Ubuntu）で YouTube 動画を解析し、Google ドキュメントに自動保存するセットアップが完了しています。

---

## 環境構成

| 項目 | 値 |
|---|---|
| OS | WSL2 (Ubuntu on Windows) |
| Shell | bash |
| uv | `~/.local/bin/uv` |
| nlm CLI バイナリ | `~/.local/bin/nlm` |
| notebooklm-mcp バイナリ | `~/.local/bin/notebooklm-mcp` |
| Google 認証情報 | `~/.google-credentials.json` |
| Google ドキュメント ID | `1foeyV5e-3eRy0rRjqEA_AgkWNfw3l0V0BIm4RrsBhL4` |

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
3. Claude がトランスクリプトを要約・整理してチャットに出力
4. `uv run --with google-api-python-client --with google-auth` で Google ドキュメントに自動保存

### 前提条件
- `uv` がインストール済みであること（`~/.local/bin/uv`）
- `~/.google-credentials.json` が存在すること（Google サービスアカウントの JSON キー）
- Google ドキュメント（YouTube-Summary）がサービスアカウントと共有済みであること

### watched.md について
`~/watched.md` は視聴済み動画のリンク集。スキル実行後に手動で更新する。
- Google ドキュメント（YouTube-Summary）: 要約全文を自動保存
- `watched.md`: タイトル・URL・日付を手動で記録

### Google ドキュメントの認証情報が切れた場合
サービスアカウントキーは期限なしのため通常は不要。
ファイルが消えた場合は Google Cloud Console から再ダウンロードして `~/.google-credentials.json` に配置する。
