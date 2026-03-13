# YouTube 動画の内容を取得して要約する

## 入力
- $ARGUMENTS: YouTube URL

## 実行手順

1. **トランスクリプトの取得**
   動画 ID を URL から抽出し（例: `youtu.be/XXXXX` や `?v=XXXXX`）、以下を実行：

   ```bash
   uv run --with youtube-transcript-api python3 - <<'EOF'
   from youtube_transcript_api import YouTubeTranscriptApi
   from youtube_transcript_api._errors import NoTranscriptFound
   import sys

   video_id = "VIDEO_ID_HERE"

   try:
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

4. **Google ドキュメントに保存**
   要約が完成したら、以下の Python スクリプトで Google ドキュメントに追記する。
   `TITLE`・`URL`・`SUMMARY` を実際の値に置き換えること。

   ```bash
   uv run --with google-api-python-client --with google-auth python3 - <<'EOF'
   from google.oauth2 import service_account
   from googleapiclient.discovery import build
   import datetime

   DOCUMENT_ID = '1foeyV5e-3eRy0rRjqEA_AgkWNfw3l0V0BIm4RrsBhL4'
   CREDENTIALS_FILE = '/home/yugo2/.google-credentials.json'

   title = "TITLE"
   url = "URL"
   summary = """SUMMARY"""
   date = datetime.date.today().isoformat()

   creds = service_account.Credentials.from_service_account_file(
       CREDENTIALS_FILE,
       scopes=['https://www.googleapis.com/auth/documents']
   )
   service = build('docs', 'v1', credentials=creds)

   # 既存のドキュメントの末尾インデックスを取得
   doc = service.documents().get(documentId=DOCUMENT_ID).execute()
   end_index = doc['body']['content'][-1]['endIndex'] - 1

   content = f"\n{'='*60}\n{title}\nURL: {url}\n日付: {date}\n\n{summary}\n"

   requests = [{
       'insertText': {
           'location': {'index': end_index},
           'text': content
       }
   }]

   service.documents().batchUpdate(
       documentId=DOCUMENT_ID,
       body={'requests': requests}
   ).execute()

   print("Google ドキュメントに保存しました")
   EOF
   ```

## 注意事項
- トランスクリプトが取得できない場合（字幕なし動画など）はその旨を伝える
- NotebookLM は使わない（クッキー不要）
- 要約はチャットにも表示し、Google ドキュメントにも保存する
