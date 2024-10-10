
Setting up a video streaming server using Nginx with the RTMP module on Ubuntu 22.04 involves several steps. Below is a detailed guide to help you through the process, including how to stream a video, for example, from YouTube.

### Step 1: Update the System

Start by ensuring your system is up to date.

```bash
sudo apt update
sudo apt upgrade -y
```

### Step 2: Install Required Packages

1. **Install build tools and libraries:**

   ```bash
   sudo apt install -y build-essential libpcre3 libpcre3-dev libssl-dev zlib1g-dev
   ```

2. **Install FFmpeg:**

   FFmpeg will be used for streaming.

   ```bash
   sudo apt install -y ffmpeg
   ```

### Step 3: Install Nginx with the RTMP Module

1. **Download the Nginx source code:**

   You need to download the Nginx source along with the RTMP module.

   ```bash
   cd /usr/local/src
   sudo git clone https://github.com/arut/nginx-rtmp-module.git
   sudo wget http://nginx.org/download/nginx-1.22.0.tar.gz
   ```

   *(Check the Nginx website for the latest version.)*

2. **Extract the Nginx source code:**

   ```bash
   sudo tar -zxvf nginx-1.22.0.tar.gz
   ```

3. **Compile Nginx with the RTMP module:**

   ```bash
   cd nginx-1.22.0
   sudo ./configure --with-http_ssl_module --add-module=../nginx-rtmp-module
   sudo make
   sudo make install
   ```

4. **Start Nginx:**

   ```bash
   sudo /usr/local/nginx/sbin/nginx
   ```

### Step 4: Configure Nginx for RTMP

1. **Open the Nginx configuration file:**

   ```bash
   sudo nano /usr/local/nginx/conf/nginx.conf
   ```

2. **Modify the configuration to include the RTMP block:**

   Add the following configuration at the end of the file, outside of the `http {}` block:

   ```nginx
   rtmp {
       server {
           listen 1935;  # RTMP port
           chunk_size 4096;

           application live {
               live on;
               record off;  # Disable recording
           }
       }
   }
   ```

3. **Save and exit the file.** (Press `CTRL + X`, then `Y`, then `Enter`.)

### Step 5: Restart Nginx

To apply your changes, restart Nginx:

```bash
sudo /usr/local/nginx/sbin/nginx -s reload
```

### Step 6: Test the RTMP Stream

1. **Obtain a YouTube video URL:**

   For streaming a YouTube video, you can use a tool to extract a direct stream link or use a sample video file on your local machine.

   If you want to stream a local video, place it in a directory, e.g., `/home/youruser/video.mp4`.

2. **Use FFmpeg to push the stream to your Nginx server:**

   For a local video file:

   ```bash
   ffmpeg -re -i /home/youruser/video.mp4 -c:v libx264 -preset veryfast -c:a aac -f flv rtmp://localhost/live/stream
   ```

   For a YouTube video, you would typically need a tool like `youtube-dl` to fetch the streamable URL:

   ```bash
   sudo apt install -y youtube-dl
   youtube-dl -f best -o - 'https://www.youtube.com/watch?v=VIDEO_ID' | ffmpeg -re -i pipe:0 -c:v copy -c:a aac -f flv rtmp://localhost/live/stream
   ```

   *(Replace `VIDEO_ID` with the actual ID of the YouTube video you want to stream.)*

### Step 7: View the Stream

1. **Install VLC Media Player:**

   If you don’t have VLC installed, you can install it with:

   ```bash
   sudo apt install -y vlc
   ```

2. **Open VLC and go to `Media > Open Network Stream`.**

3. **Enter the RTMP URL:**

   ```plaintext
   rtmp://your_server_ip/live/stream
   ```

   *(Replace `your_server_ip` with your actual server IP address, or use `localhost` if you're streaming locally.)*

### Conclusion

You have successfully set up an Nginx RTMP server on Ubuntu 22.04. You can now stream videos from local files or YouTube. For more advanced features, consider looking into authentication, recording, or adaptive bitrate streaming.
