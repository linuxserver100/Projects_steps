To set up email OTP (One-Time Password) verification for logging into an Ubuntu server on AWS, you can use a combination of tools such as **SSH**, **PAM (Pluggable Authentication Modules)**, and an email service (like **SMTP**) to send the OTP. Here’s a step-by-step guide to achieve this:

### Prerequisites
1. An Ubuntu server running on AWS.
2. A domain name or email address configured to send emails.
3. Basic knowledge of SSH and server administration.

### Step 1: Install Required Packages

Connect to your Ubuntu server using SSH, then update your package list and install the necessary packages:

```bash
sudo apt update
sudo apt install libpam-google-authenticator ssmtp
```

### Step 2: Set Up Google Authenticator

1. Each user who will log in will need to configure the Google Authenticator.

```bash
google-authenticator
```

2. Follow the prompts to set up your OTP. This will generate a QR code and a secret key. Make sure to save the backup codes provided.

3. Ensure to answer "yes" when asked if you want to update the `~/.google_authenticator` file.

### Step 3: Configure PAM

1. Open the PAM configuration for SSH:

```bash
sudo nano /etc/pam.d/sshd
```

2. Add the following line at the top of the file:

```plaintext
auth required pam_google_authenticator.so
```

3. Save and exit the editor.

### Step 4: Configure SSH

1. Open the SSH configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```

2. Find the line `ChallengeResponseAuthentication` and set it to `yes`:

```plaintext
ChallengeResponseAuthentication yes
```

3. Also ensure that `UsePAM` is set to `yes`:

```plaintext
UsePAM yes
```

4. Save and exit the editor.

5. Restart the SSH service for changes to take effect:

```bash
sudo systemctl restart sshd
```

### Step 5: Set Up Email Sending for OTP

You'll need to configure `ssmtp` or another mail transfer agent (MTA) to send OTPs via email.

1. Open the `ssmtp` configuration file:

```bash
sudo nano /etc/ssmtp/ssmtp.conf
```

2. Configure the SMTP server settings. Here’s an example configuration for Gmail:

```plaintext
root=your-email@gmail.com
mailhub=smtp.gmail.com:587
AuthUser=your-email@gmail.com
AuthPass=your-email-password
UseSTARTTLS=YES
FromLineOverride=YES
```

Replace `your-email@gmail.com` and `your-email-password` with your actual email and password. For security, consider using an app password for Gmail if you have two-factor authentication enabled.

3. Save and exit the editor.

### Step 6: Modify the OTP Script to Send Email

1. Create a script that will send an OTP via email. For example, create a file called `send_otp.sh`:

```bash
nano /usr/local/bin/send_otp.sh
```

2. Add the following content to the script:

```bash
#!/bin/bash

EMAIL="recipient-email@example.com"
OTP=$(cat /dev/urandom | tr -dc '0-9' | fold -w 6 | head -n 1)

echo "Your OTP is: $OTP" | ssmtp $EMAIL

# Save OTP to a temporary file or environment variable for later verification
echo $OTP > /tmp/otp.txt
```

Replace `recipient-email@example.com` with the actual recipient email address.

3. Make the script executable:

```bash
sudo chmod +x /usr/local/bin/send_otp.sh
```

### Step 7: Configure OTP Verification

You need to modify the OTP verification to check the sent OTP against user input. 

1. Edit the `/etc/pam.d/sshd` file again and append:

```plaintext
auth required pam_exec.so /usr/local/bin/verify_otp.sh
```

2. Create a verification script, e.g., `verify_otp.sh`:

```bash
nano /usr/local/bin/verify_otp.sh
```

3. Add the following code:

```bash
#!/bin/bash

read -p "Enter the OTP sent to your email: " input_otp
saved_otp=$(cat /tmp/otp.txt)

if [ "$input_otp" == "$saved_otp" ]; then
    exit 0
else
    echo "Invalid OTP"
    exit 1
fi
```

4. Make it executable:

```bash
sudo chmod +x /usr/local/bin/verify_otp.sh
```

### Step 8: Testing

1. Disconnect from the SSH session and try to log in again.
2. You should be prompted for your password and then for the OTP, which should be sent to your email.

### Conclusion

With these steps, you can set up email OTP verification for SSH logins on your Ubuntu server. Be sure to test everything thoroughly and consider the security implications of storing and sending OTPs over email. If your server needs to be more secure, consider additional layers of security, such as fail2ban or a VPN.
