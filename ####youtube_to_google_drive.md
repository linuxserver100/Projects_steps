Sure, here is the corrected and updated version of the code, ensuring the GitHub Actions workflow runs on the `main` branch of your GitHub repository.

### GitHub Actions Workflow (`.github/workflows/youtube-download.yml`)

```yaml
name: Download Latest YouTube Video to Google Drive

on:
  push:
    branches:
      - main  # Runs on push to the main branch
  schedule:
    - cron: '*/5 * * * *'  # This makes it run every 5 minutes
  workflow_dispatch:  # Allows manual triggering

jobs:
  download-video:
    runs-on: ubuntu-latest  # Runs the job on an Ubuntu-based virtual machine

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2  # This step checks out the code from the repository

    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '16'  # Set up Node.js version 16

    - name: Install dependencies
      run: |
        npm install axios ytdl-core googleapis ffmpeg  # Installs required libraries

    - name: Run download script
      run: |
        node downloadLatestVideo.js  # Runs the script to download the video
      env:
        YOUTUBE_API_KEY: ${{ secrets.YOUTUBE_API_KEY }}  # Your YouTube API key stored securely in GitHub secrets
        GOOGLE_DRIVE_API_KEY: ${{ secrets.GOOGLE_DRIVE_API_KEY }}  # Your Google Drive API key stored securely in GitHub secrets
        GOOGLE_DRIVE_FOLDER_ID: ${{ secrets.GOOGLE_DRIVE_FOLDER_ID }}  # Folder ID in Google Drive where the video will be uploaded
```

### Explanation of GitHub Actions Workflow:

1. **`push`**: This trigger ensures the workflow runs when changes are pushed to the `main` branch. 
2. **`schedule`**: Defines the cron job to run every 5 minutes.
3. **`workflow_dispatch`**: Allows manual triggering of the workflow.
4. **`steps`**: Each step of the job:
   - **Checkout repository**: Checks out the code in the repository.
   - **Set up Node.js**: Installs Node.js version 16.
   - **Install dependencies**: Installs necessary Node.js packages (`axios`, `ytdl-core`, `googleapis`, `ffmpeg`).
   - **Run the script**: Runs the `downloadLatestVideo.js` script to download and upload the latest YouTube video.

### JavaScript Script (`downloadLatestVideo.js`)

```javascript
const axios = require('axios');  // To make HTTP requests
const ytdl = require('ytdl-core');  // To download videos from YouTube
const fs = require('fs');  // For working with files
const path = require('path');  // For handling file paths
const { google } = require('googleapis');  // To interact with Google APIs
const ffmpeg = require('fluent-ffmpeg');  // To merge video and audio files

// Environment variables that will be passed from GitHub secrets
const YOUTUBE_API_KEY = process.env.YOUTUBE_API_KEY;
const GOOGLE_DRIVE_API_KEY = process.env.GOOGLE_DRIVE_API_KEY;
const GOOGLE_DRIVE_FOLDER_ID = process.env.GOOGLE_DRIVE_FOLDER_ID;
const CHANNEL_ID = 'UC_x5XG1OV2P6uZZ5Q8oP3vA';  // Replace with your YouTube channel ID

// 1. Fetch the latest video from the YouTube API
async function fetchLatestVideo() {
  const url = `https://www.googleapis.com/youtube/v3/search?key=${YOUTUBE_API_KEY}&channelId=${CHANNEL_ID}&order=date&part=snippet&type=video&maxResults=1`;
  const response = await axios.get(url);
  const video = response.data.items[0];

  return {
    id: video.id.videoId,
    title: video.snippet.title,
    url: `https://www.youtube.com/watch?v=${video.id.videoId}`,
  };
}

// 2. Check if the video already exists on Google Drive
async function checkVideoExistence(videoId) {
  const drive = google.drive({ version: 'v3', auth: GOOGLE_DRIVE_API_KEY });
  const res = await drive.files.list({
    q: `name = '${videoId}.mp4' and '${GOOGLE_DRIVE_FOLDER_ID}' in parents`,
    spaces: 'drive',
  });

  return res.data.files.length > 0;  // If file exists, return true
}

