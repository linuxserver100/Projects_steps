
To set up SSH login on an Ubuntu server with email OTP (One-Time Password) verification using Nodemailer, you will need to follow these general steps. This process will involve creating a script to send OTPs via email and modifying the SSH login process to include the OTP verification step.

### Prerequisites

1. **Ubuntu Server**: Make sure you have an Ubuntu server set up with SSH access.
2. **Node.js**: You need to have Node.js installed to run your Nodemailer script. You can install it using:
   ```bash
   sudo apt update
   sudo apt install nodejs npm
   ```
3. **Nodemailer**: You will install Nodemailer via npm.
4. **Email Provider**: You need access to an SMTP server (like Gmail, SendGrid, etc.) to send emails.

### Steps

#### Step 1: Create a Nodemailer Script

1. **Create a directory for your script**:
   ```bash
   mkdir ~/otp-auth
   cd ~/otp-auth
   ```

2. **Initialize a new Node.js project**:
   ```bash
   npm init -y
   ```

3. **Install Nodemailer**:
   ```bash
   npm install nodemailer
   ```

4. **Create a script to send OTP**:
   Create a file named `sendOTP.js`:
   ```javascript
   const nodemailer = require('nodemailer');
   const crypto = require('crypto');

   // Generate a random OTP
   const otp = Math.floor(100000 + Math.random() * 900000).toString(); // 6-digit OTP

   // Set up your SMTP transporter
   const transporter = nodemailer.createTransport({
       service: 'gmail', // For example, using Gmail
       auth: {
           user: 'your-email@gmail.com',
           pass: 'your-email-password'
       }
   });

   // Send OTP email
   const sendOtpEmail = (recipientEmail) => {
       const mailOptions = {
           from: 'your-email@gmail.com',
           to: recipientEmail,
           subject: 'Your OTP Code',
           text: `Your OTP code is ${otp}. It is valid for 5 minutes.`
       };

       transporter.sendMail(mailOptions, (error, info) => {
           if (error) {
               return console.log('Error sending email: ', error);
           }
           console.log('OTP sent: ' + info.response);
       });
   };

   sendOtpEmail(process.argv[2]);
   ```

5. **Store the OTP in a temporary file** (optional):
   You might want to store the OTP to validate it later. You can use a temporary file or a better way would be to implement in-memory storage with expiration.

#### Step 2: Modify SSH Configuration

1. **Backup SSH configuration**:
   ```bash
   sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
   ```

2. **Edit the SSH configuration file**:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
   - Add the following line to enable PAM (Pluggable Authentication Module):
     ```
     UsePAM yes
     ```

3. **Setup PAM**:
   Create a new PAM configuration file:
   ```bash
   sudo nano /etc/pam.d/sshd
   ```
   Add the following lines to call your OTP script:
   ```bash
   auth required pam_exec.so /path/to/your/otp_script.sh
   ```

#### Step 3: Create the OTP Verification Script

1. **Create a new script**:
   Create a script named `otp_script.sh`:
   ```bash
   sudo nano /path/to/your/otp_script.sh
   ```
   Add the following content:
   ```bash
   #!/bin/bash

   read -p "Enter the OTP sent to your email: " user_otp

   # Here you need to compare the user_otp with the generated one
   # For simplicity, we'll assume the OTP is stored in a file
   if [[ "$user_otp" == "$(cat /tmp/otp_file)" ]]; then
       exit 0 # Success
   else
       echo "Invalid OTP."
       exit 1 # Failure
   fi
   ```

   Make sure to store your generated OTP in `/tmp/otp_file` for verification.

2. **Make the script executable**:
   ```bash
   sudo chmod +x /path/to/your/otp_script.sh
   ```

#### Step 4: Run on Reboot

To ensure your script runs on reboot, you can use systemd:

1. **Create a systemd service**:
   Create a new service file:
   ```bash
   sudo nano /etc/systemd/system/otp.service
   ```
   Add the following content:
   ```ini
   [Unit]
   Description=OTP Service

   [Service]
   ExecStart=/usr/bin/node /path/to/your/sendOTP.js your-email@gmail.com
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```

