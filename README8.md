
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
