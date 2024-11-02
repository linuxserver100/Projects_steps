Setting up email OTP (One-Time Password) login with Google Authenticator for SSH on Ubuntu involves several steps. Below is a comprehensive guide to help you configure this system, including making it run automatically after a reboot.

### Prerequisites

1. **Ubuntu Server**: Make sure you have SSH access to your server.
2. **Root/Sudo Access**: Ensure you have administrative privileges.
3. **Email Account**: Have access to an email service to send OTPs.

### Step 1: Install Required Packages

First, update your package list and install `libpam-google-authenticator` for OTP functionality.

```bash
sudo apt update
sudo apt install libpam-google-authenticator mailutils
```

### Step 2: Configure Google Authenticator

1. Run the Google Authenticator setup:

   ```bash
   google-authenticator
   ```

   - Follow the prompts to generate a QR code and a secret key. You can scan the QR code with the Google Authenticator app on your phone. 
   - Make sure to answer the questions, especially about updating the `.google_authenticator` file.

2. This will create a `.google_authenticator` file in your home directory, which contains the configuration.

### Step 3: Configure SSH

1. Edit the SSH configuration file:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Find and modify the following lines (if they exist) or add them if not:

   ```plaintext
   ChallengeResponseAuthentication yes
   UsePAM yes
   ```

3. Save the file and exit the editor (Ctrl + X, then Y, and Enter).

### Step 4: Configure PAM for Google Authenticator

1. Edit the PAM configuration file for SSH:

   ```bash
   sudo nano /etc/pam.d/sshd
   ```

2. Add the following line at the top:

   ```plaintext
   auth required pam_google_authenticator.so
   ```

3. Save the file and exit.

### Step 5: Configure Email Sending for OTP

You will need a script to send the OTP via email. Here’s a simple example using `mail` command.

1. Create a script file:

   ```bash
   sudo nano /usr/local/bin/send_otp.sh
   ```

2. Add the following content, replacing `your_email@example.com` with your actual email address:

   ```bash
   #!/bin/bash

   OTP=$(cat ~/.google_authenticator | grep -A1 "^$USER" | tail -n1 | awk '{print $2}')

   echo "Your OTP is: $OTP" | mail -s "Your OTP Code" your_email@example.com
   ```

3. Make the script executable:

   ```bash
   sudo chmod +x /usr/local/bin/send_otp.sh
   ```

### Step 6: Modify the SSH Login Process

To ensure that the OTP is sent every time you attempt to log in via SSH, you can modify the `.bashrc` or the `.profile` file.

1. Edit your `.bashrc` or `.profile` file:

   ```bash
   nano ~/.bashrc
   ```

2. Add the following line at the end of the file:

   ```bash
   /usr/local/bin/send_otp.sh
   ```

3. Save the file and exit.

### Step 7: Restart SSH Service

Restart the SSH service to apply the changes:

```bash
sudo systemctl restart sshd
```

### Step 8: Test the Configuration

1. Open a new SSH session.
2. You should receive an email with the OTP each time you attempt to log in.

### Step 9: Automate the Script After Reboot

If you want to ensure the OTP script runs automatically after reboot (in case of server restarts), consider using a `cron` job.

1. Open the crontab:

   ```bash
   crontab -e
   ```

2. Add the following line to run the script at reboot:

   ```bash
   @reboot /usr/local/bin/send_otp.sh
   ```

3. Save and exit.

### Security Considerations

- Make sure that your email credentials are secure.
- Consider using a dedicated email account for sending OTPs to limit exposure.
- Regularly rotate your Google Authenticator secret keys.

### Conclusion

Now, you should have a functioning setup that allows you to log into your Ubuntu server using an OTP sent to your email, which will be triggered automatically on server restart. Always test the configuration thoroughly before relying on it for critical access.
