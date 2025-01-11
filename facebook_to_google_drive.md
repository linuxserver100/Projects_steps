Certainly! Below is the complete and rewritten code, which ensures that it runs in the main branch of your GitHub repository:

### Prerequisites
- **GitHub repository** (main branch).
- **Google Drive API** credentials to upload files.
- **Facebook API** access token to fetch videos.
- **GitHub Actions secrets** for securely storing your credentials (API keys, access tokens).

This approach doesn't use Python and relies on GitHub Actions workflows with shell scripting (bash). Here's how to set it up:

### Steps

1. **Create Google Drive API Credentials**:
   - Visit the [Google Developer Console](https://console.developers.google.com/).
   - Create a new project.
   - Enable the Google Drive API.
   - Create OAuth 2.0 credentials (download the `credentials.json` file).
   - Upload `credentials.json` to GitHub repository secrets.

2. **Facebook API Access Token**:
   - Generate a Facebook access token using the [Facebook Graph API Explorer](https://developers.facebook.com/tools/explorer/).

3. **GitHub Secrets**:
   - Add the following secrets in your GitHub repository:
     - `GOOGLE_DRIVE_CREDENTIALS` (contents of `credentials.json`).
     - `FACEBOOK_ACCESS_TOKEN`.

4. **GitHub Actions Workflow**:
   - In your GitHub repository, create a `.github/workflows/facebook_video_to_drive.yml` file with the following content:

```yaml
name: Download Facebook Videos to Google Drive

on:
  schedule:
    - cron: "*/5 * * * *"  # Every 5 minutes
  push:
    branches:
      - main  # Ensures that it runs on push to the main branch

jobs:
  download_video:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Repository
      uses: actions/checkout@v2

    - name: Set up Google Drive Credentials
      run: |
        echo "${{ secrets.GOOGLE_DRIVE_CREDENTIALS }}" > credentials.json
        chmod 600 credentials.json

    - name: Install Dependencies
      run: |
        sudo apt-get update
        sudo apt-get install -y jq curl

    - name: Authenticate Google Drive API
      run: |
        google-drive-ocamlfuse -headless -id ${GOOGLE_CLIENT_ID} -secret ${GOOGLE_CLIENT_SECRET} -refresh-token ${GOOGLE_REFRESH_TOKEN}

    - name: Get Latest Facebook Video
      run: |
        # Fetch the latest video from Facebook
        VIDEO_DATA=$(curl -s "https://graph.facebook.com/v12.0/{facebook-page-id}/videos?access_token=${{ secrets.FACEBOOK_ACCESS_TOKEN }}")
        VIDEO_ID=$(echo $VIDEO_DATA | jq -r '.data[0].id')
        VIDEO_URL="https://graph-video.facebook.com/v2.12/${VIDEO_ID}/source"

        # Check if video has been already downloaded (avoid duplicates)
        VIDEO_FILENAME="video_${VIDEO_ID}.mp4"
        if [[ ! -f "/mnt/gdrive/${VIDEO_FILENAME}" ]]; then
          echo "Downloading video..."
          curl -L -o "${VIDEO_FILENAME}" "${VIDEO_URL}"

          # Upload to Google Drive
          echo "Uploading video to Google Drive..."
          gdrive upload --file "${VIDEO_FILENAME}" --parent "root"

          # Clean up
          rm "${VIDEO_FILENAME}"
        else
          echo "Video already downloaded"
        fi

    - name: Clean up
      run: |
        rm -f credentials.json
```

### Breakdown of the Workflow:
- **Scheduled Trigger** (`*/5 * * * *`): This cron expression ensures that the action runs every 5 minutes.
- **Push Trigger** (`push` to `main` branch): The workflow will also run when there is a push to the `main` branch.
- **Google Drive Authentication**: The Google credentials are retrieved from the repository secrets.
- **Fetching Facebook Videos**: The latest videos are fetched using the Facebook Graph API.
- **Download and Upload**: If the video hasn’t been downloaded yet, it will be downloaded and uploaded to Google Drive.
- **Duplicate Check**: Before downloading, the script checks if the video already exists on Google Drive by checking the file name (`video_${VIDEO_ID}.mp4`).

### Important Notes:
- Replace `{facebook-page-id}` with your actual Facebook Page ID in the API URL.
- This setup assumes you're using `gdrive`, a simple command-line tool to interact with Google Drive. You can install it by adding it to the `Install Dependencies` step or using other tools like `rclone`.
- You may need to adjust authentication details (`GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `GOOGLE_REFRESH_TOKEN`) depending on your setup.
- The `gdrive` tool needs access to Google Drive through OAuth2 authentication.

### Setup:
1. Upload your Google Drive credentials to GitHub Secrets.
2. Set the Facebook access token in GitHub Secrets.
3. Adjust the `facebook-page-id` and ensure the `gdrive` tool is installed in the Action.

By following these steps, the GitHub Actions workflow will automatically run every 5 minutes to fetch the latest Facebook videos and upload them to Google Drive, preventing duplicate downloads.
