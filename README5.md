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
