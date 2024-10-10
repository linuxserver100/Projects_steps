
Setting up browser-based RDP access on an Ubuntu server using Cloudflared allows you to connect to your server's desktop environment from anywhere via a web browser. This guide will provide detailed steps to accomplish this without using Guacamole or other intermediary solutions.

### Prerequisites

1. **Ubuntu Server**: Ensure you have a working Ubuntu server (20.04 or later recommended).
2. **Cloudflare Account**: You need a Cloudflare account and a registered domain.
3. **Basic knowledge of command line operations**: You should be comfortable with terminal commands.

### Step 1: Install XRDP on Ubuntu

**1. Update the System**

First, ensure your system is up-to-date:

```bash
sudo apt update && sudo apt upgrade -y
```

**2. Install XRDP and a Desktop Environment**

Next, install XRDP along with XFCE, a lightweight desktop environment:

```bash
sudo apt install xrdp xfce4 xfce4-goodies -y
```

**3. Set XFCE as the Default Session**

To ensure XRDP uses XFCE, set it as the default session by creating or modifying the `~/.xsession` file:

```bash
echo "xfce4-session" > ~/.xsession
```

**4. Enable and Start XRDP Service**

Enable and start the XRDP service:

```bash
sudo systemctl enable xrdp
sudo systemctl start xrdp
```

**5. Configure the Firewall**

Allow traffic on the RDP port (3389) using UFW (Uncomplicated Firewall):

```bash
sudo ufw allow 3389
```

### Step 2: Install and Configure Cloudflared

**1. Download and Install Cloudflared**

Download the Cloudflared binary:

```bash
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64
sudo mv cloudflared-linux-amd64 /usr/local/bin/cloudflared
sudo chmod +x /usr/local/bin/cloudflared
```

**2. Authenticate Cloudflared**

Authenticate your Cloudflare account:

```bash
cloudflared tunnel login
```

This command will open a browser window for you to log in to your Cloudflare account and authorize Cloudflared.

**3. Create a New Tunnel**

Create a new tunnel for your RDP connection:

```bash
cloudflared tunnel create <tunnel-name>
```

This command generates a unique tunnel ID and a credentials file. Note the tunnel ID for later use.

**4. Configure the Tunnel**

Edit the Cloudflared configuration file, typically located at `~/.cloudflared/config.yml`. If it doesn't exist, create it:

```bash
nano ~/.cloudflared/config.yml
```

Add the following content:

```yaml
tunnel: <tunnel-id>
credentials-file: /home/<username>/.cloudflared/<tunnel-id>.json

ingress:
  - hostname: <your-subdomain.example.com>
    service: rdp://localhost:3389
  - service: http_status:404
```

Replace:
- `<tunnel-id>` with your tunnel ID.
- `<username>` with your Ubuntu username.
- `<your-subdomain.example.com>` with your actual subdomain.

### Step 3: Run the Cloudflared Tunnel

**1. Start the Tunnel**

Run the tunnel using the command:

```bash
cloudflared tunnel run <tunnel-name>
```

This command establishes the connection and makes your RDP service accessible via the domain specified.

### Step 4: Access RDP from a Browser

To access the RDP session via a web browser, we will use **Apache FreeRDP-WebConnect**, which allows RDP connections through the browser.

**1. Install Required Dependencies**

Install Git and Node.js:

```bash
sudo apt install git nodejs npm -y
```

**2. Download FreeRDP-WebConnect**

Clone the FreeRDP-WebConnect repository:

```bash
git clone https://github.com/FreeRDP/FreeRDP-WebConnect.git
cd FreeRDP-WebConnect
```

**3. Install Required Packages**

Navigate into the directory and install necessary Node.js packages:

```bash
npm install
```

**4. Configure FreeRDP-WebConnect**

Modify the `config.json` file to configure your RDP connection. Open the file with a text editor:

```bash
nano config.json
```

Make sure the RDP parameters are correctly set to point to your server's details (localhost, port 3389).

**5. Start FreeRDP-WebConnect**

Run the FreeRDP-WebConnect server:

```bash
node app.js
```

This will start the web server, typically on port 8080.

### Step 5: Access RDP Through the Browser

1. **Open Your Browser**: Go to `http://<your-subdomain.example.com>:8080`.
   
2. **Login to RDP**: Enter your RDP credentials as prompted.

### Step 6: Finalize Configuration and Security

1. **Secure Cloudflared**: Ensure your Cloudflared tunnel is running securely. Consider setting up a service to run Cloudflared automatically on boot.

2. **DNS Configuration**: Ensure your Cloudflare DNS settings point to your tunnel.

3. **Access Control**: Use Cloudflare's security features to restrict access to your tunnel.

### Conclusion

You have now successfully set up browser-rendered RDP access to your Ubuntu server using Cloudflared. You can access your server's desktop environment through a web browser using your domain name, ensuring a flexible and secure remote access solution. Adjust configurations as needed based on your specific use case and security requirements.





