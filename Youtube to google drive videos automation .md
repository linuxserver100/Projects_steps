Sure! Here's the complete rewritten process, following the same structure with all the steps included, and making sure the process for handling the JSON credentials files is clear:

---

### Step-by-Step Process:

#### Step 1: Create a GitHub Repository
1. Go to GitHub and create a new repository (e.g., `youtube-to-google-drive`).
2. Clone this repository to your local machine or use GitHub's web editor to add files.

#### Step 2: Set up Google APIs
1. **Create a Project in Google Cloud Console**:
   - Go to [Google Cloud Console](https://console.cloud.google.com/).
   - Create a new project (e.g., `youtube-to-drive`).

2. **Enable YouTube Data API & Google Drive API**:
   - In the **APIs & Services** section, search for and enable both the **YouTube Data API v3** and **Google Drive API**.

3. **Create Credentials**:
   - For the **YouTube API**, create **OAuth 2.0 credentials** (to access YouTube).
   - For **Google Drive API**, create **OAuth 2.0 credentials** (to upload files to Drive).
   - Instead of downloading the JSON files, you will copy the content of these files and store them as GitHub secrets.

#### Step 3: Prepare Your Project Locally
1. **Create the GitHub Actions Workflow File**:
   - In your GitHub repository, create the `.github/workflows` directory if it doesn't already exist.
   - Inside `.github/workflows/`, create a YAML file (e.g., `youtube-to-drive.yml`).

2. **Write a Python Script to Fetch YouTube Videos and Upload to Google Drive**:
   - In the root of your repository, create a file like `main.py` that contains the logic for:
     - **Authenticating with Google APIs** (using OAuth2 for both YouTube and Google Drive).
     - **Fetching the latest videos** from a specified YouTube channel.
     - **Uploading the videos** to Google Drive.

3. **Python Libraries**:
   You’ll need `google-auth`, `google-api-python-client`, `google-auth-oauthlib`, and `google-auth-httplib2`. You can install them by adding a `requirements.txt` file with these dependencies.

4. **Store OAuth 2.0 Credentials in GitHub Secrets**:
   - Go to the repository on GitHub.
   - In **Settings**, under **Secrets** and **Variables**, create secrets for:
     - `GOOGLE_DRIVE_CREDENTIALS` (Store the raw content of your Google Drive OAuth 2.0 credentials JSON).
     - `YOUTUBE_API_CREDENTIALS` (Store the raw content of your YouTube API OAuth 2.0 credentials JSON).

#### Step 4: Create the GitHub Actions Workflow

In your `.github/workflows/youtube-to-drive.yml`, create the following:

```yaml
name: YouTube to Google Drive

on:
  schedule:
    - cron: '*/5 * * * *'  # Runs every 5 minutes
  push:
    branches:
      - main  # Ensure it runs for the main branch

jobs:
  upload_videos:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
      
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: 3.x

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
      
      - name: Authenticate with Google APIs
        run: |
          echo "${{ secrets.GOOGLE_DRIVE_CREDENTIALS }}" > /tmp/google_drive_credentials.json
          echo "${{ secrets.YOUTUBE_API_CREDENTIALS }}" > /tmp/youtube_credentials.json
        env:
          GOOGLE_DRIVE_CREDENTIALS: ${{ secrets.GOOGLE_DRIVE_CREDENTIALS }}
          YOUTUBE_API_CREDENTIALS: ${{ secrets.YOUTUBE_API_CREDENTIALS }}

      - name: Run Python script
        run: python main.py
```

### Explanation of the Workflow:

1. **Triggers**:
   - The `on.schedule` section specifies that the workflow should run every 5 minutes (as defined by `*/5 * * * *` cron expression).
   - The `push` trigger ensures the workflow runs when there are changes pushed to the `main` branch.

2. **Jobs**:
   - The job `upload_videos` runs on the latest Ubuntu environment.
   
3. **Steps**:
   - `actions/checkout@v2`: This step checks out the repository so that the workflow can access your files.
   - `actions/setup-python@v2`: Sets up Python 3.x for the environment.
   - `Install dependencies`: Installs the necessary Python libraries from `requirements.txt`.
   - `Authenticate with Google APIs`: Uses GitHub secrets to authenticate with both YouTube and Google Drive APIs by creating a credentials file for each service. It places these credentials in the `/tmp` directory on the runner using the `echo` command.
   - `Run Python script`: Executes the `main.py` script to fetch the latest videos from YouTube and upload them to Google Drive.

#### Step 5: Write the Python Script (main.py)
In the `main.py` script, use the Google APIs to fetch YouTube videos and upload them to Google Drive. Here's a simple outline:

```python
import os
import google.auth
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build
from googleapiclient.http import MediaFileUpload

# Authenticate to YouTube
def authenticate_youtube():
    credentials_file = '/tmp/youtube_credentials.json'
    creds = InstalledAppFlow.from_client_secrets_file(credentials_file, ['https://www.googleapis.com/auth/youtube.readonly']).run_local_server(port=0)
    youtube = build('youtube', 'v3', credentials=creds)
    return youtube

# Authenticate to Google Drive
def authenticate_drive():
    credentials_file = '/tmp/google_drive_credentials.json'
    creds = InstalledAppFlow.from_client_secrets_file(credentials_file, ['https://www.googleapis.com/auth/drive.file']).run_local_server(port=0)
    drive = build('drive', 'v3', credentials=creds)
    return drive

# Fetch the latest YouTube videos
def get_latest_videos(youtube, channel_id):
    request = youtube.search().list(
        part="snippet",
        channelId=channel_id,
        order="date",
        maxResults=5
    )
    response = request.execute()
    return response['items']

# Upload video to Google Drive
def upload_to_drive(drive, video_file_path):
    file_metadata = {'name': os.path.basename(video_file_path)}
    media = MediaFileUpload(video_file_path)
    file = drive.files().create(body=file_metadata, media_body=media, fields='id').execute()
    return file

# Main function
def main():
    youtube = authenticate_youtube()
    drive = authenticate_drive()
    
    # Replace with the actual YouTube Channel ID
    channel_id = "YOUR_YOUTUBE_CHANNEL_ID"
    
    videos = get_latest_videos(youtube, channel_id)
    
    for video in videos:
        video_file_path = download_video(video)  # You need to implement video download
        upload_to_drive(drive, video_file_path)

if __name__ == '__main__':
    main()
```

### Explanation of the Python Script:
- **authenticate_youtube** and **authenticate_drive** handle authentication to the YouTube and Google Drive APIs, respectively. They use the credentials stored in the `/tmp` directory, which are created during the workflow by echoing the GitHub secrets into JSON files.
- **get_latest_videos** fetches the latest videos from the YouTube channel.
- **upload_to_drive** uploads a downloaded video to Google Drive.
- **download_video** would be a function where you would download the video to a local or temporary directory (you might need `youtube-dl` or a similar library).

#### Step 6: Add Required Python Packages to `requirements.txt`

```txt
google-auth
google-auth-oauthlib
google-auth-httplib2
google-api-python-client
```

#### Step 7: Test Your Workflow
- After setting up the workflow, commit and push the changes to the `main` branch.
- The GitHub Actions workflow will trigger every 5 minutes as per the cron schedule.
- Check the **Actions** tab in your GitHub repository to monitor the workflow's progress.

---

### How to Get Google Drive and YouTube API Keys:

1. **For YouTube API**:
   - Go to [Google Developers Console](https://console.developers.google.com/).
   - Create a new project (if you don't have one).
   - In the **Library**, search for **YouTube Data API v3** and enable it.
   - Go to **Credentials**, create **OAuth 2.0 credentials** for accessing the API, and copy the content of the downloaded credentials JSON file to store it as `YOUTUBE_API_CREDENTIALS` in GitHub secrets.

2. **For Google Drive API**:
   - Go to the [Google Developers Console](https://console.developers.google.com/).
   - Create a new project (if you don't have one).
   - In the **Library**, search for **Google Drive API** and enable it.
   - Go to **Credentials**, create **OAuth 2.0 credentials** for accessing the API, and copy the content of the downloaded credentials JSON file to store it as `GOOGLE_DRIVE_CREDENTIALS` in GitHub secrets.

These credentials will now be available as GitHub secrets and used directly within your workflow.

---

This should cover all the steps and ensure that the credentials files are handled correctly within the GitHub Actions workflow.