// 3. Download video and audio streams from YouTube
async function downloadVideo(videoId) {
  const videoStream = ytdl(`https://www.youtube.com/watch?v=${videoId}`, { quality: 'highestvideo' });
  const audioStream = ytdl(`https://www.youtube.com/watch?v=${videoId}`, { quality: 'highestaudio' });

  const videoFilePath = path.join(__dirname, `${videoId}_video.mp4`);
  const audioFilePath = path.join(__dirname, `${videoId}_audio.mp3`);
  const mergedFilePath = path.join(__dirname, `${videoId}.mp4`);

  // Save video and audio streams to local files
  videoStream.pipe(fs.createWriteStream(videoFilePath));
  audioStream.pipe(fs.createWriteStream(audioFilePath));

  return new Promise((resolve, reject) => {
    audioStream.on('end', () => {
      // Merge video and audio into one file using ffmpeg
      ffmpeg()
        .input(videoFilePath)
        .input(audioFilePath)
        .output(mergedFilePath)
        .on('end', () => resolve(mergedFilePath))  // Resolve with the merged file path
        .on('error', reject)  // Reject if there's an error
        .run();
    });
  });
}

// 4. Upload the merged video to Google Drive
async function uploadToGoogleDrive(filePath, videoId) {
  const drive = google.drive({ version: 'v3', auth: GOOGLE_DRIVE_API_KEY });

  const fileMetadata = {
    name: `${videoId}.mp4`,
    parents: [GOOGLE_DRIVE_FOLDER_ID],
  };

  const media = {
    mimeType: 'video/mp4',
    body: fs.createReadStream(filePath),  // Read the video file
  };

  const res = await drive.files.create({
    resource: fileMetadata,
    media: media,
    fields: 'id',
  });

  console.log('Uploaded to Google Drive:', res.data.id);
  fs.unlinkSync(filePath);  // Delete the local file after uploading
}

// Main function to run the workflow
async function main() {
  try {
    // Step 1: Fetch the latest video from YouTube
    const latestVideo = await fetchLatestVideo();
    console.log('Latest Video:', latestVideo);

    // Step 2: Check if video already exists on Google Drive
    const exists = await checkVideoExistence(latestVideo.id);
    if (exists) {
      console.log('Video already exists on Google Drive. Skipping download.');
      return;
    }

    // Step 3: Download the video and audio streams
    const mergedFilePath = await downloadVideo(latestVideo.id);
    console.log('Downloaded and merged video:', mergedFilePath);

    // Step 4: Upload the video to Google Drive
    await uploadToGoogleDrive(mergedFilePath, latestVideo.id);

    // Cleanup the local files
    fs.unlinkSync(mergedFilePath);
    console.log('Process completed successfully');
  } catch (error) {
    console.error('Error occurred:', error);
  }
}

// Run the main function
main();
```

### Explanation of the JavaScript Script:

1. **Fetch Latest Video**:
   - The script uses the YouTube API to get the latest video from the specified channel (`CHANNEL_ID`).
   - It sends a GET request to the YouTube API to fetch video details (video ID, title, etc.).

2. **Check if Video Exists in Google Drive**:
   - The script checks if the video file already exists in your Google Drive folder by searching for the video by its ID. If the video exists, it skips downloading.

3. **Download Video and Audio**:
   - The script downloads both the video and audio streams using `ytdl-core` from YouTube.
   - It saves the video and audio in separate files and then merges them using `ffmpeg`.

4. **Upload to Google Drive**:
   - After merging the video and audio, the script uploads the final file to a specified folder on Google Drive.

### Setting Up GitHub Secrets:

Before running this workflow, you need to set up secrets in GitHub:

1. **YOUTUBE_API_KEY**: Your YouTube API key.
2. **GOOGLE_DRIVE_API_KEY**: Your Google Drive API key (OAuth credentials).
3. **GOOGLE_DRIVE_FOLDER_ID**: The ID of the folder where the video will be uploaded on Google Drive.

### How to Use:

1. **Create a new repository** on GitHub.
2. Add the above files (`youtube-download.yml` in `.github/workflows/` and `downloadLatestVideo.js` in the root of your repository).
3. Set up your GitHub secrets as described.
4. Push the repository to GitHub.
5. The workflow will automatically run when changes are pushed to the `main` branch or every 5 minutes.

This setup will help you automate the process of downloading the latest YouTube video to Google Drive.