above example with ssl secured👍😀🤩😀💯😍🥹😀😄😔😃😃👍🔑🔑😄👍
Setting up browser-rendered RDP access on an Ubuntu server using Cloudflared with SSL for secure connections involves several steps. This guide will take you through the entire process in detail.

### Prerequisites

1. **Ubuntu Server**: Make sure you have Ubuntu 20.04 or later installed.
2. **Cloudflare Account**: You need a Cloudflare account with a registered domain.
3. **Basic command-line knowledge**: Familiarity with terminal commands is required.

### Step 1: Install XRDP on Ubuntu

**1. Update the System**

Start by updating your system packages:

```bash
sudo apt update && sudo apt upgrade -y
```

**2. Install XRDP and a Desktop Environment**

Install XRDP and a lightweight desktop environment, such as XFCE:

```bash
sudo apt install xrdp xfce4 xfce4-goodies -y
```

**3. Set XFCE as the Default Session**

To make sure XRDP uses XFCE, set it as the default session:

```bash
echo "xfce4-session" > ~/.xsession
```

**4. Enable and Start XRDP**

Enable and start the XRDP service:

```bash
sudo systemctl enable xrdp
sudo systemctl start xrdp
```

**5. Allow RDP Through the Firewall**

Use UFW to allow traffic on the RDP port (3389):

```bash
sudo ufw allow 3389
```

### Step 2: Install and Configure Cloudflared

**1. Download and Install Cloudflared**

Download the Cloudflared binary:

```bash
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64
sudo mv cloudflared-linux-amd64 /usr/local/bin/cloudflared
sudo chmod +x /usr/local/bin/cloudflared
```

**2. Authenticate Cloudflared**

Authenticate your Cloudflare account:

```bash
cloudflared tunnel login
```

This command will open a browser window prompting you to log in to your Cloudflare account.

**3. Create a New Tunnel**

Create a new tunnel for RDP:

```bash
cloudflared tunnel create <tunnel-name>
```

This command generates a unique tunnel ID and credentials file. Take note of these.

**4. Configure the Tunnel**

Edit the Cloudflared configuration file, usually located at `~/.cloudflared/config.yml`. Create it if it doesn't exist:

```bash
nano ~/.cloudflared/config.yml
```

Add the following configuration:

```yaml
tunnel: <tunnel-id>
credentials-file: /home/<username>/.cloudflared/<tunnel-id>.json

ingress:
  - hostname: <your-subdomain.example.com>
    service: rdp://localhost:3389
  - service: http_status:404
```

Replace:
- `<tunnel-id>` with your generated tunnel ID.
- `<username>` with your Ubuntu username.
- `<your-subdomain.example.com>` with your actual subdomain.

### Step 3: Run the Cloudflared Tunnel

**1. Start the Tunnel**

Run the tunnel with:

```bash
cloudflared tunnel run <tunnel-name>
```

This command will start the tunnel and link it to the specified domain.

### Step 4: Secure Your Connection with SSL

By using Cloudflare, your connection will be secured with SSL automatically. Ensure you have SSL enabled for your domain in Cloudflare settings:

1. **Log into Cloudflare**: Go to the Cloudflare dashboard.
2. **Select Your Domain**: Choose the domain you are working with.
3. **SSL/TLS Settings**: Ensure that the SSL option is set to "Full" or "Full (strict)" for optimal security.
4. **Automatic HTTPS Rewrites**: Enable this feature to ensure all HTTP traffic is redirected to HTTPS.

### Step 5: Access RDP from a Browser

**1. Open Your Browser**

Go to your RDP service using the domain name:

```
https://<your-subdomain.example.com>
```

**2. Login to RDP**

You should be prompted to enter your RDP credentials (username and password). 

### Step 6: Final Configuration and Security

1. **Set Up Cloudflared as a Service**: To ensure Cloudflared runs on startup, create a systemd service:

```bash
sudo nano /etc/systemd/system/cloudflared.service
```

Add the following content:

```ini
[Unit]
Description=Cloudflared Tunnel
After=network.target

[Service]
User=<username>
ExecStart=/usr/local/bin/cloudflared tunnel run <tunnel-name>
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Replace `<username>` and `<tunnel-name>` with your details.

2. **Enable and Start the Service**:

```bash
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

3. **DNS Configuration**: Ensure that your DNS records in Cloudflare point to your tunnel.

4. **Access Control**: Use Cloudflare's security features, such as Firewall Rules, to restrict access.

### Conclusion

You have successfully set up browser-rendered RDP access to your Ubuntu server using Cloudflared, with SSL secured via Cloudflare. You can now access your server’s desktop environment securely through a web browser using your domain name. Adjust configurations as needed for enhanced security and performance.
