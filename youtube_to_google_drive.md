Here is the rewritten, complete guide and implementation as per your request. The `.yml` file has been updated to run on the `main` branch.

---

## **Comprehensive Guide: Save YouTube Videos to Google Drive Using GitHub Actions**

### **Overview**
This setup automates the following:
1. Fetching the latest video from a YouTube channel.
2. Downloading the video if not already downloaded.
3. Uploading the video to Google Drive.
4. Scheduling the process to run periodically via GitHub Actions.

---

### **Pre-requisites**
1. **YouTube API Key**: Obtain from the Google Developer Console.
2. **Google Drive API**: Configure and generate credentials for the Google Drive API.
3. **Environment Variables**: Store sensitive data like API keys and credentials as GitHub Secrets.

---

## **GitHub Actions Workflow**
Save this YAML configuration in your repository as `.github/workflows/youtube-to-drive.yml`:

```yaml
name: Save YouTube Video to Google Drive

on:
  push:
    branches:
      - main
  schedule:
    - cron: "*/5 * * * *" # Runs every 5 minutes

jobs:
  save_latest_video:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18"

      - name: Install dependencies
        run: npm install

      - name: Save latest video to Google Drive
        env:
          YOUTUBE_API_KEY: ${{ secrets.YOUTUBE_API_KEY }}
          CHANNEL_ID: ${{ secrets.CHANNEL_ID }}
          GOOGLE_DRIVE_CREDENTIALS: ${{ secrets.GOOGLE_DRIVE_CREDENTIALS }}
        run: node saveVideo.js
```

---

## **Node.js Script**
Create a `saveVideo.js` file with this content:

```javascript
const { google } = require("googleapis");
const fs = require("fs");
const axios = require("axios");
const path = require("path");
const ytdl = require("ytdl-core");

// Load environment variables
const YOUTUBE_API_KEY = process.env.YOUTUBE_API_KEY;
const CHANNEL_ID = process.env.CHANNEL_ID;
const GOOGLE_DRIVE_CREDENTIALS = JSON.parse(process.env.GOOGLE_DRIVE_CREDENTIALS);

// File to store downloaded video IDs
const downloadedVideosFile = "downloaded_videos.json";
let downloadedVideos = [];

// Initialize Google Drive API
const auth = new google.auth.GoogleAuth({
  credentials: GOOGLE_DRIVE_CREDENTIALS,
  scopes: ["https://www.googleapis.com/auth/drive.file"],
});
const drive = google.drive({ version: "v3", auth });

// Load previously downloaded video IDs
if (fs.existsSync(downloadedVideosFile)) {
  downloadedVideos = JSON.parse(fs.readFileSync(downloadedVideosFile, "utf-8"));
}

// Function to fetch the latest video from the YouTube channel
async function getLatestVideo() {
  const url = `https://www.googleapis.com/youtube/v3/search?part=snippet&channelId=${CHANNEL_ID}&maxResults=1&order=date&key=${YOUTUBE_API_KEY}`;
  const response = await axios.get(url);
  const video = response.data.items[0];

  return {
    id: video.id.videoId,
    title: video.snippet.title,
  };
}

// Function to download a YouTube video
async function downloadVideo(videoId, title) {
  const videoUrl = `https://www.youtube.com/watch?v=${videoId}`;
  const filePath = path.join(__dirname, `${title}.mp4`);

  const stream = ytdl(videoUrl, { quality: "highest" }).pipe(fs.createWriteStream(filePath));

  return new Promise((resolve, reject) => {
    stream.on("finish", () => resolve(filePath));
    stream.on("error", reject);
  });
}

// Function to upload a video to Google Drive
async function uploadToDrive(filePath, title) {
  const fileMetadata = { name: title };
  const media = { mimeType: "video/mp4", body: fs.createReadStream(filePath) };

  const response = await drive.files.create({
    resource: fileMetadata,
    media: media,
    fields: "id",
  });

  return response.data.id;
}

// Main function to execute the workflow
(async () => {
  try {
    const latestVideo = await getLatestVideo();

    if (!downloadedVideos.includes(latestVideo.id)) {
      console.log("Downloading latest video:", latestVideo.title);
      const filePath = await downloadVideo(latestVideo.id, latestVideo.title);

      console.log("Uploading to Google Drive...");
      const fileId = await uploadToDrive(filePath, latestVideo.title);
      console.log("Uploaded video ID:", fileId);

      // Update the list of downloaded videos
      downloadedVideos.push(latestVideo.id);
      fs.writeFileSync(downloadedVideosFile, JSON.stringify(downloadedVideos, null, 2));

      // Clean up local file
      fs.unlinkSync(filePath);
    } else {
      console.log("Latest video has already been downloaded.");
    }
  } catch (error) {
    console.error("Error:", error);
  }
})();
```

---

## **Dependencies**
Include these dependencies in `package.json`:

```json
{
  "name": "youtube-to-drive",
  "version": "1.0.0",
  "main": "saveVideo.js",
  "scripts": {},
  "dependencies": {
    "axios": "^1.4.0",
    "googleapis": "^122.0.0",
    "ytdl-core": "^4.11.0"
  }
}
```

Install them using:

```bash
npm install
```

---

## **GitHub Secrets**
Add these secrets to your repository:
1. **`YOUTUBE_API_KEY`**: Your YouTube API key.
2. **`CHANNEL_ID`**: The ID of the YouTube channel to monitor.
3. **`GOOGLE_DRIVE_CREDENTIALS`**: Google Drive API credentials in JSON format.

---

## **How It Works**
1. **Workflow Trigger**: The workflow runs on every push to the `main` branch or every 5 minutes as per the cron schedule.
2. **Fetch Latest Video**: The script uses the YouTube API to fetch the most recent video uploaded to the channel.
3. **Avoid Duplicates**: Checks if the video ID already exists in a tracking file (`downloaded_videos.json`).
4. **Download Video**: Downloads the video using `ytdl-core`.
5. **Upload to Google Drive**: Uploads the video file to Google Drive.
6. **Track Progress**: Updates the tracking file with the new video ID to prevent re-downloading.

---

### **Advantages**
- Fully automated and periodic backups.
- Prevents duplicate downloads.
- Easy to extend for additional functionalities.

This implementation ensures a robust and seamless backup workflow for YouTube videos to Google Drive.
