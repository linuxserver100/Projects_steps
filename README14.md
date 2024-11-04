To set up SSH login on an Ubuntu server using email-based OTP authentication along with a password, without using PAM or modifying `.bashrc`, follow these steps:

### Step 1: Install Required Packages

You'll need to install a few packages. Use the following commands:

```bash
sudo apt update
sudo apt install openssh-server sendmail
```

### Step 2: Set Up OTP Generation

You can use a simple Python script to send OTPs via email. First, ensure you have Python and pip installed:

```bash
sudo apt install python3 python3-pip
```

Then install the necessary library for sending emails:

```bash
pip3 install secure-smtplib
```

### Step 3: Create the OTP Script

Create a script to generate and send OTPs. Save the following code as `otp_sender.py`:

```python
import smtplib
import random
import string

# Configuration
EMAIL_ADDRESS = 'your_email@example.com'
EMAIL_PASSWORD = 'your_email_password'
SMTP_SERVER = 'smtp.example.com'
SMTP_PORT = 587

def send_otp(to_email):
    otp = ''.join(random.choices(string.digits, k=6))
    
    # Create email content
    subject = "Your OTP Code"
    body = f"Your OTP code is: {otp}"
    message = f"Subject: {subject}\n\n{body}"
    
    # Send email
    with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
        server.starttls()
        server.login(EMAIL_ADDRESS, EMAIL_PASSWORD)
        server.sendmail(EMAIL_ADDRESS, to_email, message)
    
    return otp

if __name__ == "__main__":
    email = input("Enter your email: ")
    otp = send_otp(email)
    print(f"OTP sent to {email}.")
```

### Step 4: Update SSH Configuration

Edit the SSH configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```

Ensure these lines are present or add them if they are missing:

```plaintext
ChallengeResponseAuthentication yes
```

### Step 5: Create a Custom Authentication Script

You need to create a script that will handle OTP verification. Create a script named `otp_auth.sh`:

```bash
sudo nano /usr/local/bin/otp_auth.sh
```

Add the following content:

```bash
#!/bin/bash

# This script prompts for the OTP and validates it.

OTP_FILE="/tmp/otp_code"

# Prompt for the OTP
read -p "Enter your OTP: " user_otp

if [ -f "$OTP_FILE" ]; then
    stored_otp=$(cat "$OTP_FILE")
    if [ "$user_otp" == "$stored_otp" ]; then
        echo "OTP verified."
        exit 0
    else
        echo "Invalid OTP."
        exit 1
    fi
else
    echo "No OTP found."
    exit 1
fi
```

Make the script executable:

```bash
sudo chmod +x /usr/local/bin/otp_auth.sh
```

### Step 6: Modify the Login Process

Edit your `authorized_keys` file for the user(s) you want to set up OTP for, adding the following line before the public key:

```bash
command="/usr/local/bin/otp_auth.sh; exec /bin/bash",no-port-forwarding,no-X11-forwarding ssh-rsa AAAAB3... your_email@example.com
```

Replace the `ssh-rsa ...` part with the actual public key.

### Step 7: Test the Setup

1. **Generate and Send OTP**: Run the `otp_sender.py` script to generate and send the OTP to your email.

2. **Log In via SSH**: Attempt to SSH into your server. After entering your password, it should prompt you for the OTP.

### Step 8: Cleanup

Ensure that the OTP file is removed after use:

Modify `otp_auth.sh`:

```bash
# Add this line after verifying the OTP
rm -f "$OTP_FILE"
```

### Note

- This setup is a simple implementation and should be secured further for production use.
- Make sure to use secure methods to store and handle your email credentials.
- Consider adding error handling and logging to the scripts for better reliability.
