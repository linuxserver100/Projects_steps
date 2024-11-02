To set up SSH login on Ubuntu with email OTP (One-Time Password) verification using SMTP configuration without installing additional packages, you can use the built-in tools of Ubuntu along with a script that sends an OTP via email. Here’s a step-by-step guide to achieve this.

### Prerequisites
1. **Ubuntu Server**: Ensure you have an Ubuntu server set up and SSH installed.
2. **Email Server**: You need access to an SMTP server to send emails.
3. **User Privileges**: You should have root or sudo access.

### Steps to Configure Email OTP for SSH

#### Step 1: Configure SSH
1. **Edit SSH Configuration**:
   Open the SSH configuration file using a text editor.
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. **Disable Password Authentication**:
   Ensure the following lines are set to:
   ```bash
   PasswordAuthentication no
   ChallengeResponseAuthentication yes
   ```

3. **Restart SSH Service**:
   After making changes, restart the SSH service.
   ```bash
   sudo systemctl restart sshd
   ```

#### Step 2: Create a Script for OTP Generation and Email Sending
1. **Create a Script**:
   Create a new script to generate an OTP and send it via email.
   ```bash
   sudo nano /usr/local/bin/ssh-otp.sh
   ```

2. **Add the Following Code**:
   Replace the SMTP settings with your own email credentials and settings.
   ```bash
   #!/bin/bash

   EMAIL="your_email@example.com"      # Change this to your email
   SMTP_SERVER="smtp.example.com"       # Change to your SMTP server
   SMTP_PORT="587"                      # Change if necessary
   SMTP_USER="your_smtp_user"          # SMTP username
   SMTP_PASS="your_smtp_password"      # SMTP password
   OTP=$(shuf -i 100000-999999 -n 1)    # Generate a 6-digit OTP
   echo "Your OTP is: $OTP" | mail -s "Your SSH OTP" -S smtp="$SMTP_SERVER:$SMTP_PORT" -S smtp-auth=login -S smtp-auth-user="$SMTP_USER" -S smtp-auth-password="$SMTP_PASS" -S ssl-verify=ignore "$EMAIL"

   # Store the OTP temporarily
   echo "$OTP" > /tmp/otp.txt
   ```

3. **Make the Script Executable**:
   ```bash
   sudo chmod +x /usr/local/bin/ssh-otp.sh
   ```

#### Step 3: Configure PAM for OTP
1. **Edit the PAM SSH Configuration**:
   Open the PAM SSH configuration file.
   ```bash
   sudo nano /etc/pam.d/sshd
   ```

2. **Add OTP Verification**:
   Add the following line at the top of the file:
   ```bash
   auth required pam_exec.so /usr/local/bin/ssh-otp.sh
   ```

3. **Add a Line for OTP Verification**:
   Add the following line to allow OTP verification:
   ```bash
   auth required pam_unix.so nullok
   ```

#### Step 4: Modify User Authentication to Check OTP
1. **Create a New Script for OTP Verification**:
   Create a script to verify the OTP entered by the user.
   ```bash
   sudo nano /usr/local/bin/otp-verify.sh
   ```

2. **Add the Following Code**:
   ```bash
   #!/bin/bash

   read -p "Enter the OTP sent to your email: " user_otp
   stored_otp=$(cat /tmp/otp.txt)

   if [ "$user_otp" == "$stored_otp" ]; then
       echo "OTP Verified."
       exit 0
   else
       echo "Invalid OTP."
       exit 1
   fi
   ```

3. **Make the Script Executable**:
   ```bash
   sudo chmod +x /usr/local/bin/otp-verify.sh
   ```

#### Step 5: Configure SSH to Use OTP Verification
1. **Modify PAM Again**:
   Open the PAM SSH configuration again.
   ```bash
   sudo nano /etc/pam.d/sshd
   ```

2. **Add the OTP Verification Line**:
   Add the following line after the existing lines:
   ```bash
   auth required pam_exec.so /usr/local/bin/otp-verify.sh
   ```

#### Step 6: Ensure Everything Starts Automatically
1. **Ensure SMTP and Mail Utilities are Installed**:
   If not already installed, install `mailutils` for the `mail` command:
   ```bash
   sudo apt-get install mailutils
   ```

2. **Enable the SSH Service on Boot**:
   This should already be enabled, but check with:
   ```bash
   sudo systemctl enable ssh
   ```

### Step 7: Testing the Setup
1. **Reboot the Server**:
   ```bash
   sudo reboot
   ```

2. **SSH into Your Server**:
   When you SSH into your server, you should receive an OTP via email. After entering your username and password (if applicable), you will be prompted for the OTP.

3. **Enter the OTP**:
   When prompted, enter the OTP sent to your email.

### Important Notes
- Ensure your SMTP server settings are correct and that you can send emails from the command line.
- You might need to allow less secure apps if you're using services like Gmail.
- Always test thoroughly to ensure the setup works as expected.

This configuration provides an additional layer of security to your SSH login process by requiring a valid OTP sent to your email.