2. **Enable and start the service**:
   ```bash
   sudo systemctl enable otp.service
   sudo systemctl start otp.service
   ```

#### Step 5: Test Your Setup

1. **Restart SSH service**:
   ```bash
   sudo systemctl restart ssh
   ```

2. **Test SSH login**:
   Try to log in via SSH. You should receive an OTP email, and upon entering it, access should be granted.

### Important Security Considerations

- Ensure your email account used for sending OTPs is secured.
- Be cautious of email interception; use SSL/TLS for email sending.
- Consider using secure storage or encryption for OTPs and emails.

### Final Notes

This setup is relatively simple and may not cover all security aspects or production use cases. For production, consider using more robust solutions like 2FA (Two-Factor Authentication) tools or services that handle OTP more securely.

☺️👇😜😜☺️😜👇☺️👇😜☺️👇😜😍👇😜☺️😜😊👇😜🥰😜👇🥰😜👇🥰👇😜😊👇😜🥰👇😜🥰👇😜😜😍👇😉😜☺️😉☺️🙃☺️🙃👇☺️🙃☺️😉☺️👇😜☺️😜👇☺️😜👇☺️😜👇☺️🔗☺️😜👇🔗☺️😜☺️☺️🙃☺️🙃☺️😜☺️👇😜☺️👇😜☺️👇😜☺️👇🙃☺️☺️☺️👇😜☺️😜☺️☺️👇😜☺️😜☺️😜☺️😜☺️👇😜☺️👇😜☺️👇😜☺️😜☺️😜☺️😜☺️👇😜☺️☺️😆😜☺️☺️😅☺️😅☺️😆😜😃☺️😆😜☺️☺️😃☺️😅😅😅😅☺️😅
Setting up SSH login on Ubuntu with email link click verification using Nodemailer requires multiple steps. This setup will involve:

1. **Setting Up SSH**: Installing and configuring the SSH server.
2. **Node.js and Nodemailer**: Setting up a Node.js application that sends verification emails using Nodemailer.
3. **Verification Logic**: Implementing a system to verify login attempts through email links.
4. **Auto-starting the Node.js application**: Configuring it to restart on boot.

Here’s a step-by-step guide:

### 1. Install and Configure SSH

First, make sure you have SSH installed on your Ubuntu server.

```bash
sudo apt update
sudo apt install openssh-server
```

Check the status of the SSH service:

```bash
sudo systemctl status ssh
```

If it’s not running, start it with:

```bash
sudo systemctl start ssh
```

### 2. Install Node.js and Nodemailer

Install Node.js. You can use NodeSource for a more up-to-date version:

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```

Now, create a new directory for your Node.js application:

```bash
mkdir ~/ssh-verification
cd ~/ssh-verification
npm init -y
```

Install Nodemailer:

```bash
npm install nodemailer express body-parser
```

### 3. Create the Node.js Application

Create a file called `app.js` in the `ssh-verification` directory:

```javascript
const express = require('express');
const bodyParser = require('body-parser');
const nodemailer = require('nodemailer');
const crypto = require('crypto');

const app = express();
const port = 3000;

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));

let verificationTokens = {};

// Configure Nodemailer
const transporter = nodemailer.createTransport({
  service: 'gmail', // Use your email provider
  auth: {
    user: 'your-email@gmail.com',
    pass: 'your-email-password', // Consider using an app password for security
  },
});

// Route to send verification email
app.post('/send-verification', (req, res) => {
  const email = req.body.email;
  const token = crypto.randomBytes(20).toString('hex');

  verificationTokens[token] = email; // Store token with email

  const url = `http://localhost:${port}/verify/${token}`;

  const mailOptions = {
    from: 'your-email@gmail.com',
    to: email,
    subject: 'SSH Login Verification',
    text: `Click this link to verify your login: ${url}`,
  };

  transporter.sendMail(mailOptions, (error, info) => {
    if (error) {
      return res.status(500).send('Error sending email');
    }
    res.status(200).send('Verification email sent');
  });
});

