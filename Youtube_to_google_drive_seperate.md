Here is the rewritten version of your code with **all the same content as requested**. I've also added clarification on where to obtain the refresh tokens for the APIs.

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
   - Instead of downloading the JSON files, copy the individual keys (e.g., `client_id`, `client_secret`, `refresh_token`) and store them as separate GitHub secrets.
   - **To obtain the refresh token**:
     1. Use tools like [OAuth Playground](https://developers.google.com/oauthplayground/).
     2. Authenticate using your client ID and client secret.
     3. Authorize scopes for **YouTube Data API v3** and **Google Drive API**.
     4. Exchange the authorization code for a refresh token.

#### Step 3: Prepare Your Project Locally
1. **Create the GitHub Actions Workflow File**:
   - In your GitHub repository, create the `.github/workflows` directory if it doesn't already exist.
   - Inside `.github/workflows/`, create a YAML file (e.g., `youtube-to-drive.yml`).

2. **Write a JavaScript Script to Fetch YouTube Videos and Upload to Google Drive**:
   - In the root of your repository, create a file like `main.js` that contains the logic for:
     - **Authenticating with Google APIs** (using OAuth2 for both YouTube and Google Drive).
     - **Fetching the latest videos** from a specified YouTube channel.
     - **Uploading the videos** to Google Drive.

3. **Node.js Libraries**:
   - You’ll need `googleapis`, `google-auth-library`, and `ytdl-core`. Install them by creating a `package.json` file and running `npm install`.

4. **Store OAuth 2.0 Credentials as Individual Secrets in GitHub**:
   - Go to the repository on GitHub.
   - In **Settings**, under **Secrets and Variables**, create secrets for:
     - `GOOGLE_DRIVE_CLIENT_ID`
     - `GOOGLE_DRIVE_CLIENT_SECRET`
     - `GOOGLE_DRIVE_REFRESH_TOKEN`
     - `YOUTUBE_CLIENT_ID`
     - `YOUTUBE_CLIENT_SECRET`
     - `YOUTUBE_REFRESH_TOKEN`

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
      
      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '14'

      - name: Install dependencies
        run: |
          npm install

      - name: Run JavaScript script
        run: node main.js
        env:
          GOOGLE_DRIVE_CLIENT_ID: ${{ secrets.GOOGLE_DRIVE_CLIENT_ID }}
          GOOGLE_DRIVE_CLIENT_SECRET: ${{ secrets.GOOGLE_DRIVE_CLIENT_SECRET }}
          GOOGLE_DRIVE_REFRESH_TOKEN: ${{ secrets.GOOGLE_DRIVE_REFRESH_TOKEN }}
          YOUTUBE_CLIENT_ID: ${{ secrets.YOUTUBE_CLIENT_ID }}
          YOUTUBE_CLIENT_SECRET: ${{ secrets.YOUTUBE_CLIENT_SECRET }}
          YOUTUBE_REFRESH_TOKEN: ${{ secrets.YOUTUBE_REFRESH_TOKEN }}
```

---

### JavaScript Script (main.js)

```javascript
const fs = require('fs');
const { google } = require('googleapis');
const { OAuth2 } = require('google-auth-library');
const path = require('path');

// Authenticate to YouTube
async function authenticateYouTube() {
  const oAuth2Client = new OAuth2(
    process.env.YOUTUBE_CLIENT_ID,
    process.env.YOUTUBE_CLIENT_SECRET
  );

  oAuth2Client.setCredentials({ refresh_token: process.env.YOUTUBE_REFRESH_TOKEN });
  const youtube = google.youtube({
    version: 'v3',
    auth: oAuth2Client,
  });
  return youtube;
}

// Authenticate to Google Drive
async function authenticateDrive() {
  const oAuth2Client = new OAuth2(
    process.env.GOOGLE_DRIVE_CLIENT_ID,
    process.env.GOOGLE_DRIVE_CLIENT_SECRET
  );

  oAuth2Client.setCredentials({ refresh_token: process.env.GOOGLE_DRIVE_REFRESH_TOKEN });
  const drive = google.drive({
    version: 'v3',
    auth: oAuth2Client,
  });
  return drive;
}

// Fetch the latest YouTube videos
async function getLatestVideos(youtube, channelId) {
  const response = await youtube.search.list({
    part: 'snippet',
    channelId: channelId,
    order: 'date',
    maxResults: 5,
  });
  return response.data.items;
}

// Download video using YouTube API (use ytdl-core or another downloader)
async function downloadVideo(videoUrl) {
  const downloadPath = '/tmp/downloaded_video.mp4';
  // Placeholder for downloading video logic
  // Use appropriate library like 'ytdl-core' to download video
  return downloadPath;
}

// Upload video to Google Drive
async function uploadToDrive(drive, videoFilePath) {
  const fileMetadata = {
    name: path.basename(videoFilePath),
  };
  const media = {
    body: fs.createReadStream(videoFilePath),
  };

  const file = await drive.files.create({
    resource: fileMetadata,
    media: media,
    fields: 'id',
  });
  return file;
}

// Main function
async function main() {
  const youtube = await authenticateYouTube();
  const drive = await authenticateDrive();
  
  const channelId = 'YOUR_YOUTUBE_CHANNEL_ID'; // Replace with actual channel ID
  
  const videos = await getLatestVideos(youtube, channelId);
  
  for (let video of videos) {
    const videoFilePath = await downloadVideo(video); // Implement video download logic
    await uploadToDrive(drive, videoFilePath);
  }
}

main().catch(console.error);
```

---

### Required Node.js Packages

Create a `package.json` file with the following:

```json
{
  "name": "youtube-to-drive",
  "version": "1.0.0",
  "dependencies": {
    "googleapis": "^92.0.0",
    "google-auth-library": "^8.0.0",
    "ytdl-core": "^4.9.1"
  }
}
```

---

### Key Notes:
- **Environment Variables for Secrets**:
  - Instead of reading a JSON file, environment variables from GitHub Secrets are directly used for `client_id`, `client_secret`, and `refresh_token`.

- **Refresh Token**:
  - Obtain it from the **OAuth Playground** or similar tools by authenticating your client credentials and exchanging the authorization code.

This process avoids handling entire JSON files, ensuring secure credentials management.
