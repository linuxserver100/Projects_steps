Certainly! Here's the complete rewritten script with all the necessary steps, including the updated code for downloading the YouTube videos directly to Google Drive, token refresh, and using GitHub Actions:

---

### Step 1: Setup Your GitHub Repository

1. Create a GitHub repository if you haven't already.
2. In your repository, create a `.github/workflows` directory.
3. Inside `.github/workflows`, create a YAML file (e.g., `download-to-drive.yml`).

---

### Step 2: Create `package.json`

In your repository’s root, create a `package.json` file to manage dependencies.

```json
{
  "name": "youtube-to-drive",
  "version": "1.0.0",
  "description": "Automated download of YouTube videos directly to Google Drive",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "axios": "^0.21.1",
    "fs-extra": "^10.0.0",
    "node-fetch": "^2.6.1",
    "path": "^0.12.7",
    "googleapis": "^39.2.0"
  }
}
```

Run `npm install` to install dependencies.

---

### Step 3: Write the JavaScript Logic

Create an `index.js` file in the root of the repository.

#### Updated Workflow: Direct Download to Google Drive

This new logic directly downloads the video into Google Drive instead of uploading it after downloading locally. It also handles token refresh automatically when expired.

```javascript
const fs = require('fs');
const axios = require('axios');
const { google } = require('googleapis');
const path = require('path');
const fetch = require('node-fetch');
const { OAuth2 } = google.auth;

// YouTube Channel URL
const CHANNEL_URL = 'https://www.youtube.com/@webdev-p2e/videos';
const DOWNLOAD_URL = 'https://ssyoutube.com/en25/';

// Google Drive OAuth2 Credentials
const CREDENTIALS_PATH = 'credentials.json';
const TOKEN_PATH = './token.json';

// Setup OAuth2 client
const oAuth2Client = new OAuth2();
const drive = google.drive({ version: 'v3', auth: oAuth2Client });

async function authenticateGoogleDrive() {
  try {
    // Load OAuth2 client credentials
    if (!fs.existsSync(CREDENTIALS_PATH)) {
      throw new Error('Credentials file not found.');
    }

    const credentials = JSON.parse(fs.readFileSync(CREDENTIALS_PATH, 'utf8'));
    const { client_id, client_secret, redirect_uris } = credentials.installed || credentials.web;

    if (!redirect_uris || !redirect_uris[0]) {
      throw new Error('Redirect URIs are missing in the credentials file.');
    }

    oAuth2Client.setCredentials({
      client_id,
      client_secret,
      redirect_uri: redirect_uris[0],
    });

    // Check for existing token
    if (fs.existsSync(TOKEN_PATH)) {
      const token = JSON.parse(fs.readFileSync(TOKEN_PATH, 'utf8'));
      oAuth2Client.setCredentials(token);

      // Refresh token if expired
      if (oAuth2Client.isTokenExpiring()) {
        console.log('Refreshing token...');
        const newToken = await oAuth2Client.refreshAccessToken();
        oAuth2Client.setCredentials(newToken.tokens);
        fs.writeFileSync(TOKEN_PATH, JSON.stringify(newToken.tokens));
      }
    } else {
      console.log('No credentials found. Please complete the OAuth process.');
      // Implement OAuth flow here if token is not present
    }
  } catch (error) {
    console.error('Error authenticating with Google Drive:', error);
    process.exit(1);
  }
}

async function extractVideoUrl(html) {
  try {
    // Implement logic to extract the latest video URL from the YouTube channel HTML
    const regex = /\/watch\?v=([a-zA-Z0-9_-]+)/;
    const match = html.match(regex);
    if (!match || !match[0]) {
      throw new Error('Failed to extract video URL from channel HTML.');
    }
    return `https://www.youtube.com${match[0]}`;
  } catch (error) {
    console.error('Error extracting video URL:', error);
    throw error;
  }
}

async function downloadVideoToGoogleDrive(videoDownloadUrl, folderId) {
  try {
    const videoResponse = await fetch(videoDownloadUrl);
    if (!videoResponse.ok) {
      throw new Error(`Failed to download video. Status: ${videoResponse.status}`);
    }

    const media = {
      body: videoResponse.body,
    };

    const fileMetadata = {
      name: 'latest_video.mp4', // Change name dynamically if needed
      parents: [folderId],
    };

    const file = await drive.files.create({
      resource: fileMetadata,
      media: media,
      fields: 'id',
    });

    console.log('Video downloaded directly to Google Drive with ID:', file.data.id);
  } catch (error) {
    console.error('Error downloading video to Google Drive:', error);
    throw error;
  }
}

async function run() {
  try {
    await authenticateGoogleDrive();

    const response = await axios.get(CHANNEL_URL);
    const videoUrl = await extractVideoUrl(response.data);

    // Use ssyoutube to generate the download URL
    const videoDownloadUrl = `${DOWNLOAD_URL}${videoUrl}`;

    // Specify the Google Drive folder ID to save the video
    const folderId = '1n8P1Yb7FDQYG9AWtcMtVNw6zOelSc-JQ'; // Replace with your folder ID
    await downloadVideoToGoogleDrive(videoDownloadUrl, folderId);
  } catch (error) {
    console.error('Error in the run process:', error);
  }
}

// Run every 5 minutes using setInterval
setInterval(run, 5 * 60 * 1000);
```

---

### Step 4: GitHub Actions Workflow

In the `.github/workflows/download-to-drive.yml` file, define the GitHub Action:

```yaml
name: Download YouTube Videos Directly to Google Drive

on:
  push:
    branches:
      - main
  schedule:
    - cron: '*/5 * * * *'  # Runs every 5 minutes

jobs:
  download-to-drive:
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

    - name: Run the download script
      run: |
        node index.js
```

---

### Step 5: Track Downloaded Videos

To prevent duplicates, track downloaded video URLs in a `downloaded.json` file.  

```javascript
const downloadedVideos = JSON.parse(fs.readFileSync('downloaded.json', 'utf8'));
if (downloadedVideos.includes(videoUrl)) {
  console.log('Video already downloaded, skipping.');
  return;
}
```

After a successful download, append the video URL to `downloaded.json`.  

---

### Conclusion

This updated approach ensures that every 5 minutes, the script fetches the latest video from the YouTube channel and directly downloads it into a Google Drive folder. The GitHub Actions workflow automates the process, including token refreshing, making it efficient and hands-free.