// Route to verify the token
app.get('/verify/:token', (req, res) => {
  const token = req.params.token;
  const email = verificationTokens[token];

  if (email) {
    delete verificationTokens[token]; // Remove token after use
    // Proceed with SSH login (You can implement your SSH login logic here)
    res.send('Email verified. You can now log in via SSH.');
  } else {
    res.status(400).send('Invalid or expired token.');
  }
});

app.listen(port, () => {
  console.log(`Server running at http://localhost:${port}`);
});
```

### 4. Configure Your SSH to Allow This Mechanism

You might want to modify the SSH configuration to restrict normal password authentication and use this mechanism instead. Edit your SSH config:

```bash
sudo nano /etc/ssh/sshd_config
```

Change or add the following lines:

```plaintext
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM yes
```

Restart the SSH service:

```bash
sudo systemctl restart ssh
```

### 5. Run the Node.js Application

Run the application:

```bash
node app.js
```

You can test the email sending by sending a POST request to `/send-verification` with a JSON body containing the email address.

### 6. Restart on Boot

To ensure your Node.js application runs on boot, you can use `systemd`. Create a new service file:

```bash
sudo nano /etc/systemd/system/ssh-verification.service
```

Add the following configuration:

```ini
[Unit]
Description=SSH Verification Service
After=network.target

[Service]
ExecStart=/usr/bin/node /home/your-username/ssh-verification/app.js
Restart=always
User=your-username
Environment=PATH=/usr/bin:/usr/local/bin

