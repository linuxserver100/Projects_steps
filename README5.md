
Setting up SSH login on Ubuntu with email-based One-Time Password (OTP) verification without any further installations involves utilizing existing tools and services. Here, I'll guide you through the process step-by-step:

### Prerequisites

1. **A running Ubuntu server** (with SSH access).
2. **An email account** to send OTPs.
3. **SSH access** to the server.

### Step 1: Configure SSH

1. **Access your server** via SSH:

   ```bash
   ssh username@your_server_ip
   ```

2. **Edit the SSH configuration file**:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

3. **Modify the following lines** in the configuration file:

   - Ensure the following settings are configured:

     ```bash
     PasswordAuthentication no
     ChallengeResponseAuthentication yes
     UsePAM yes
     ```

4. **Save and exit** the file (`CTRL + X`, then `Y`, then `Enter`).

5. **Restart the SSH service** to apply changes:

   ```bash
   sudo systemctl restart ssh
   ```

### Step 2: Set Up PAM for OTP

1. **Edit the PAM configuration file** for SSH:

   ```bash
   sudo nano /etc/pam.d/sshd
   ```

2. **Add the following line** at the top of the file (after the initial comments):

   ```bash
   auth required pam_exec.so /usr/local/bin/otp-email.sh
   ```

   This line will execute a custom script that we'll create in the next step.

3. **Save and exit** the file.

### Step 3: Create the OTP Script

1. **Create the directory for the script**:

   ```bash
   sudo mkdir -p /usr/local/bin
   ```

2. **Create the OTP script**:

   ```bash
   sudo nano /usr/local/bin/otp-email.sh
   ```

3. **Add the following script** to the file. This script generates an OTP and sends it via email:

   ```bash
   #!/bin/bash

   # Email configuration
   EMAIL="your_email@example.com"
   SUBJECT="Your OTP Code"
   OTP=$(date +%s | sha256sum | base64 | head -c 6; echo)

   # Send the email
   echo "Your OTP code is: $OTP" | mail -s "$SUBJECT" $EMAIL

   # Prompt for the OTP
   echo "Enter the OTP sent to your email:"
   read USER_OTP

   # Validate the OTP (this is a very basic implementation)
   if [ "$USER_OTP" == "$OTP" ]; then
       exit 0
   else
       echo "Invalid OTP"
       exit 1
   fi
   ```

   **Note:** Replace `your_email@example.com` with the email address from which you want to send the OTP.

4. **Save and exit** the file.

5. **Make the script executable**:

   ```bash
   sudo chmod +x /usr/local/bin/otp-email.sh
   ```

### Step 4: Install Mail Utilities

Since you requested no further installations, if `mail` is not already available on your Ubuntu server, it is usually included in packages like `mailutils` or `postfix`. However, the email-sending functionality might depend on the setup of your SMTP server.

If `mail` is not installed and you want to use it, you can install it using:

```bash
sudo apt update
sudo apt install mailutils
```

### Step 5: Testing the Setup

1. **Log out of the server**:

   ```bash
   exit
   ```

2. **Attempt to log in again**:

   ```bash
   ssh username@your_server_ip
   ```

3. **You should receive an OTP email. Enter the OTP** to log in.

### Troubleshooting

- Ensure that your server can send emails (check firewall settings or SMTP configurations if necessary).
- Verify that the email address is correctly configured in the script.
- If using a custom SMTP service, you might need to adjust the email-sending mechanism.

### Security Considerations

- This implementation is basic and can be improved. Consider using secure methods for sending emails and storing OTPs.
- You might want to log failed attempts for security audits.

With this setup, you will have SSH login secured by email-based OTP verification without any additional installations.
