To create a GitHub Actions workflow that automatically downloads the latest YouTube video from a channel to Google Drive, we can break the task into multiple steps. I will explain each part in simple terms and walk through the necessary steps for a beginner.

### Steps:

1. **Setting up GitHub Actions**: We will create a GitHub Actions workflow that runs every 5 minutes to check for the latest YouTube video.
2. **Fetching the latest YouTube video**: Using the YouTube API to get the latest video from a YouTube channel.
3. **Downloading the video and audio**: We will download the video and audio from YouTube.
4. **Merging video and audio**: Sometimes YouTube videos are split into video and audio streams, so we need to combine them.
5. **Uploading to Google Drive**: Finally, we upload the merged video to Google Drive.

### GitHub Actions Workflow (`.github/workflows/youtube-download.yml`)

```yaml
name: Download Latest YouTube Video to Google Drive

on:
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

1. **`schedule`**: This defines how often the workflow should run. `*/5 * * * *` means it will run every 5 minutes.
2. **`workflow_dispatch`**: This lets you manually trigger the workflow in GitHub.
3. **`steps`**: Each step defines a task that the workflow will do.
   - **Checkout repository**: This step checks out the code in the repository.
   - **Set up Node.js**: Installs Node.js to run the script.
   - **Install dependencies**: Installs necessary Node.js packages (`axios`, `ytdl-core`, `googleapis`, and `ffmpeg`).
   - **Run the script**: Runs the `downloadLatestVideo.js` script that performs the video download and upload tasks.

### JavaScript Script (`downloadLatestVideo.js`)

This script does the actual work of downloading and uploading the video.

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
5. The workflow will automatically run every 5 minutes or can be triggered manually.

This setup will help you automate the process of downloading the latest YouTube video to Google Drive.
