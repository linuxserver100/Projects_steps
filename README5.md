
To set up email-based authentication for SSH on an Ubuntu server, you can create a system that sends a one-time password (OTP) to users via email when they attempt to log in. This method typically involves a combination of PAM (Pluggable Authentication Modules) and a mail server. Below are the steps to achieve this:

### Prerequisites

1. **An Ubuntu Server**: Ensure you have administrative access to the server.
2. **Mail Transfer Agent (MTA)**: Install a mail server like `Postfix` or `Sendmail` to send emails.
3. **Python**: Install Python and necessary libraries to generate and send the OTP.

### Step 1: Install Postfix

1. **Install Postfix**:
   ```bash
   sudo apt update
   sudo apt install postfix mailutils
   ```

2. **Configure Postfix**:
   - During installation, you'll be prompted to configure Postfix. Choose "Internet Site" and set the system mail name to your domain name or server IP.

### Step 2: Install Required Packages

You need Python and some libraries for OTP generation and sending emails.

1. **Install Python and Required Libraries**:
   ```bash
   sudo apt install python3 python3-pip
   pip3 install pyotp
   ```

### Step 3: Create the OTP Script

Create a Python script to generate an OTP and send it via email.

1. **Create a Directory for the Script**:
   ```bash
   mkdir ~/otp_auth
   cd ~/otp_auth
   ```

2. **Create the OTP Script**:
   ```bash
   nano send_otp.py
   ```

3. **Add the Following Code**:
   ```python
   import os
   import sys
   import smtplib
   import pyotp
   from email.mime.text import MIMEText

   # Configuration
   EMAIL_FROM = 'your-email@example.com'  # Change to your email
   EMAIL_PASSWORD = 'your-email-password'  # Change to your email password
   SMTP_SERVER = 'smtp.example.com'  # Change to your SMTP server
   SMTP_PORT = 587  # Port number (587 for TLS)

   # Generate OTP
   totp = pyotp.TOTP('base32secret3232')  # Change this secret for each user
   otp = totp.now()

   # Send OTP via Email
   def send_email(to_email, otp):
       msg = MIMEText(f'Your OTP is: {otp}')
       msg['Subject'] = 'Your OTP Code'
       msg['From'] = EMAIL_FROM
       msg['To'] = to_email

       with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
           server.starttls()
           server.login(EMAIL_FROM, EMAIL_PASSWORD)
           server.sendmail(EMAIL_FROM, to_email, msg.as_string())

   if __name__ == "__main__":
       if len(sys.argv) != 2:
           print("Usage: python send_otp.py <user_email>")
           sys.exit(1)

       user_email = sys.argv[1]
       send_email(user_email, otp)
       print(f"OTP sent to {user_email}")
   ```

4. **Save and Exit** (`Ctrl + X`, then `Y`, then `Enter`).

### Step 4: Modify SSH Configuration

1. **Edit the SSH Configuration**:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. **Add or Modify the Following Lines**:
   ```bash
   ChallengeResponseAuthentication yes
   ```

3. **Save and Exit**.

4. **Restart SSH**:
   ```bash
   sudo systemctl restart ssh
   ```

### Step 5: Configure PAM

1. **Edit the PAM Configuration for SSH**:
   ```bash
   sudo nano /etc/pam.d/sshd
   ```

2. **Add the Following Line at the Top**:
   ```bash
   auth required pam_exec.so /usr/bin/python3 /home/your-username/otp_auth/send_otp.py %u
   ```

   Replace `/home/your-username/` with the actual path to the script.

3. **Save and Exit**.

### Step 6: Test the Configuration

1. **Log in via SSH** from a different terminal or machine.
2. After entering your username and password, the OTP should be sent to the configured email.
3. Check your email for the OTP and enter it in the SSH prompt to gain access.

### Notes

- Ensure that your firewall allows outbound SMTP traffic if you're using a cloud provider.
- You may want to handle rate limiting and more advanced security measures to protect against brute force attacks.
- Make sure that the email address configured in the script can be sent emails from the server.
- For production environments, consider using a more robust solution like 2FA libraries or dedicated authentication services. 

### Troubleshooting

- If you encounter issues with email delivery, check your mail logs (usually in `/var/log/mail.log`) for errors.
- Verify that your email settings are correct, especially SMTP server, port, and authentication details.
