To set up SSH login on an Ubuntu server with email-based OTP (One-Time Password) authentication, you can use the following guide. This involves configuring `PAM` (Pluggable Authentication Modules) and integrating an email-based OTP system.

## Prerequisites

1. **SMTP setup** on your server to send emails.
2. SSH access to the server with `sudo` privileges.
3. **Python** and **ssmtp** (or another email client) to send the OTP email.

### Step 1: Install Required Packages

Install the `pam-python` package to use Python-based PAM scripts:

```bash
sudo apt update
sudo apt install libpam-python
```

### Step 2: Create the PAM Python Script

Create a Python script to handle OTP generation and email delivery. You can save it as `/etc/security/pam_email_otp.py`.

```bash
sudo nano /etc/security/pam_email_otp.py
```

Add the following code:

```python
import pam
import random
import smtplib
import os

def send_email(to_email, otp_code):
    from_email = "your_email@example.com"
    smtp_server = "smtp.example.com"
    smtp_port = 587
    smtp_user = "your_email@example.com"
    smtp_pass = "your_email_password"

    with smtplib.SMTP(smtp_server, smtp_port) as server:
        server.starttls()
        server.login(smtp_user, smtp_pass)
        message = f"Subject: SSH OTP\n\nYour OTP is: {otp_code}"
        server.sendmail(from_email, to_email, message)

def pam_sm_authenticate(pamh, flags, argv):
    try:
        user_email = "user@example.com"  # replace with your email
        otp_code = str(random.randint(100000, 999999))
        
        send_email(user_email, otp_code)
        response = pamh.conversation(pamh.Message(pamh.PAM_PROMPT_ECHO_OFF, "Enter OTP: "))

        if response.resp == otp_code:
            return pamh.PAM_SUCCESS
        else:
            return pamh.PAM_AUTH_ERR
    except Exception as e:
        print(f"Error: {e}")
        return pamh.PAM_AUTH_ERR
```

Replace `your_email@example.com`, `smtp.example.com`, and other email credentials with your actual SMTP settings.

### Step 3: Set Permissions

Ensure the script has the correct permissions:

```bash
sudo chmod 700 /etc/security/pam_email_otp.py
sudo chown root:root /etc/security/pam_email_otp.py
```

### Step 4: Configure PAM for SSH Authentication

Edit the PAM configuration file for SSH at `/etc/pam.d/sshd` to include your custom OTP module.

```bash
sudo nano /etc/pam.d/sshd
```

Add this line at the top of the file:

```bash
auth required pam_python.so /etc/security/pam_email_otp.py
```

### Step 5: Configure SSH to Use PAM

Ensure that PAM is enabled for SSH in the SSH configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```

Make sure the following line is set:

```bash
UsePAM yes
```

### Step 6: Restart SSH Service

After making changes, restart the SSH service:

```bash
sudo systemctl restart ssh
```

### Step 7: Test SSH Login with OTP

Now, when you try to SSH into the server, it should prompt you for the OTP after sending it via email. The SSH login flow should be:

1. User enters their SSH password.
2. User is prompted for an OTP, which is sent to the configured email.
3. User enters the OTP to gain access.

---

This configuration sets up a basic OTP over email for SSH logins using PAM on Ubuntu. Adjust the email settings and Python script as needed to fit your environment and security requirements.
