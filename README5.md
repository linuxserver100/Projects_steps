Setting up email OTP (One-Time Password) login with Google Authenticator for SSH on an Ubuntu server involves several steps, including installing necessary packages, configuring SSH, and ensuring the system sends emails with OTPs. Here’s a comprehensive guide to automate this setup, including running it after a reboot:

### Step 1: Install Required Packages

1. **Open a terminal on your Ubuntu server** and install the required packages:

   ```bash
   sudo apt update
   sudo apt install libpam-google-authenticator
   sudo apt install mailutils
   ```

   - `libpam-google-authenticator` provides the PAM module for OTP.
   - `mailutils` is used for sending emails.

### Step 2: Configure Google Authenticator for Your User

2. **Run the Google Authenticator setup for your user**:

   ```bash
   google-authenticator
   ```

   You will be prompted with several questions:

   - Scan the QR code with the Google Authenticator app on your mobile device.
   - Answer "yes" to the questions to enable rate limiting, emergency scratch codes, and other settings.

### Step 3: Configure PAM for Google Authenticator

3. **Edit the PAM configuration file**:

   ```bash
   sudo nano /etc/pam.d/sshd
   ```

   Add the following line at the top of the file:

   ```plaintext
   auth required pam_google_authenticator.so
   ```

4. **Configure SSH**:

   Edit the SSH configuration file:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

   Ensure the following settings are configured:

   ```plaintext
   ChallengeResponseAuthentication yes
   UsePAM yes
   ```

   You may also want to adjust the `PasswordAuthentication` setting based on your preference.

5. **Restart the SSH service**:

   ```bash
   sudo systemctl restart ssh
   ```

### Step 4: Create a Script for OTP Emailing

6. **Create a script to send OTP via email**:

   Create a script in your home directory (or any preferred directory):

   ```bash
   nano ~/send_otp_email.sh
   ```

   Add the following content to the script:

   ```bash
   #!/bin/bash

   # Read the OTP from the Google Authenticator config
   OTP=$(google-authenticator -l ~/.google_authenticator -q)

   # Email settings
   TO="your_email@example.com"
   SUBJECT="Your OTP Code"
   BODY="Your OTP Code is: $OTP"

   # Send email
   echo "$BODY" | mail -s "$SUBJECT" "$TO"
   ```

   Replace `your_email@example.com` with your actual email address. 

7. **Make the script executable**:

   ```bash
   chmod +x ~/send_otp_email.sh
   ```

### Step 5: Set Up the Script to Run at Startup

8. **Create a systemd service** to run the script at startup:

   ```bash
   sudo nano /etc/systemd/system/send_otp_email.service
   ```

   Add the following content:

   ```ini
   [Unit]
   Description=Send OTP email at startup

   [Service]
   ExecStart=/bin/bash /home/your_username/send_otp_email.sh
   User=your_username
   Type=oneshot

   [Install]
   WantedBy=multi-user.target
   ```

   Make sure to replace `your_username` with your actual username.

9. **Enable the service** to run on boot:

   ```bash
   sudo systemctl enable send_otp_email.service
   ```

### Step 6: Test the Setup

10. **Reboot your server** to test the setup:

    ```bash
    sudo reboot
    ```

After the reboot, check your email for the OTP sent by the script. Now, when you SSH into your server, you should be prompted for the OTP after entering your password.

### Additional Security Measures

- **Secure your SSH access**: Consider using key-based authentication along with OTP for added security.
- **Firewall**: Ensure your firewall settings allow only necessary ports and limit access to SSH.

### Conclusion

You have now set up email OTP login using Google Authenticator for SSH on Ubuntu, with the script running automatically after reboot. Make sure to test the entire flow and adjust any settings as per your security requirements.
