Certainly! Below is the entire code rewritten as requested, ensuring that each script aligns with the documentation and corrects any issues from the original code.

---

## 1. JavaScript Script

### `index.js`

```javascript
const { driveAuth } = require('./auth');
const { getLatestVideo, streamVideoToDrive, refreshYouTubeToken } = require('./youtube');
const { uploadToDrive, checkIfVideoExists, refreshDriveToken } = require('./googleDrive');

// Set the channel ID to track (replace with your YouTube channel ID)
const channelId = 'YOUR_CHANNEL_ID';

// Folder on Google Drive where videos will be saved
const driveFolderId = 'YOUR_GOOGLE_DRIVE_FOLDER_ID';

async function downloadAndUpload() {
  try {
    // Refresh YouTube token
    await refreshYouTubeToken();

    // Refresh Google Drive token
    await refreshDriveToken();

    // Authenticate with Google Drive
    const drive = await driveAuth();

    // Get the latest video from the channel
    const video = await getLatestVideo(channelId);

    // Check if the video has already been uploaded
    const videoExists = await checkIfVideoExists(drive, video.id, driveFolderId);
    if (videoExists) {
      console.log('Video already exists in Google Drive. Skipping...');
      return;
    }

    // Stream the video directly to Google Drive
    await streamVideoToDrive(drive, video.url, video.title, driveFolderId);

  } catch (error) {
    console.error('Error:', error.message);
  }
}

// Run the process
downloadAndUpload();
```

---

### `youtube.js`

```javascript
const axios = require('axios');
const ytdl = require('ytdl-core'); // Video download package
const { refreshYouTubeTokenIfNeeded } = require('./refreshYouTubeToken');

// Get the latest video from the YouTube channel
async function getLatestVideo(channelId) {
  try {
    await refreshYouTubeTokenIfNeeded(); // Ensure the token is valid

    const response = await axios.get('https://www.googleapis.com/youtube/v3/search', {
      params: {
        part: 'snippet',
        channelId,
        order: 'date',
        maxResults: 1,
        key: 'YOUR_YOUTUBE_API_KEY', // Replace with your YouTube API key
      },
    });

    const video = response.data.items[0];
    return {
      id: video.id.videoId,
      title: video.snippet.title,
      url: `https://www.youtube.com/watch?v=${video.id.videoId}`,
    };
  } catch (error) {
    throw new Error('Error fetching the latest video: ' + error.message);
  }
}

// Stream the video directly to Google Drive using ytdl-core
async function streamVideoToDrive(drive, videoUrl, videoTitle, folderId) {
  const videoFileName = `${videoTitle.replace(/[^a-zA-Z0-9]/g, '_')}.mp4`;

  try {
    const videoStream = ytdl(videoUrl, { quality: 'highest' });

    const fileMetadata = {
      name: videoFileName,
      parents: [folderId],
    };

    const media = {
      mimeType: 'video/mp4',
      body: videoStream,
    };

    await drive.files.create({
      resource: fileMetadata,
      media: media,
      fields: 'id',
    });

    console.log(`Successfully uploaded: ${videoTitle}`);
  } catch (error) {
    throw new Error('Error streaming video to Google Drive: ' + error.message);
  }
}

module.exports = { getLatestVideo, streamVideoToDrive, refreshYouTubeToken };
```

---

### `googleDrive.js`

```javascript
const { refreshDriveTokenIfNeeded } = require('./refreshDriveToken');

// Check if a video already exists in the specified Google Drive folder
async function checkIfVideoExists(drive, videoId, folderId) {
  try {
    const res = await drive.files.list({
      q: `'${folderId}' in parents and name contains '${videoId}'`,
      fields: 'files(id, name)',
    });
    return res.data.files.length > 0;
  } catch (error) {
    throw new Error('Error checking if video exists on Drive: ' + error.message);
  }
}

// Refresh Google Drive token if needed
async function refreshDriveToken() {
  await refreshDriveTokenIfNeeded();
}

module.exports = { checkIfVideoExists, refreshDriveToken };
```

---

### `auth.js`

```javascript
const { google } = require('googleapis');
const path = require('path');
const fs = require('fs');

const TOKEN_PATH = path.resolve(__dirname, 'token.json');
const CREDENTIALS_PATH = path.resolve(__dirname, 'credentials.json');

// Authenticate with Google Drive
async function driveAuth() {
  const credentials = JSON.parse(fs.readFileSync(CREDENTIALS_PATH, 'utf8'));
  const { client_secret, client_id, redirect_uris } = credentials.installed;
  const auth = new google.auth.OAuth2(client_id, client_secret, redirect_uris[0]);

  const token = JSON.parse(fs.readFileSync(TOKEN_PATH, 'utf8'));
  auth.setCredentials(token);

  return google.drive({ version: 'v3', auth });
}

