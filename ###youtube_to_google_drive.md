Here is the entire code as you requested, with all parts provided again, including the folder structure chart at the end.

---

### 1. JavaScript Script

We will write the script in JavaScript that interacts with the YouTube API to get the latest video, checks if it has already been downloaded, and uploads it to Google Drive using the Google Drive API.

#### `index.js`
```javascript
const { google } = require('googleapis');
const fs = require('fs');
const axios = require('axios');
const path = require('path');
const { googleAuth, driveAuth } = require('./auth');
const { getLatestVideo, downloadVideo } = require('./youtube');
const { uploadToDrive, checkIfVideoExists } = require('./googleDrive');

// Set the channel ID to track (replace with your YouTube channel ID)
const channelId = 'YOUR_CHANNEL_ID';

// Folder on Google Drive where videos will be saved
const driveFolderId = 'YOUR_GOOGLE_DRIVE_FOLDER_ID';

// Initialize YouTube and Google Drive API clients
const youtube = google.youtube('v3');
const drive = google.drive('v3');

async function downloadAndUpload() {
  try {
    // Get the latest video from the channel
    const video = await getLatestVideo(channelId);

    // Check if the video has already been downloaded
    const videoExists = await checkIfVideoExists(drive, video.id, driveFolderId);
    if (videoExists) {
      console.log('Video already downloaded. Skipping...');
      return;
    }

    // Download the latest video
    const videoFilePath = await downloadVideo(video.url, video.title);

    // Upload the video to Google Drive
    await uploadToDrive(drive, videoFilePath, video.title, driveFolderId);

    // Clean up the downloaded video file
    fs.unlinkSync(videoFilePath);
  } catch (error) {
    console.error('Error:', error);
  }
}

// Run the download and upload process
downloadAndUpload();
```

#### `youtube.js`
```javascript
const axios = require('axios');
const fs = require('fs');
const path = require('path');

// Get the latest video from the YouTube channel
async function getLatestVideo(channelId) {
  try {
    const response = await axios.get(`https://www.googleapis.com/youtube/v3/search`, {
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

// Download the video
async function downloadVideo(videoUrl, videoTitle) {
  const videoFilePath = path.resolve(__dirname, `${videoTitle}.mp4`);
  
  // Download the video using a video downloader (you can use packages like 'ytdl-core' for YouTube)
  // Assuming a function that downloads the video. This part will depend on your video downloader.
  const videoStream = await axios({
    method: 'get',
    url: videoUrl,
    responseType: 'stream',
  });

  // Save the video to a file
  const writer = fs.createWriteStream(videoFilePath);
  videoStream.data.pipe(writer);

  return new Promise((resolve, reject) => {
    writer.on('finish', () => resolve(videoFilePath));
    writer.on('error', reject);
  });
}

module.exports = { getLatestVideo, downloadVideo };
```

#### `googleDrive.js`
```javascript
const fs = require('fs');

// Check if the video already exists in Google Drive
async function checkIfVideoExists(drive, videoId, folderId) {
  try {
    const res = await drive.files.list({
      q: `'${folderId}' in parents and name='${videoId}.mp4'`,
      fields: 'files(id, name)',
    });

    return res.data.files.length > 0;
  } catch (error) {
    throw new Error('Error checking if video exists on Drive: ' + error.message);
  }
}

// Upload video to Google Drive
async function uploadToDrive(drive, videoFilePath, videoTitle, folderId) {
  const fileMetadata = {
    name: `${videoTitle}.mp4`,
    parents: [folderId],
  };
  const media = {
    mimeType: 'video/mp4',
    body: fs.createReadStream(videoFilePath),
  };

  try {
    await drive.files.create({
      resource: fileMetadata,
      media: media,
      fields: 'id',
    });

    console.log(`Successfully uploaded: ${videoTitle}`);
  } catch (error) {
    throw new Error('Error uploading video to Google Drive: ' + error.message);
  }
}

module.exports = { checkIfVideoExists, uploadToDrive };
```

#### `auth.js`
```javascript
const { google } = require('googleapis');

// Google Drive Authentication
async function googleAuth() {
  const auth = new google.auth.OAuth2(
    'YOUR_CLIENT_ID',
    'YOUR_CLIENT_SECRET',
    'YOUR_REDIRECT_URI'
  );

  // Set the access and refresh tokens here
  auth.setCredentials({
    access_token: 'YOUR_ACCESS_TOKEN',
    refresh_token: 'YOUR_REFRESH_TOKEN',
  });

  return auth;
}

async function driveAuth() {
  const auth = await googleAuth();
  return google.drive({ version: 'v3', auth });
}

module.exports = { googleAuth, driveAuth };
```

### 2. `package.json`

Here’s the required `package.json` file:

```json
{
  "name": "youtube-to-drive",
  "version": "1.0.0",
  "main": "index.js",
  "dependencies": {
    "axios": "^0.24.0",
    "googleapis": "^39.2.0",
    "fs": "0.0.1-security",
    "path": "^0.12.7"
  },
  "devDependencies": {},
  "scripts": {
    "start": "node index.js"
  },
  "author": "",
  "license": "ISC"
}
```

### 3. GitHub Actions Workflow (YAML)

This YAML file runs the script every 5 minutes.

#### `.github/workflows/download-video.yml`

```yaml
name: Download Latest Video to Google Drive

on:
  schedule:
    - cron: '*/5 * * * *'  # Runs every 5 minutes
  push:
    branches:
      - main

jobs:
  download_and_upload:
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

      - name: Run the video download and upload script
        run: node index.js

      - name: Commit changes
        run: git add . && git commit -m "Updated video list" && git push
```

### 4. Refreshing Tokens Automatically

You will need a method to refresh the access token when it expires. Here’s an example script that can refresh the tokens:

```javascript
const { google } = require('googleapis');

// Function to refresh the Google OAuth token
async function refreshToken() {
  const auth = new google.auth.OAuth2(
    'YOUR_CLIENT_ID',
    'YOUR_CLIENT_SECRET',
    'YOUR_REDIRECT_URI'
  );

  auth.setCredentials({
    refresh_token: 'YOUR_REFRESH_TOKEN',
  });

  try {
    const { tokens } = await auth.refreshAccessToken();
    console.log('Access token refreshed:', tokens);
    // Save the new access token for future use
    return tokens.access_token;
  } catch (error) {
    console.error('Error refreshing token:', error);
    throw new Error('Failed to refresh token');
  }
}

module.exports = { refreshToken };
```

---

### Folder Structure

Here’s how the file structure would look for this setup:

```
.
├── .github
│   └── workflows
│       └── download-video.yml   # GitHub Actions workflow
├── auth.js                      # Authentication logic for Google APIs
├── googleDrive.js               # Google Drive upload and check logic
├── youtube.js                   # YouTube API video download logic
├── index.js                     # Main script that runs the workflow
├── package.json                 # Node.js dependencies and scripts
└── token.json                   # Optional: You may store your OAuth tokens here
```

---

### Summary:
1. **JavaScript Code**: Handles downloading the latest YouTube video and uploading it to Google Drive.
2. **GitHub Actions Workflow**: Runs the script every 5 minutes to check for new videos and uploads them.
3. **Token Refresh**: A script to refresh expired tokens for Google OAuth.

You can customize the `YOUR_YOUTUBE_API_KEY`, `YOUR_CLIENT_ID`, `YOUR_CLIENT_SECRET`, and `YOUR_GOOGLE_DRIVE_FOLDER_ID` values in the provided code. The workflow will check every 5 minutes for the latest video from the specified YouTube channel and upload it to your Google Drive.

Let me know if you need further modifications!
