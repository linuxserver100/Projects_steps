To configure SSH login on an Ubuntu server using email OTP (One-Time Password) authentication without relying on PAM (Pluggable Authentication Modules) or modifying `.bashrc`, you can follow these steps. This method will involve using a script for OTP generation and validation.

### Prerequisites

1. **Ubuntu Server** with SSH installed and running.
2. **Email account** to send OTPs.
3. **`sendmail`** or any mail utility installed on your server.

### Step 1: Install Required Packages

Make sure you have `sendmail` or `mailutils` installed:

```bash
sudo apt update
sudo apt install sendmail
```

### Step 2: Create an OTP Generation Script

Create a script that generates a random OTP and sends it via email. Save this script as `/usr/local/bin/otp_auth.sh`.

```bash
sudo nano /usr/local/bin/otp_auth.sh
```

Add the following content to the script:

```bash
#!/bin/bash

# Configuration
EMAIL="your_email@example.com"
OTP_FILE="/tmp/otp.txt"

# Generate a random 6-digit OTP
OTP=$(shuf -i 100000-999999 -n 1)

# Send OTP via email
echo "Your OTP is: $OTP" | sendmail $EMAIL

# Store OTP in a temporary file
echo $OTP > $OTP_FILE
chmod 600 $OTP_FILE
```

Replace `your_email@example.com` with your actual email address.

Make the script executable:

```bash
sudo chmod +x /usr/local/bin/otp_auth.sh
```

### Step 3: Modify SSH Configuration

Edit your SSH configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```

Add or modify the following settings:

```plaintext
ChallengeResponseAuthentication yes
```

### Step 4: Create the OTP Verification Script

Create a script to verify the OTP input by the user. Save this as `/usr/local/bin/otp_verify.sh`.

```bash
sudo nano /usr/local/bin/otp_verify.sh
```

Add the following content:

```bash
#!/bin/bash

# Configuration
OTP_FILE="/tmp/otp.txt"

# Read the OTP from the temporary file
if [ ! -f "$OTP_FILE" ]; then
    echo "OTP expired or not generated."
    exit 1
fi

# Prompt for OTP
read -p "Enter the OTP: " USER_OTP

# Read the stored OTP
STORED_OTP=$(cat $OTP_FILE)

# Verify the OTP
if [ "$USER_OTP" == "$STORED_OTP" ]; then
    echo "OTP verified."
    rm -f $OTP_FILE # Remove OTP after verification
    exit 0
else
    echo "Invalid OTP."
    exit 1
