To set up SSH login on Ubuntu using an email-based one-time password (OTP), you can follow these general steps:

### Prerequisites

1. **Ubuntu Server**: You need access to an Ubuntu server with SSH installed.
2. **Mail Transfer Agent (MTA)**: You will need an MTA like `postfix` or `sendmail` to send emails from your server.
3. **Python**: A script to generate and send the OTP.
4. **SSH Access**: Ensure you have SSH access to your server.

### Step 1: Install Required Packages

Make sure you have `postfix` and `python3` installed on your server:

```bash
sudo apt update
sudo apt install postfix mailutils python3
```

During the `postfix` installation, you will be prompted to select a configuration type. For a simple setup, you can choose "Internet Site" and set the system mail name (usually your domain name).

### Step 2: Create the OTP Generation Script

Create a Python script that generates a random OTP and sends it via email:

```bash
sudo nano /usr/local/bin/send_otp.py
```

Add the following code to the script:

```python
import smtplib
import random
import string
import sys

# Generate a random OTP
otp = ''.join(random.choices(string.digits, k=6))

# Email details
sender_email = "youremail@example.com"
receiver_email = sys.argv[1]  # The email address passed as an argument
password = "your_email_password"  # Your email password or app password

# Create the email
subject = "Your OTP Code"
body = f"Your OTP code is: {otp}"

message = f"Subject: {subject}\n\n{body}"

# Send the email
with smtplib.SMTP('smtp.example.com', 587) as server:
    server.starttls()
    server.login(sender_email, password)
    server.sendmail(sender_email, receiver_email, message)

print(otp)  # Print OTP for verification
```

Replace `youremail@example.com`, `your_email_password`, and `smtp.example.com` with your actual email address, email password, and SMTP server.

### Step 3: Set Up SSH Configuration

Edit the SSH configuration file to allow OTP login:

```bash
sudo nano /etc/ssh/sshd_config
```

Add or modify the following line:

```plaintext
ChallengeResponseAuthentication yes
```

### Step 4: Create a PAM Configuration for OTP

Create a new PAM configuration file:

```bash
sudo nano /etc/pam.d/sshd
```

Add the following lines:

```plaintext
auth required pam_exec.so /usr/local/bin/send_otp.py
auth required pam_unix.so nullok
```

### Step 5: Restart SSH Service

Restart the SSH service to apply changes:

```bash
sudo systemctl restart ssh
```

### Step 6: Test the Configuration

1. Attempt to SSH into your server:

   ```bash
   ssh username@your-server-ip
   ```

2. When prompted, provide your email address. The server will send an OTP to that address.
3. Check your email for the OTP, enter it when prompted, and you should be logged in.

### Additional Security Notes

- **Security**: Ensure that your email account is secured with two-factor authentication (2FA).
- **App Passwords**: If using Gmail or similar services, consider using an app-specific password instead of your main password.
- **Firewall**: Ensure your firewall allows SSH traffic.

### Conclusion

This setup allows you to authenticate via SSH using an email OTP. Make sure to test the configuration and adjust as necessary for your specific use case and security requirements.
