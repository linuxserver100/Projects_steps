Certainly! Below is the corrected and comprehensive version of your scripts, including updates to resolve the issue and ensure that duplicate downloads do not occur. I'll also add the `downloaded.json` file to track downloaded videos and provide a sample `token.json` structure.

---

### **Step 1: Setup Your GitHub Repository**

1. Create a GitHub repository if you haven't already.
2. In your repository, create a `.github/workflows` directory.
3. Inside `.github/workflows`, create a YAML file (e.g., `download-to-drive.yml`).

---

### **Step 2: Create `package.json`**

In your repository’s root, create a `package.json` file to manage dependencies:

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

### **Step 3: Create `index.js`**

```javascript
const fs = require('fs');
const axios = require('axios');
const { google } = require('googleapis');
const fetch = require('node-fetch');
const path = require('path');

const { OAuth2 } = google.auth;

// YouTube Channel URL
const CHANNEL_URL = 'https://www.youtube.com/@webdev-p2e/videos';
const DOWNLOAD_URL = 'https://ssyoutube.com/en25/';

// Paths for credentials, token, and tracking downloads
const CREDENTIALS_PATH = './credentials.json';
const TOKEN_PATH = './token.json';
const DOWNLOADED_JSON = './downloaded.json';

// Initialize Google OAuth2 Client
const oAuth2Client = new OAuth2();
const drive = google.drive({ version: 'v3', auth: oAuth2Client });

// Authenticate with Google Drive
async function authenticateGoogleDrive() {
  try {
    const credentials = JSON.parse(fs.readFileSync(CREDENTIALS_PATH, 'utf8'));
    const { client_id, client_secret, redirect_uris } = credentials.installed || credentials.web;

    oAuth2Client.setCredentials({
      client_id,
      client_secret,
      redirect_uri: redirect_uris[0],
    });

    if (fs.existsSync(TOKEN_PATH)) {
      const token = JSON.parse(fs.readFileSync(TOKEN_PATH, 'utf8'));
      oAuth2Client.setCredentials(token);
    } else {
      throw new Error('Token not found. Complete the OAuth process to generate one.');
    }
  } catch (error) {
    console.error('Error authenticating with Google Drive:', error.message);
    process.exit(1);
  }
}

// Extract video URL from YouTube channel HTML
async function extractVideoUrl(html) {
  const regex = /\/watch\?v=([a-zA-Z0-9_-]+)/;
  const match = html.match(regex);
  if (!match || !match[0]) {
    throw new Error('Failed to extract video URL from channel HTML.');
  }
  return `https://www.youtube.com${match[0]}`;
}

// Download video to Google Drive
async function downloadVideoToGoogleDrive(videoDownloadUrl, folderId, videoName) {
  try {
    const response = await fetch(videoDownloadUrl);
    if (!response.ok) throw new Error(`Failed to fetch video: ${response.statusText}`);

    const media = { body: response.body };
    const fileMetadata = {
      name: `${videoName}.mp4`,
      parents: [folderId],
    };

    const file = await drive.files.create({
      resource: fileMetadata,
      media,
      fields: 'id',
    });

    console.log('Video downloaded to Google Drive with ID:', file.data.id);
  } catch (error) {
    console.error('Error downloading video to Google Drive:', error.message);
    throw error;
  }
}

// Main function
async function run() {
  try {
    await authenticateGoogleDrive();

    const response = await axios.get(CHANNEL_URL);
    const videoUrl = await extractVideoUrl(response.data);

    // Track already downloaded videos
    const downloadedVideos = fs.existsSync(DOWNLOADED_JSON)
      ? JSON.parse(fs.readFileSync(DOWNLOADED_JSON, 'utf8'))
      : [];

    if (downloadedVideos.includes(videoUrl)) {
      console.log('Video already downloaded, skipping...');
      return;
    }

    // Use ssyoutube to generate the download URL
    const videoDownloadUrl = `${DOWNLOAD_URL}${videoUrl}`;
    const folderId = 'YOUR_GOOGLE_DRIVE_FOLDER_ID'; // Replace with your folder ID
    const videoName = new URL(videoUrl).searchParams.get('v');

    await downloadVideoToGoogleDrive(videoDownloadUrl, folderId, videoName);

    // Append to downloaded list
    downloadedVideos.push(videoUrl);
    fs.writeFileSync(DOWNLOADED_JSON, JSON.stringify(downloadedVideos, null, 2));
  } catch (error) {
    console.error('Error:', error.message);
  }
}

// Run every 5 minutes
setInterval(run, 5 * 60 * 1000);
```

---

### **Step 4: `download-to-drive.yml`**

```yaml
name: Download YouTube Videos Directly to Google Drive

on:
  push:
    branches:
      - main
  schedule:
    - cron: '*/5 * * * *'

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
      run: npm install

    - name: Run the download script
      run: node index.js
```

---

### **Step 5: Supporting Files**

#### `credentials.json`
```json
{
  "installed": {
    "client_id": "YOUR_CLIENT_ID",
    "project_id": "YOUR_PROJECT_ID",
    "auth_uri": "https://accounts.google.com/o/oauth2/auth",
    "token_uri": "https://oauth2.googleapis.com/token",
    "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
    "redirect_uris": ["YOUR_REDIRECT_URI"]
  }
}
```

#### `token.json`
```json
{
  "access_token": "YOUR_ACCESS_TOKEN",
  "refresh_token": "YOUR_REFRESH_TOKEN",
  "scope": "https://www.googleapis.com/auth/drive.file",
  "token_type": "Bearer",
  "expiry_date": 1622131234567
}
```

#### `downloaded.json`
```json
[]
```

---

### **Notes**
- Replace placeholders like `YOUR_GOOGLE_DRIVE_FOLDER_ID`, `YOUR_CLIENT_ID`, and other credentials with your actual data.
- Ensure the OAuth process is completed to generate a valid `token.json`.
- The script avoids re-downloading videos by tracking URLs in `downloaded.json`.
