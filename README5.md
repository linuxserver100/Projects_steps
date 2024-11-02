To set up email-based OTP (One-Time Password) login verification on an Ubuntu server running on AWS, you’ll need to configure multi-factor authentication (MFA) with email as a factor. Below is a guide to set up email OTP for SSH login. This setup assumes you have sudo access to the server.

### Prerequisites:
1. **Ubuntu server** on AWS (e.g., EC2 instance) with SSH access.
2. **Email account** to send OTP codes (either using an SMTP server or a mail relay service).
3. **Postfix** for sending email via SMTP (or an alternative mail agent).
4. **OATH Toolkit** or **Google Authenticator PAM module** for OTP generation.

### Steps:

#### Step 1: Update and Install Required Packages
SSH into the server and update the package list.
```bash
sudo apt update && sudo apt upgrade -y
```

Then, install the required packages:
```bash
sudo apt install postfix libsasl2-modules google-authenticator pam-mail -y
```

#### Step 2: Configure Postfix for Email Delivery
If you want to use Postfix to send email, configure it with the SMTP server details.

1. **Configure Postfix** by editing its main configuration file:
   ```bash
   sudo nano /etc/postfix/main.cf
   ```
2. Update the following settings for your SMTP relay (replace with actual SMTP server details):

   ```bash
   relayhost = [smtp.example.com]:587
   smtp_sasl_auth_enable = yes
   smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
   smtp_sasl_security_options = noanonymous
   smtp_sasl_tls_security_options = noanonymous
   smtp_tls_security_level = encrypt
   ```

3. **Save and close the file**.

4. **Create the password file**:
   ```bash
   sudo nano /etc/postfix/sasl_passwd
   ```
   Add your SMTP credentials in this format:
   ```
   [smtp.example.com]:587 username:password
   ```

5. Secure the password file and apply changes:
   ```bash
   sudo chmod 600 /etc/postfix/sasl_passwd
   sudo postmap /etc/postfix/sasl_passwd
   sudo systemctl restart postfix
   ```

#### Step 3: Set Up OTP with Google Authenticator
1. Run `google-authenticator` for the user account that will use OTP:
   ```bash
   google-authenticator
   ```

2. Answer the prompts according to your requirements:
   - Enable time-based OTP.
   - Configure the number of tokens.
   - Record the secret key (or QR code).
   - Set up backup codes.

#### Step 4: Configure PAM for OTP and Email Notifications
1. Open the PAM SSH configuration file:
   ```bash
   sudo nano /etc/pam.d/sshd
   ```

2. Add the following lines to require OTP and send an email notification upon login:
   ```bash
   auth required pam_google_authenticator.so
   auth optional pam_exec.so /usr/local/bin/send-email-otp.sh
   ```

#### Step 5: Create the Email OTP Script
Create a script to send the OTP code to the user’s email. Use the user's configured email or a central account.

1. Create a new script:
   ```bash
   sudo nano /usr/local/bin/send-email-otp.sh
   ```

2. Add the following content to the script:
   ```bash
   #!/bin/bash
   OTP_CODE=$(google-authenticator -q -d -t | tail -n 1)
   echo "Your OTP Code: $OTP_CODE" | mail -s "Your SSH OTP Code" your-email@example.com
   ```

3. Make the script executable:
   ```bash
   sudo chmod +x /usr/local/bin/send-email-otp.sh
   ```

#### Step 6: Enable MFA in SSHD Configuration
1. Open the SSH configuration file:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Make sure the following lines are set:
   ```bash
   ChallengeResponseAuthentication yes
   AuthenticationMethods publickey,keyboard-interactive
   ```

3. Restart SSH to apply changes:
   ```bash
   sudo systemctl restart ssh
   ```

#### Step 7: Test the Configuration
1. Try logging in via SSH.
2. After entering the SSH key, you should receive an email with the OTP.
3. Enter the OTP to complete login.

This setup should allow you to use email OTP as a second factor for SSH login.
