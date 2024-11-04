To set up SSH login on an Ubuntu server with email-based One-Time Password (OTP) authentication, you can use the Pluggable Authentication Module (PAM). PAM is a framework used on Linux systems to integrate various authentication technologies, making it possible to create custom authentication methods.

Here’s a step-by-step guide to configuring SSH with email OTP using PAM:

### Prerequisites
1. **Ubuntu Server** with SSH access.
2. A valid SMTP server or email setup that can send OTP emails to users.

### Steps

#### Step 1: Install Required Packages
You’ll need `libpam-python` to allow Python scripts to be used in PAM, as well as `ssmtp` or `sendmail` (or a similar package) to send emails.

```bash
sudo apt update
sudo apt install libpam-python ssmtp
```

#### Step 2: Configure Email Sending Utility
Edit the `ssmtp` configuration file to set up email sending with your SMTP server.

```bash
sudo nano /etc/ssmtp/ssmtp.conf
```

Configure the following parameters as per your SMTP server:

```plaintext
root=your-email@example.com
mailhub=smtp.your-email-provider.com:587
AuthUser=your-email@example.com
AuthPass=your-email-password
FromLineOverride=YES
UseTLS=YES
```

Replace the values accordingly, then save and exit.

#### Step 3: Create a Python Script for OTP Email

Create a script that generates an OTP, emails it to the user, and checks the input OTP. Save it in `/etc/security/otp_email.py`.

```bash
sudo nano /etc/security/otp_email.py
```

Paste the following code into the file:

```python
import random
import smtplib
import pam
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

OTP_STORAGE = "/tmp/otp_storage"

def generate_otp():
    return random.randint(100000, 999999)

def send_email(user_email, otp):
    smtp_server = "smtp.your-email-provider.com"
    smtp_port = 587
    smtp_user = "your-email@example.com"
    smtp_pass = "your-email-password"
    
    subject = "Your OTP Code for SSH Login"
    message = f"Your OTP code is: {otp}"
    
    msg = MIMEMultipart()
    msg["From"] = smtp_user
    msg["To"] = user_email
    msg["Subject"] = subject
    msg.attach(MIMEText(message, "plain"))
    
    with smtplib.SMTP(smtp_server, smtp_port) as server:
        server.starttls()
        server.login(smtp_user, smtp_pass)
        server.sendmail(smtp_user, user_email, msg.as_string())

def pam_sm_authenticate(pamh, flags, argv):
    user = pamh.get_user()
    if not user:
        return pamh.PAM_USER_UNKNOWN

    otp = generate_otp()
    user_email = f"{user}@example.com"  # Adjust to use the user's email
    send_email(user_email, otp)
    
    with open(OTP_STORAGE, "w") as otp_file:
        otp_file.write(str(otp))

    user_input = pamh.conversation(pamh.Message(pamh.PAM_PROMPT_ECHO_OFF, "Enter OTP sent to your email: "))

    with open(OTP_STORAGE, "r") as otp_file:
        saved_otp = otp_file.read().strip()

    if user_input != saved_otp:
        return pamh.PAM_AUTH_ERR
    
    return pamh.PAM_SUCCESS
```

Save and exit. Then set the appropriate permissions for the script:

```bash
sudo chmod 700 /etc/security/otp_email.py
```

Replace `smtp.your-email-provider.com`, `your-email@example.com`, and `your-email-password` with actual values for your SMTP setup.

#### Step 4: Configure PAM to Use the OTP Script
Edit the SSH PAM configuration file to include the new OTP mechanism.

```bash
sudo nano /etc/pam.d/sshd
```

Add the following line at the beginning of the file to include the OTP authentication script:

```plaintext
auth required pam_python.so /etc/security/otp_email.py
```

This tells PAM to require successful execution of the OTP Python script for SSH login.

#### Step 5: Configure SSH to Use PAM
Edit the SSH configuration file to ensure it’s set to use PAM for authentication.

```bash
sudo nano /etc/ssh/sshd_config
```

Find and set the following parameters:

```plaintext
ChallengeResponseAuthentication yes
UsePAM yes
```

Save and exit the file, then restart SSH to apply the changes.

```bash
sudo systemctl restart ssh
```

#### Step 6: Test SSH Login
1. Try to log in via SSH to the server.
2. You should be prompted to enter an OTP, which will be sent to your configured email.
3. Enter the OTP to complete the login process.

### Explanation of PAM Authentication Workflow
When you log in via SSH:
1. PAM checks the authentication modules in `/etc/pam.d/sshd`.
2. The `pam_python.so` module calls the OTP script, which generates a code, emails it, and verifies it against the user input.
3. If the OTP is correct, authentication succeeds; otherwise, it fails.

This setup allows a second factor of authentication via email, enhancing the security of SSH login.
