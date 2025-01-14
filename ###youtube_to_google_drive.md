Sure! Below is the complete code as described, including an explanation of how to handle the `credentials.json` and `token.json` files.

---

### 1. GitHub Action Workflow File (`.github/workflows/auto_download.yml`)

```yaml
name: Auto Download and Upload YouTube Video to Google Drive

on:
  schedule:
    - cron: '*/5 * * * *'  # Run every 5 minutes
  workflow_dispatch:  # Manual trigger in case you want to run it manually

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
          pip install -r requirements.txt

      - name: Download and upload latest video
        run: |
          python download_video.py
        env:
          GOOGLE_DRIVE_CLIENT_ID: ${{ secrets.GOOGLE_DRIVE_CLIENT_ID }}
          GOOGLE_DRIVE_CLIENT_SECRET: ${{ secrets.GOOGLE_DRIVE_CLIENT_SECRET }}
          YOUTUBE_CHANNEL_URL: ${{ secrets.YOUTUBE_CHANNEL_URL }}
          LAST_DOWNLOADED_FILE: "last_downloaded.txt"
```

---

### 2. Python Script (`download_video.py`)

```python
import os
import sys
import logging
from pytube import Channel, YouTube
from googleapiclient.discovery import build
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.http import MediaFileUpload
from google.auth.transport.requests import Request
from google.auth.credentials import Credentials

# Configure logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# Function to authenticate Google Drive API
def authenticate_google_drive():
    SCOPES = ['https://www.googleapis.com/auth/drive.file']
    creds = None
    if os.path.exists('token.json'):
        creds = Credentials.from_authorized_user_file('token.json', SCOPES)
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file(
                'credentials.json', SCOPES)
            creds = flow.run_local_server(port=0)
        with open('token.json', 'w') as token:
            token.write(creds.to_json())
    return build('drive', 'v3', credentials=creds)

# Function to upload video to Google Drive
def upload_to_drive(file_path, file_name):
    try:
        drive_service = authenticate_google_drive()
        file_metadata = {'name': file_name}
        media = MediaFileUpload(file_path, mimetype='video/mp4')
        file = drive_service.files().create(body=file_metadata, media_body=media, fields='id').execute()
        logging.info(f"Video uploaded to Google Drive: {file_name}")
    except Exception as e:
        logging.error(f"Error uploading video to Google Drive: {str(e)}")

# Function to get the latest video from the channel
def get_latest_video(channel_url):
    try:
        channel = Channel(channel_url)
        latest_video_url = channel.video_urls[0]  # The first URL is the latest uploaded video
        video_id = latest_video_url.split('v=')[-1]
        logging.info(f"Latest video found: {video_id}")
        return latest_video_url, video_id
    except Exception as e:
        logging.error(f"Error fetching the latest video: {str(e)}")
        return None, None

# Function to download the video
def download_video(video_url, output_path):
    try:
        yt = YouTube(video_url)
        stream = yt.streams.get_highest_resolution()
        title = yt.title
        logging.info(f"Downloading video: {title}")
        stream.download(output_path)
        logging.info("Video download completed successfully.")
        return title
    except Exception as e:
        logging.error(f"Error downloading video: {str(e)}")
        return None

# Main logic
def main():
    channel_url = os.getenv('YOUTUBE_CHANNEL_URL')
    last_downloaded_file = os.getenv('LAST_DOWNLOADED_FILE')
    output_path = "Videos"

    # Create output directory if it doesn't exist
    if not os.path.exists(output_path):
        os.makedirs(output_path)

    # Read the last downloaded video ID
    if os.path.exists(last_downloaded_file):
        with open(last_downloaded_file, 'r') as f:
            last_downloaded_video_id = f.read().strip()
    else:
        last_downloaded_video_id = None

    # Get the latest video from the YouTube channel
    video_url, video_id = get_latest_video(channel_url)

    if video_url and video_id != last_downloaded_video_id:
        # Download the video if it's new
        video_title = download_video(video_url, output_path)

        if video_title:
            # Upload to Google Drive
            upload_to_drive(os.path.join(output_path, video_title), video_title)

            # Update the last downloaded video ID
            with open(last_downloaded_file, 'w') as f:
                f.write(video_id)
    else:
        logging.info("No new video to download.")

if __name__ == "__main__":
    main()
```

---

### 3. Google Drive API Authentication

#### Setting Up Google Drive API:
1. Go to the [Google Developer Console](https://console.developers.google.com/).
2. Create a new project or select an existing one.
3. Enable the **Google Drive API**.
4. Create **OAuth 2.0 credentials** for your project.
   - Go to the **Credentials** page.
   - Click on **Create Credentials** and choose **OAuth 2.0 Client IDs**.
   - Select **Desktop App** and follow the instructions.
   - Download the `credentials.json` file.

#### Store the `credentials.json` and `token.json` files:

- **`credentials.json`**: This file contains the OAuth 2.0 credentials and should be placed in your repository root directory or securely stored in GitHub Secrets.
- **`token.json`**: This file stores the access tokens and refresh tokens required for authentication and is generated automatically after the first successful authentication.

### Storing the Files Securely:

1. **Store `credentials.json` in GitHub Secrets**:
   - Add your `credentials.json` file content to a GitHub secret like `GOOGLE_DRIVE_CLIENT_ID` and `GOOGLE_DRIVE_CLIENT_SECRET`.
   
2. **Generate and Store `token.json`**:
   - After running the script for the first time and performing OAuth authentication, `token.json` will be generated.
   - You can either push it to GitHub securely or regenerate it after any session.

---

### 4. `requirements.txt`

```txt
pytube
google-api-python-client
google-auth-httplib2
google-auth-oauthlib
```

---

### Explanation:
- **GitHub Action** (`auto_download.yml`): This file defines the cron schedule and runs the Python script every 5 minutes.
- **Python Script** (`download_video.py`): Handles the process of fetching the latest YouTube video, downloading it, and uploading it to Google Drive.
- **Google API Authentication**: The `credentials.json` file contains your Google OAuth credentials. When the script first runs, it will generate a `token.json` file containing the user's access and refresh tokens.