fi
```

Make this script executable as well:

```bash
sudo chmod +x /usr/local/bin/otp_verify.sh
```

### Step 5: Update `.ssh/authorized_keys`

Add a command to the SSH authorized keys for the users who should use OTP. For example, if your public key is in `~/.ssh/authorized_keys`, prepend the command:

```plaintext
command="/usr/local/bin/otp_verify.sh",no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty ssh-rsa AAAAB3... your_email@example.com
```

### Step 6: Restart SSH Service

Restart the SSH service to apply the changes:

```bash
sudo systemctl restart ssh
```

### Step 7: Testing

1. Attempt to SSH into your server.
2. After entering the username and password, the OTP will be sent to your email.
3. Enter the OTP in the prompt to gain access.

### Note

- Ensure your email server is configured correctly to send emails.
- This setup does not utilize PAM and relies on a custom solution, which may not be as secure or robust as using established libraries and protocols. Use with caution in production environments.
- Consider implementing additional security measures as needed.



🥰🥰😄😃🥰😌😃😅😃😅😗😌😃🤗😍🗣️😶‍🌫️😍😶‍🌫️🗣️😶‍🌫️🗣️😍🗣️😍🗣️😶‍🌫️😍🤗🗣️😍🗣️😍🗣️🤗😍😶‍🌫️🗣️😍😶‍🌫️🗣️😍😶‍🌫️🗣️😍😶‍🌫️🗣️😶‍🌫️🗣️😍😍😶‍🌫️🗣️😍🎃😶‍🌫️🎃☺️🤮😌😶‍🌫️🗣️😍🗣️😌🎃🤮🤣😌🤣🎃😌🤣🤣🤮😌🎃😌😌🎃☺️☺️🎃🥱☺️🤮☺️🎃😶‍🌫️☺️🫨☺️☺️☺️🤪🥱🥱🫨🎃🤪😌🤪🤮🤪🤪🤣🥱🥱😜🤮🎃🎃🤪🤪🎃🤪🤪😶‍🌫️🎃😌🤣🎃🤪🤪🎃🤪🎃🤪😶‍🌫️🎃🤪😶‍🌫️🎃🤪😶‍🌫️🎃🤪🎃😜😶‍🌫️🤪🎃😌😶‍🌫️🤣🎃🤪🎃🤣😌🤣🎃😌🎃😌
To configure SSH login on an Ubuntu server using email-based OTP (One-Time Password) authentication without PAM or modifying `.bashrc`, you can set up a custom solution. Here's a high-level outline of the steps involved:

### Prerequisites

1. **Mail Server**: Ensure your server can send emails (e.g., using Postfix or another mail service).
2. **Python**: Install Python for scripting (if not already installed).

### Steps to Configure SSH with Email OTP

1. **Install Required Packages**:
   ```bash
   sudo apt update
   sudo apt install python3 python3-smtplib
   ```

2. **Create OTP Generation Script**:
   Create a Python script to generate and send the OTP via email.

   ```bash
   sudo nano /usr/local/bin/send_otp.py
   ```

   Insert the following code (customize email settings):

   ```python
   import smtplib
   import random
   import sys

   def send_email(to_email, otp):
       from_email = 'your_email@example.com'
       subject = 'Your OTP Code'
       body = f'Your OTP code is: {otp}'

       with smtplib.SMTP('smtp.example.com', 587) as server:
           server.starttls()
           server.login(from_email, 'your_email_password')
           message = f'Subject: {subject}\n\n{body}'
           server.sendmail(from_email, to_email, message)

   if __name__ == "__main__":
       user_email = sys.argv[1]
       otp = str(random.randint(100000, 999999))
       send_email(user_email, otp)
       print(otp)  # Print OTP for later comparison
   ```

   Make it executable:
   ```bash
   sudo chmod +x /usr/local/bin/send_otp.py
   ```

3. **Modify SSH Configuration**:
   Edit your SSH configuration to allow command execution after login attempts.

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

   Ensure the following settings are present:
   ```
   ChallengeResponseAuthentication yes
   ```

4. **Create the OTP Verification Script**:
   Create another script to handle OTP verification.

   ```bash
   sudo nano /usr/local/bin/otp_verification.sh
   ```

   Add the following content:

   ```bash
   #!/bin/bash

   read -p "Enter your email: " user_email
   otp_generated=$(python3 /usr/local/bin/send_otp.py "$user_email")

   read -p "Enter the OTP sent to your email: " user_otp

   if [ "$user_otp" == "$otp_generated" ]; then
       echo "Authentication successful."
       exit 0
   else
       echo "Invalid OTP."
       exit 1
   fi
   ```

   Make it executable:
   ```bash
   sudo chmod +x /usr/local/bin/otp_verification.sh
   ```

5. **Set Up SSH Command**:
   You need to create a mechanism to call the OTP verification script after a user tries to log in via SSH. One way to do this is to use a forced command in the `~/.ssh/authorized_keys` file.

   For each user, edit their `authorized_keys`:
   ```bash
   nano /home/username/.ssh/authorized_keys
   ```

   Add the following line before the key:
   ```
   command="/usr/local/bin/otp_verification.sh",no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty ssh-rsa AAAAB3Nza...
   ```

6. **Restart SSH Service**:
   After making changes, restart the SSH service:

   ```bash
   sudo systemctl restart ssh
   ```

### Testing

1. Try to SSH into the server.
2. The OTP script will prompt for the email and send an OTP.
3. You will need to enter the OTP to complete the login process.

### Notes

- This setup is fairly basic and does not include error handling or security improvements.
- Make sure to use secure methods for sending emails and storing passwords (consider using environment variables or secure storage).
- For production, consider using established libraries or services for OTP generation and management. 

This solution provides a basic framework for email OTP without PAM or `.bashrc` modifications. Adjust it according to your specific requirements and security practices.
