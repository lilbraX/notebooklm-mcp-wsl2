# YouTube 動画の内容を取得して要約する

## 入力
- $ARGUMENTS: YouTube URL

## 実行手順

1. **トランスクリプトの取得**
   以下の Python スクリプトを Bash で実行してトランスクリプトを取得する。
   動画 ID は URL から抽出すること（例: `youtu.be/XXXXX` や `?v=XXXXX`）。

   ```bash
   uv run --with youtube-transcript-api python3 - <<'EOF'
   from youtube_transcript_api import YouTubeTranscriptApi
   from youtube_transcript_api._errors import NoTranscriptFound
   import sys

   video_id = "VIDEO_ID_HERE"

   try:
       # インスタンス化してから使う（v1.2.4+）
       api = YouTubeTranscriptApi()
       transcript_list = api.list(video_id)
       try:
           transcript = transcript_list.find_transcript(['ja', 'en'])
       except NoTranscriptFound:
           transcript = transcript_list.find_generated_transcript(['ja', 'en'])
       entries = transcript.fetch()
       text = " ".join([e.text for e in entries])
       print(text)
   except Exception as e:
       print(f"ERROR: {e}", file=sys.stderr)
       sys.exit(1)
   EOF
   ```

2. **動画タイトルの取得**
   WebFetch(url: "https://noembed.com/embed?url=${ARGUMENTS}") でタイトルを取得する。

3. **内容の要約・整理**
   取得したトランスクリプトをもとに、以下の形式で日本語で整理してまとめる：
   - 動画タイトル
   - 概要（3〜5文）
   - 主なトピック（箇条書き）
   - 詳細内容（見出しと箇条書きで構造化）
   - 重要なポイントや結論

## 注意事項
- トランスクリプトが取得できない場合（字幕なし動画など）はその旨を伝える
- NotebookLM は使わない（クッキー不要）
