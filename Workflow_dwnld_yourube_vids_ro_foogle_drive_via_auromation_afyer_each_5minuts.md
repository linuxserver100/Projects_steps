

Here’s a detailed and corrected version of the setup to download YouTube videos in HD quality and upload them to Google Drive using GitHub Actions.

---

### Requirements:
1. **GitHub Repository**: To store your workflow and automate the tasks.
2. **YouTube API Key**: Required for fetching details from YouTube via the YouTube API.
3. **Google Drive API Credentials**: To interact with Google Drive.
4. **YouTube Video Download Tool**: `yt-dlp` to download YouTube videos in HD quality.

---

### Steps to Set Up the Workflow:

#### 1. **Set Up API Credentials:**

   - **Google Cloud Console**:
     - **Create a Project**: Go to the [Google Cloud Console](https://console.cloud.google.com/), create a new project.
     - **Enable APIs**: Enable the **YouTube Data API v3** and **Google Drive API** in the Google Cloud Console.
     - **Create OAuth 2.0 Credentials**:
       - In the **APIs & Services > Credentials** section, create **OAuth 2.0 Client IDs** for Google Drive. Select **Web application** as the application type.
       - Download the generated `client_secret.json` file which will be used for authenticating your application.
     - **Create YouTube API Key**: 
       - In the same section, create an API key for the **YouTube Data API v3** to fetch video details from a channel.
     - **Download `client_secret.json`**: This file contains OAuth credentials for accessing Google Drive and is required for authentication.

#### 2. **Add Secrets to GitHub Repository**:

   - Navigate to your **GitHub repository** and go to **Settings > Secrets and variables > Actions**.
   - Add the following secrets:
     - `YOUTUBE_API_KEY`: The YouTube API key obtained in the previous step.
     - `GOOGLE_DRIVE_CREDENTIALS_JSON`: The contents of your `client_secret.json` file, base64 encoded for security.
     - `CHANNEL_ID`: The YouTube channel ID from which you want to fetch the latest videos.

#### 3. **Create GitHub Actions Workflow**:

   - In your GitHub repository, create a new workflow file `.github/workflows/download_video.yml` with the following content:

```yaml
name: Download and Upload YouTube Videos to Google Drive

on:
  schedule:
    - cron: '*/5 * * * *'  # Runs every 5 minutes
  workflow_dispatch:

jobs:
  download-upload:
    runs-on: ubuntu-latest

    steps:
      - name: Check out the repository
        uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.x'

      - name: Install dependencies
        run: |
          pip install --upgrade pip
          pip install google-auth google-auth-oauthlib google-api-python-client yt-dlp requests

      - name: Fetch Latest YouTube Video
        id: youtube
        run: |
          python -c "
import requests
import os
YOUTUBE_API_KEY = os.getenv('YOUTUBE_API_KEY')
CHANNEL_ID = os.getenv('CHANNEL_ID')
url = f'https://www.googleapis.com/youtube/v3/search?key={YOUTUBE_API_KEY}&channelId={CHANNEL_ID}&order=date&part=snippet&type=video&maxResults=1'
response = requests.get(url)
video = response.json()['items'][0]['snippet']
video_id = video['resourceId']['videoId']
video_url = f'https://www.youtube.com/watch?v={video_id}'
print(f'Video URL: {video_url}')
print(f'::set-output name=video_url::{video_url}')
          "

      - name: Download HD Video
        id: download
        run: |
          VIDEO_URL=${{ steps.youtube.outputs.video_url }}
          yt-dlp -f 'bestaudio[ext=m4a]+bestvideo[height<=1080]' -o 'video.mp4' $VIDEO_URL

      - name: Authenticate with Google Drive
        run: |
          echo "${{ secrets.GOOGLE_DRIVE_CREDENTIALS_JSON }}" > client_secret.json
          python -m pip install --upgrade google-api-python-client google-auth-httplib2 google-auth-oauthlib

      - name: Upload to Google Drive
        run: |
          python -c "
import os
import pickle
from googleapiclient.discovery import build
from google_auth_oauthlib.flow import InstalledAppFlow
from google.auth.transport.requests import Request
from googleapiclient.http import MediaFileUpload

# Authenticate with Google Drive
SCOPES = ['https://www.googleapis.com/auth/drive.file']
creds = None
if os.path.exists('token.pickle'):
    with open('token.pickle', 'rb') as token:
        creds = pickle.load(token)
if not creds or not creds.valid:
    if creds and creds.expired and creds.refresh_token:
        creds.refresh(Request())
    else:
        flow = InstalledAppFlow.from_client_secrets_file('client_secret.json', SCOPES)
        creds = flow.run_local_server(port=0)
    with open('token.pickle', 'wb') as token:
        pickle.dump(creds, token)

# Upload video to Google Drive
service = build('drive', 'v3', credentials=creds)
file_metadata = {'name': 'video.mp4'}
media = MediaFileUpload('video.mp4', mimetype='video/mp4')
file = service.files().create(body=file_metadata, media_body=media, fields='id').execute()
print(f'File uploaded with ID: {file["id"]}')
          "
```

---

### Workflow Breakdown:

1. **Schedule**:
   - The workflow runs every 5 minutes based on the cron expression `*/5 * * * *`. You can manually trigger the workflow using the `workflow_dispatch` event.
   
2. **Dependencies**:
   - Installs the required dependencies such as `yt-dlp`, `google-auth`, `google-api-python-client`, and `requests` for downloading YouTube videos and interacting with Google Drive.

3. **YouTube API Call**:
   - The script fetches the most recent video from the specified YouTube channel using the YouTube Data API and extracts the video URL.

4. **Download HD Video**:
   - `yt-dlp` is used to download the highest quality video available (up to 1080p) with the audio in m4a format. The video is saved as `video.mp4`.

5. **Authenticate with Google Drive**:
   - Google OAuth 2.0 credentials are used to authenticate the app. The `client_secret.json` file is required for the authentication process.

6. **Upload to Google Drive**:
   - The video is uploaded to Google Drive using the Google Drive API. If this is the first time authentication occurs, it will generate a token (`token.pickle`) and store it for future use.
   - The video is uploaded with the name `video.mp4`.

---

### Notes:
- **API Credentials**: Ensure your YouTube API key and Google Drive OAuth credentials are correctly set up and added to GitHub Secrets.
- **Download Quality**: The download quality is limited to 1080p or lower (based on the available resolution). If you want to download higher resolutions, adjust the `-f` argument in the `yt-dlp` command.
- **OAuth Token**: The first time the script runs, you will need to authenticate manually using the OAuth flow. After the first successful authentication, the `token.pickle` will store the credentials for subsequent runs.

---

This configuration will fully automate the process of downloading the latest video from a YouTube channel in HD quality and uploading it to Google Drive using GitHub Actions.
