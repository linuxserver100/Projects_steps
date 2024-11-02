Setting up SSH login on Ubuntu with email OTP (One-Time Password) verification can enhance security significantly. Here’s a step-by-step guide on how to configure this without requiring further installations.

### Prerequisites
1. **An Ubuntu Server**: Ensure you have SSH access to your Ubuntu server.
2. **Email Account**: You’ll need an email account to send OTPs. This guide assumes you will use a simple script to send emails via the command line using the `mail` command.

### Step 1: Enable SSH
1. Make sure SSH is installed and running. If it's not installed, you can typically install it using:
   ```bash
   sudo apt update
   sudo apt install openssh-server
   ```
2. Start and enable the SSH service:
   ```bash
   sudo systemctl start ssh
   sudo systemctl enable ssh
   ```

### Step 2: Set Up Email Configuration
1. You need to configure email sending capabilities. You can use `mail` for this. Install it if necessary:
   ```bash
   sudo apt install mailutils
   ```
   During the installation, you will be prompted to configure the email system; choose **"Internet Site"**.

2. Configure `/etc/ssmtp/ssmtp.conf` (assuming `ssmtp` is available):
   ```bash
   sudo nano /etc/ssmtp/ssmtp.conf
   ```
   Add or modify the following lines to reflect your email account:
   ```
   root=your_email@example.com
   mailhub=smtp.example.com:587
   AuthUser=your_email@example.com
   AuthPass=your_password
   UseSTARTTLS=YES
   ```
   Replace `smtp.example.com`, `your_email@example.com`, and `your_password` with your SMTP server's details and your email credentials.

### Step 3: Generate an OTP
1. Create a script to generate and send the OTP:
   ```bash
   sudo nano /usr/local/bin/otp_script.sh
   ```
2. Add the following script content:
   ```bash
   #!/bin/bash

   EMAIL="your_email@example.com"
   OTP=$(shuf -i 100000-999999 -n 1)  # Generate a 6-digit OTP
   echo "Your OTP is: $OTP" | mail -s "Your OTP Code" $EMAIL

   # Store the OTP in a temporary file with a timestamp
   echo "$OTP" > /tmp/otp.txt
   echo "$(date +%s)" >> /tmp/otp.txt
   ```

   Replace `your_email@example.com` with your actual email address.

3. Make the script executable:
   ```bash
   sudo chmod +x /usr/local/bin/otp_script.sh
   ```

### Step 4: Modify SSH Configuration
1. Open the SSH configuration file:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Add the following lines at the end:
   ```bash
   Match User your_username
       ForceCommand /usr/local/bin/otp_script.sh
       PermitTunnel no
       AllowTcpForwarding no
   ```

   Replace `your_username` with your actual username.

3. Ensure you disable password authentication (if desired):
   ```bash
   PasswordAuthentication no
   ```

4. Restart the SSH service to apply the changes:
   ```bash
   sudo systemctl restart ssh
   ```

### Step 5: Create a Verification Script
1. Create another script to verify the OTP:
   ```bash
   sudo nano /usr/local/bin/verify_otp.sh
   ```
2. Add the following content:
   ```bash
   #!/bin/bash

   if [ ! -f /tmp/otp.txt ]; then
       echo "No OTP found. Please request a new one."
       exit 1
   fi

   read -p "Enter your OTP: " USER_OTP
   OTP=$(head -n 1 /tmp/otp.txt)
   TIMESTAMP=$(tail -n 1 /tmp/otp.txt)
   CURRENT_TIME=$(date +%s)

   # Check if OTP is valid for 5 minutes
   if [ "$USER_OTP" == "$OTP" ] && [ $((CURRENT_TIME - TIMESTAMP)) -le 300 ]; then
       echo "OTP is valid. You are logged in."
       rm /tmp/otp.txt  # Remove the OTP file
   else
       echo "Invalid OTP or OTP expired."
       exit 1
   fi
   ```

3. Make this script executable:
   ```bash
   sudo chmod +x /usr/local/bin/verify_otp.sh
   ```

### Step 6: Modify SSH Login to Use OTP Verification
1. Modify your `.bashrc` file to run the OTP verification script after login:
   ```bash
   nano ~/.bashrc
   ```
2. Add the following line at the end:
   ```bash
   /usr/local/bin/verify_otp.sh
   ```

3. Save the file and exit.

### Step 7: Testing
1. Open a new terminal and attempt to SSH into your server:
   ```bash
   ssh your_username@your_server_ip
   ```
2. You should receive an OTP email. Enter the OTP when prompted to verify.

### Notes
- This method requires that your email sending works correctly. Test sending an email using the command:
  ```bash
  echo "Test email" | mail -s "Test" your_email@example.com
  ```
- Make sure to secure your email credentials as necessary.
- The OTP is valid for 5 minutes; adjust the timing in the `verify_otp.sh` script as needed.

### Final Thoughts
This method provides a basic implementation of email-based OTP verification for SSH login on Ubuntu. Depending on your security needs, consider using a more robust solution like a dedicated OTP application (e.g., Google Authenticator) or a dedicated two-factor authentication (2FA) service.
