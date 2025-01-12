Sure! Here's the same setup with the additional detail of ensuring the GitHub Actions workflow runs on the `main` branch, including the process to get the API credentials for Google Drive:

### Step 1: Setup Your GitHub Repository

1. Create a GitHub repository if you haven't already.
2. In your repository, create a `.github/workflows` directory.
3. Inside `.github/workflows`, create a YAML file (e.g., `download-upload.yml`).

### Step 2: Create `package.json`

In your repository’s root, create a `package.json` file to manage dependencies.

```json
{
  "name": "youtube-to-drive",
  "version": "1.0.0",
  "description": "Automated download and upload YouTube videos to Google Drive",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "axios": "^0.21.1",
    "googleapis": "^39.2.0",
    "fs-extra": "^10.0.0",
    "node-fetch": "^2.6.1",
    "path": "^0.12.7"
  }
}
```

Run `npm install` to install dependencies.

### Step 3: Write the JavaScript Logic

Create an `index.js` file in the root of the repository. This file will contain the logic to download and upload videos.

#### 1. Google Drive Authentication

To upload files to Google Drive, you need to set up Google API credentials.

**Steps to Obtain Google API Credentials:**

- Go to [Google Cloud Console](https://console.cloud.google.com/).
- Create a new project.
- Enable the **Google Drive API**:
  - Navigate to the **API & Services** > **Library**.
  - Search for **Google Drive API** and enable it.
- Create **OAuth 2.0 credentials**:
  - Go to **APIs & Services** > **Credentials**.
  - Click **Create Credentials** > **OAuth client ID**.
  - Select **Web application** and specify a name.
  - Add a **Redirect URI** (for this example, `http://localhost:3000/oauth2callback`).
- Download the `credentials.json` and store it in your repository (but do not upload it to a public repository).

#### 2. JavaScript to Handle the Workflow

```javascript
const fs = require('fs');
const axios = require('axios');
const { google } = require('googleapis');
const path = require('path');
const fetch = require('node-fetch');
const { OAuth2 } = google.auth;

// YouTube Channel URL
const CHANNEL_URL = 'https://www.youtube.com/c/YOUR_CHANNEL_NAME/videos';
const DOWNLOAD_URL = 'https://ssyoutube.com/en25/';

// Google Drive OAuth2 Credentials
const CREDENTIALS_PATH = './credentials.json';
const TOKEN_PATH = './token.json';

// Path to store the downloaded video
const VIDEO_PATH = './downloaded_video.mp4';

// Setup OAuth2 client
const oAuth2Client = new OAuth2();
const drive = google.drive({ version: 'v3', auth: oAuth2Client });

async function authenticateGoogleDrive() {
  const credentials = JSON.parse(fs.readFileSync(CREDENTIALS_PATH));
  const { client_id, client_secret, redirect_uris } = credentials.installed;
  oAuth2Client.setCredentials({
    client_id,
    client_secret,
    redirect_uri: redirect_uris[0],
  });

  // Check for existing token
  if (fs.existsSync(TOKEN_PATH)) {
    const token = JSON.parse(fs.readFileSync(TOKEN_PATH));
    oAuth2Client.setCredentials(token);
  } else {
    // OAuth 2.0 authorization process if token not found
    console.log('No credentials found, starting OAuth process...');
    // Implement OAuth flow here...
  }
}

async function downloadLatestVideo() {
  try {
    // Fetch latest video URL from YouTube Channel (you may need to scrape this or use YouTube API)
    const response = await axios.get(CHANNEL_URL);
    const videoUrl = extractVideoUrl(response.data); // Implement function to extract video URL

    // Now use ssyoutube to download the video
    const videoDownloadUrl = `${DOWNLOAD_URL}${videoUrl}`;
    const videoResponse = await fetch(videoDownloadUrl);
    const videoStream = videoResponse.body.pipe(fs.createWriteStream(VIDEO_PATH));

    videoStream.on('finish', () => {
      console.log('Video downloaded successfully.');
      uploadToGoogleDrive();
    });
  } catch (err) {
    console.error('Error downloading video:', err);
  }
}

async function uploadToGoogleDrive() {
  try {
    const fileMetadata = {
      name: 'latest_video.mp4',
      parents: ['your_drive_folder_id'],
    };
    const media = {
      body: fs.createReadStream(VIDEO_PATH),
    };

    const file = await drive.files.create({
      resource: fileMetadata,
      media: media,
      fields: 'id',
    });

    console.log('Uploaded video to Google Drive with ID:', file.data.id);
  } catch (err) {
    console.error('Error uploading video:', err);
  }
}

async function run() {
  await authenticateGoogleDrive();
  await downloadLatestVideo();
}

// Run every 5 minutes using GitHub Actions
setInterval(run, 5 * 60 * 1000);
```

### Step 4: GitHub Actions Workflow

In the `.github/workflows/download-upload.yml` file, define the GitHub Action that triggers every 5 minutes, but only on the `main` branch:

```yaml
name: Download and Upload YouTube Videos to Google Drive

on:
  push:
    branches:
      - main
  schedule:
    - cron: '*/5 * * * *'  # Runs every 5 minutes

jobs:
  download-upload:
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

    - name: Run the download and upload script
      run: |
        node index.js
```

### Step 5: Deploy the Workflow

1. Push all your files to your GitHub repository.
2. The GitHub Action will automatically run every 5 minutes as specified in the cron job, but only when changes are pushed to the `main` branch.

### Step 6: Ensure No Duplicate Downloads

You can maintain a list of previously downloaded videos (e.g., store video URLs in a `downloaded.json` file) and check against it to ensure no duplicate downloads.

```javascript
const downloadedVideos = JSON.parse(fs.readFileSync('downloaded.json', 'utf8'));
if (downloadedVideos.includes(videoUrl)) {
  console.log('Video already downloaded, skipping.');
  return;
}
```

After a successful download, add the video URL to `downloaded.json` to keep track of downloaded videos.

---

### Conclusion
This approach ensures that every 5 minutes, GitHub Actions fetches the latest video from a YouTube channel, downloads it, and uploads it to Google Drive. You can modify the video extraction logic to meet your needs (e.g., using `ssyoutube.com` or YouTube API).