module.exports = { driveAuth };
```

---

### `refreshYouTubeToken.js`

```javascript
const fs = require('fs');
const path = require('path');
const axios = require('axios');

const TOKEN_PATH = path.resolve(__dirname, 'youtube_token.json');

// Refresh YouTube API Token
async function refreshYouTubeTokenIfNeeded() {
  const tokenData = JSON.parse(fs.readFileSync(TOKEN_PATH, 'utf8'));

  // Check if token is expired
  if (Date.now() > tokenData.expiry_date) {
    try {
      const response = await axios.post(
        'https://oauth2.googleapis.com/token',
        {
          client_id: 'YOUR_CLIENT_ID',
          client_secret: 'YOUR_CLIENT_SECRET',
          refresh_token: tokenData.refresh_token,
          grant_type: 'refresh_token',
        }
      );

      // Save the new token
      const newTokenData = {
        ...tokenData,
        access_token: response.data.access_token,
        expiry_date: Date.now() + response.data.expires_in * 1000,
      };
      fs.writeFileSync(TOKEN_PATH, JSON.stringify(newTokenData, null, 2));
      console.log('YouTube token refreshed successfully!');
    } catch (error) {
      throw new Error('Error refreshing YouTube token: ' + error.message);
    }
  }
}

module.exports = { refreshYouTubeTokenIfNeeded };
```

---

### `refreshDriveToken.js`

```javascript
const fs = require('fs');
const path = require('path');
const axios = require('axios');

const DRIVE_TOKEN_PATH = path.resolve(__dirname, 'drive_token.json');

// Refresh Google Drive API Token
async function refreshDriveTokenIfNeeded() {
  const tokenData = JSON.parse(fs.readFileSync(DRIVE_TOKEN_PATH, 'utf8'));

  // Check if token is expired
  if (Date.now() > tokenData.expiry_date) {
    try {
      const response = await axios.post(
        'https://oauth2.googleapis.com/token',
        {
          client_id: 'YOUR_GOOGLE_CLIENT_ID',
          client_secret: 'YOUR_GOOGLE_CLIENT_SECRET',
          refresh_token: tokenData.refresh_token,
          grant_type: 'refresh_token',
        }
      );

      // Save the new token
      const newTokenData = {
        ...tokenData,
        access_token: response.data.access_token,
        expiry_date: Date.now() + response.data.expires_in * 1000,
      };
      fs.writeFileSync(DRIVE_TOKEN_PATH, JSON.stringify(newTokenData, null, 2));
      console.log('Google Drive token refreshed successfully!');
    } catch (error) {
      throw new Error('Error refreshing Google Drive token: ' + error.message);
    }
  }
}

module.exports = { refreshDriveTokenIfNeeded };
```

---

### Google Drive Token (`drive_token.json`)

```json
{
  "access_token": "YOUR_ACCESS_TOKEN",
  "refresh_token": "YOUR_REFRESH_TOKEN",
  "scope": "https://www.googleapis.com/auth/drive",
  "token_type": "Bearer",
  "expiry_date": 1672531200000
}
```

---

### YouTube Token (`youtube_token.json`)

```json
{
  "access_token": "YOUR_YOUTUBE_ACCESS_TOKEN",
  "refresh_token": "YOUR_YOUTUBE_REFRESH_TOKEN",
  "scope": "https://www.googleapis.com/auth/youtube.readonly",
  "token_type": "Bearer",
  "expiry_date": 1672531200000
}
```

---

### `.github/workflows/download-video.yml`

```yaml
name: Download Latest Video to Google Drive

on:
  schedule:
    - cron: '*/5 * * * *'  # Runs every 5 minutes
  workflow_dispatch:

jobs:
  download_and_upload:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'

      - name: Install dependencies
        run: npm install

      - name: Run script
        run: node index.js
```

---

### Folder Structure

```
.
├── .github
│   └── workflows
│       └── download-video.yml
├── auth.js
├── googleDrive.js
├── youtube.js
├── index.js
├── refreshYouTubeToken.js
├── refreshDriveToken.js
├── credentials.json
├── token.json
├── youtube_token.json
├── drive_token.json
├── package.json
```

---

This revised code ensures that YouTube videos are downloaded **directly to Google Drive**, avoiding the need for local file storage. The video is streamed in real-time from YouTube to Google Drive, making the process more efficient.