[Install]
WantedBy=multi-user.target
```

Replace `/home/your-username` with your actual user home directory.

Now, enable and start the service:

```bash
sudo systemctl enable ssh-verification
sudo systemctl start ssh-verification
```

### 7. Additional Considerations

- **Security**: Ensure you use secure passwords, and consider using OAuth for email sending if available.
- **Firewall**: Make sure the firewall allows traffic on the ports used by SSH and your Node.js app.
- **HTTPS**: For production, consider using HTTPS to secure your application, especially when handling login credentials.

### Conclusion

Now you have a basic setup for SSH login verification through email links using Node.js and Nodemailer. This is a fundamental implementation; you might need to enhance the security and add error handling based on your specific needs.

☺️🙂‍↕️🙂‍↕️😊☺️🙂‍↕️😊🥰🥰😛😃🥰😛☺️🥰🥰😛😍🙂‍↕️😊☺️🥭😊☺️😍🥭😊😍😠😃😍🥭😃😍🥭😃😍😍😠😃😍😍😃😛🥰😛😃🥰🙂‍↕️😃🥰😃🥰😃😍🙂‍↕️😃😃😍😍🥭😃☺️🥭😗😍☺️😜🥭🥭😜☺️😍🥭😜😂😘😶‍🌫️😂🤣😜😜😂🤣😜🤣😂😜😂🤣😶‍🌫️🤣😂🤣😶‍🌫️😜😂🤣😜😂🤣😶‍🌫️😂🤣😜😜🤣🤣😶‍🌫️🤣😂😶‍🌫️😂🤣😶‍🌫️🤣😂🤣😶‍🌫️😂🤣😶‍🌫️😂😶‍🌫️😂🤣😍🤣😶‍🌫️😂🤣😜😂😜😂🤣😶‍🌫️😂🔑🤣😜😂🤣🤣😶‍🌫️😍😘🤣😶‍🌫️😍😘🤣😶‍🌫️😍😶‍🌫️🤣🤣😶‍🌫️🤣
Setting up SSH login on Ubuntu that uses your phone number for verification via SMS (using Nodemailer) involves several steps. You can achieve this by implementing two-factor authentication (2FA) where the second factor is sent via SMS. Below, I’ll outline a basic approach for setting this up:

### Prerequisites
1. **Ubuntu Server**: Make sure you have an Ubuntu server (version 20.04 or newer).
2. **Node.js and npm**: Install Node.js and npm if not already installed.
3. **Nodemailer**: We'll use this package for sending emails.
4. **Twilio (or similar service)**: This will allow you to send SMS messages.

### Step 1: Install Required Packages

1. **Update your packages:**
   ```bash
   sudo apt update
   sudo apt upgrade
   ```

2. **Install Node.js and npm:**
   If you don’t have Node.js installed, you can install it using the following commands:
   ```bash
   sudo apt install nodejs npm
   ```

3. **Install Nodemailer:**
   Create a new directory for your project and install Nodemailer:
   ```bash
   mkdir ~/ssh-sms-auth && cd ~/ssh-sms-auth
   npm init -y
   npm install nodemailer
   ```

### Step 2: Set Up Twilio for SMS

1. **Sign Up for Twilio**: Create a Twilio account and get your API credentials (Account SID and Auth Token) and a Twilio phone number.

2. **Create a script to send SMS:**
   Create a new file called `sendSms.js` and add the following code:
   ```javascript
   const nodemailer = require('nodemailer');
   const twilio = require('twilio');

   const accountSid = 'YOUR_TWILIO_ACCOUNT_SID'; // Your Account SID
   const authToken = 'YOUR_TWILIO_AUTH_TOKEN';   // Your Auth Token
   const client = new twilio(accountSid, authToken);

   const sendSms = (to, body) => {
       client.messages.create({
           body: body,
           to: to,  // Text this number
           from: 'YOUR_TWILIO_PHONE_NUMBER' // From a valid Twilio number
       })
       .then(message => console.log(`SMS sent: ${message.sid}`))
       .catch(err => console.error(err));
   };

   module.exports = sendSms;
   ```

### Step 3: Create SSH Authentication Script

1. **Create a new script `ssh-login.js`:**
   ```javascript
   const sendSms = require('./sendSms');
   const readline = require('readline');

   const rl = readline.createInterface({
       input: process.stdin,
       output: process.stdout
   });

   const phoneNumber = 'YOUR_PHONE_NUMBER'; // Your phone number
   const otp = Math.floor(100000 + Math.random() * 900000); // Generate a random OTP

   sendSms(phoneNumber, `Your OTP is: ${otp}`);

   rl.question('Enter the OTP sent to your phone: ', (inputOtp) => {
       if (inputOtp === otp.toString()) {
           console.log('Access Granted!');
           // Place your SSH command here or a method to continue the session
       } else {
           console.log('Access Denied!');
       }
       rl.close();
   });
   ```

### Step 4: Set Up SSH to Use the Script

1. **Modify SSH Configuration:**
   You can configure SSH to call your Node.js script upon login. Open the SSH configuration file:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

   Add the following line at the end:
   ```plaintext
   ForceCommand /path/to/your/ssh-sms-auth/ssh-login.js
   ```

2. **Restart SSH Service:**
   After editing the configuration, restart the SSH service:
   ```bash
   sudo systemctl restart ssh
   ```

### Step 5: Enable Script to Run on Reboot

To ensure your SMS service runs on startup, you can use a systemd service.

1. **Create a systemd service file:**
   ```bash
   sudo nano /etc/systemd/system/sms-ssh.service
   ```

   Add the following content:
   ```plaintext
   [Unit]
   Description=SMS SSH Authentication Service

   [Service]
   ExecStart=/usr/bin/node /path/to/your/ssh-sms-auth/ssh-login.js
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```

2. **Enable the service:**
   ```bash
   sudo systemctl enable sms-ssh.service
   ```

### Step 6: Testing

1. **Connect via SSH**: When you try to connect via SSH, the script will prompt for the OTP sent to your phone.

2. **Debugging**: If you encounter issues, check logs:
   ```bash
   journalctl -u sms-ssh.service
   ```

### Notes
- This is a basic implementation and should be enhanced for security (e.g., handling invalid attempts).
- Make sure your server can send requests to Twilio or your SMS service.
- Depending on your use case, you may want to add additional security layers.

This setup provides a good foundation for SMS-based authentication via SSH. Adjust the scripts as needed to fit your specific use case!





