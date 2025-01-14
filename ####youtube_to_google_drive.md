To create a GitHub Actions workflow that downloads the latest uploaded video from a YouTube channel and uploads it to Google Drive, you need to use a combination of tools and APIs, such as `youtube-dl` or `yt-dlp` for downloading videos, the Google Drive API to upload videos, and a cron-like scheduling mechanism in GitHub Actions to periodically check for new videos.

Here is a step-by-step guide, followed by the YAML for the GitHub Actions workflow.

### Prerequisites:
1. **Google API Credentials**: You’ll need to set up Google Drive API access, create a project in Google Cloud Console, and obtain a `client_secrets.json` file to authenticate.
2. **GitHub Secrets**: Add the necessary credentials as secrets in your GitHub repository. You'll need:
   - `GOOGLE_CLIENT_SECRETS` (the JSON credentials file for Google API)
   - `YOUTUBE_CHANNEL_ID` (the ID of the YouTube channel you want to download videos from)

3. **Python Dependencies**: You need `yt-dlp` (for downloading YouTube videos) and `google-auth`/`google-api-python-client` (for interacting with Google Drive).

### Workflow Steps:

1. **Set up a Google API project** to access Google Drive and get the necessary credentials.
2. **Set up your repository's secrets** for the credentials, including the YouTube channel ID and Google client secrets.
3. **Install dependencies** like `yt-dlp` and Google client libraries.
4. **Create a workflow** that:
   - Authenticates to the YouTube API.
   - Downloads the latest video.
   - Uploads the video to Google Drive.
   - Runs periodically (every few hours or daily).

### GitHub Actions Workflow YAML:

```yaml
name: Download Latest YouTube Video and Upload to Google Drive

on:
  schedule:
    - cron: '0 * * * *'  # Runs every hour
  workflow_dispatch:

jobs:
  download-and-upload:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.x'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install yt-dlp google-auth google-auth-oauthlib google-auth-httplib2 google-api-python-client

      - name: Authenticate with Google API
        env:
          GOOGLE_CLIENT_SECRETS: ${{ secrets.GOOGLE_CLIENT_SECRETS }}
        run: |
          echo "$GOOGLE_CLIENT_SECRETS" > client_secrets.json
          python -c "
import os
import google.auth
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build
from google.auth.transport.requests import Request

SCOPES = ['https://www.googleapis.com/auth/drive.file']

def authenticate():
    creds = None
    if os.path.exists('token.json'):
        creds, project = google.auth.load_credentials_from_file('token.json')
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file('client_secrets.json', SCOPES)
            creds = flow.run_local_server(port=0)
        with open('token.json', 'w') as token:
            token.write(creds.to_json())
    return creds

creds = authenticate()
drive_service = build('drive', 'v3', credentials=creds)
          "
      
      - name: Get Latest YouTube Video URL
        env:
          YOUTUBE_CHANNEL_ID: ${{ secrets.YOUTUBE_CHANNEL_ID }}
        run: |
          VIDEO_URL=$(python -c "
import yt_dlp

def get_latest_video(channel_id):
    ydl_opts = {'quiet': True}
    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        info_dict = ydl.extract_info(f'https://www.youtube.com/channel/{channel_id}', download=False)
        latest_video = info_dict['entries'][0]
        return latest_video['url']

video_url = get_latest_video('${{ secrets.YOUTUBE_CHANNEL_ID }}')
print(video_url)
          ")
          echo "VIDEO_URL=$VIDEO_URL" >> $GITHUB_ENV

      - name: Download Latest Video
        run: |
          yt-dlp -f best -o "latest_video.%(ext)s" ${{ env.VIDEO_URL }}

      - name: Upload Video to Google Drive
        run: |
          python -c "
import os
from googleapiclient.http import MediaFileUpload

def upload_video_to_drive(video_file):
    media = MediaFileUpload(video_file, mimetype='video/mp4', resumable=True)
    file_metadata = {'name': video_file}
    file = drive_service.files().create(media_body=media, body=file_metadata, fields='id').execute()
    print(f'File uploaded to Drive with ID: {file.get('id')}')

upload_video_to_drive('latest_video.mp4')
          "

```

### Explanation:
- **Triggering**: The workflow is triggered automatically every hour using a cron schedule (`'0 * * * *'`), and also can be triggered manually with `workflow_dispatch`.
- **Step-by-step**:
  1. **Set up Python and install dependencies**: Install `yt-dlp` for video downloading and Google API libraries for interacting with Google Drive.
  2. **Authenticate with Google Drive**: Load credentials and authenticate to Google APIs to upload files.
  3. **Get the latest YouTube video URL**: Using `yt-dlp`, the script fetches the URL of the latest uploaded video from the YouTube channel.
  4. **Download the video**: The latest video is downloaded using `yt-dlp`.
  5. **Upload to Google Drive**: The downloaded video is uploaded to Google Drive using the authenticated Google API.

### Important Notes:
- **YouTube Channel ID**: Replace `YOUTUBE_CHANNEL_ID` with the actual channel ID in the repository's secrets.
- **Google Client Secrets**: Make sure the `client_secrets.json` file from Google Cloud is correctly set up and added as a GitHub secret (`GOOGLE_CLIENT_SECRETS`).
- **Video formats**: The video is downloaded in the best available format (`yt-dlp -f best`) and uploaded as an `mp4` file. Adjust as needed.

With this setup, the workflow will check for new videos every hour, download them, and upload them to your Google Drive.
