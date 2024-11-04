Setting up SSH login with email-based OTP (One-Time Password) authentication on an Ubuntu server, combined with `ForceCommand` for additional control, requires several steps. Since you want to avoid using `pip` for package installations, we’ll rely on Python’s built-in libraries for SMTP email sending. Here’s a guide to achieve this setup:

### 1. Prerequisites

- Ensure the server has SSH and Python installed (Python is pre-installed on most Ubuntu installations).
- You need access to an SMTP server (Gmail, Outlook, or a custom SMTP server) for sending the OTP emails.
- Administrative access to modify SSH and server configurations.

### 2. Create a Python Script for OTP Authentication

This script will:
1. Generate a one-time password.
2. Send the OTP to the user's email via SMTP.
3. Verify the OTP entered during SSH login.

#### Example Python Script (`/usr/local/bin/ssh_otp_auth.py`)

Create a Python script at `/usr/local/bin/ssh_otp_auth.py` with the following code:

```python
#!/usr/bin/env python3

import os
import smtplib
import random
import sys
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

# SMTP configuration
SMTP_SERVER = "smtp.your_email_provider.com"
SMTP_PORT = 587  # Typically 587 for TLS or 465 for SSL
SMTP_USERNAME = "your_email@example.com"
SMTP_PASSWORD = "your_email_password"
SENDER_EMAIL = "your_email@example.com"

# Generate and send OTP
def send_otp(email):
    otp = str(random.randint(100000, 999999))
    message = MIMEMultipart()
    message['From'] = SENDER_EMAIL
    message['To'] = email
    message['Subject'] = "Your SSH OTP Code"

    # Message body
    body = f"Your OTP code for SSH login is: {otp}"
    message.attach(MIMEText(body, 'plain'))

    # Send email via SMTP
    try:
        with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
            server.starttls()  # Use TLS
            server.login(SMTP_USERNAME, SMTP_PASSWORD)
            server.sendmail(SENDER_EMAIL, email, message.as_string())
        return otp
    except Exception as e:
        print(f"Error sending OTP: {e}")
        sys.exit(1)

# Main function
def main():
    user_email = os.environ.get("SSH_OTP_EMAIL")
    if not user_email:
        print("No email provided. Set SSH_OTP_EMAIL environment variable.")
        sys.exit(1)

    otp = send_otp(user_email)
    print("OTP sent to your email. Please enter it to proceed.")

    # Prompt user to enter OTP
    entered_otp = input("Enter OTP: ").strip()
    if entered_otp == otp:
        print("OTP verified. Access granted.")
    else:
        print("Invalid OTP. Access denied.")
        sys.exit(1)

if __name__ == "__main__":
    main()
```

#### Make the Script Executable

```bash
sudo chmod +x /usr/local/bin/ssh_otp_auth.py
```

### 3. Configure SSH to Use `ForceCommand` for OTP Authentication

1. Open the SSH configuration file for editing:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Add the following lines to the end of `sshd_config`:

   ```plaintext
   Match User your_ssh_user  # Replace with the username you want to secure with OTP
       ForceCommand /usr/local/bin/ssh_otp_auth.py
       PermitTunnel no
       AllowAgentForwarding no
       AllowTcpForwarding no
       X11Forwarding no
   ```

   This configuration will force the OTP script to run whenever the specified user logs in via SSH.

3. Set up the environment variable `SSH_OTP_EMAIL` to hold the user's email for OTP delivery. Add this to the user's `.bashrc` or `/etc/environment` file if you want it globally accessible:

   ```bash
   echo 'export SSH_OTP_EMAIL="user_email@example.com"' >> ~/.bashrc
   source ~/.bashrc
   ```

   Alternatively, set `SSH_OTP_EMAIL` directly in the SSH configuration if you want a specific email per user:

   ```plaintext
   Match User your_ssh_user
       SetEnv SSH_OTP_EMAIL=user_email@example.com
   ```

### 4. Restart SSH

After configuring SSH, restart the SSH daemon for changes to take effect:

```bash
sudo systemctl restart ssh
```

### 5. Testing the OTP Authentication

1. Open a new terminal and SSH into the server with the specified user:

   ```bash
   ssh your_ssh_user@your_server_ip
   ```

2. The server will:
   - Trigger the Python script to generate and send the OTP.
   - Prompt you to enter the OTP.

3. Enter the OTP received in the email to complete the login process.

### Troubleshooting Tips

- **SMTP Authentication Errors**: Double-check your SMTP settings, username, and password. If using Gmail, ensure [Less Secure App Access](https://myaccount.google.com/lesssecureapps) is enabled.
- **Environment Variable**: If `SSH_OTP_EMAIL` isn’t being recognized, ensure it's exported correctly in the `.bashrc` file or SSH config.
- **Python SMTP Library**: Python’s built-in `smtplib` does not need `pip` installation but ensure Python 3 is installed (`python3 --version`).

This setup should now allow OTP-based SSH login for the specified user, with the email OTP sent via SMTP before granting access.

😯🥲🥱🤭🙂😢🙂😢🤭😢😐🙃😢😐🙃😢😐🙂😢🤭🙂😢🤭🙂😢🤭🙃😢🙃🤭😮🤭😥🤭🙃😦🙃😦🤭🙃🙃🤭😦🙃🤭🤭🙂😮🤭🙃🤭😮🤭🙂😮🤭🙃😮😮🤭🙂😢😢🤭🙂😥🤭🙂😥🙂😐😢🙂😐😢😙😐😮🤭🙂😮🤭🙂😮🤭🙂😮😮🤭🙂😮😮🤭🙂😮🤭😮🤭🙂😦😮🤭🙂😮🤭🙂😮🙂😮🤭🙂
Setting up SSH login on an Ubuntu server using email link click authentication involves several steps. This approach typically requires some additional software, as native SSH does not support email link click authentication by default. One popular method is to use a combination of SSH with a web server to handle the email link, and `ForceCommand` in the SSH config to enforce the authentication method.

Here's a step-by-step guide:

### Prerequisites
1. **Ubuntu Server**: Ensure you have an Ubuntu server running.
2. **Domain Name**: You should have a domain name pointing to your server's IP address.
3. **Email Setup**: You need a working email server or a third-party service to send emails.

### Step 1: Install Necessary Software
You need a web server and a scripting language. You can use Apache or Nginx and PHP, or you could use Python or Node.js.

For this example, let’s assume you're using Apache and PHP:

```bash
sudo apt update
sudo apt install apache2 php libapache2-mod-php
```

### Step 2: Create a Simple PHP Script for Authentication

1. Navigate to the web directory:

   ```bash
   cd /var/www/html
   ```

2. Create a file called `auth.php`:

   ```bash
   sudo nano auth.php
   ```

3. Add the following code to handle authentication:

   ```php
   <?php
   session_start();
   
   if (isset($_GET['token'])) {
       $token = $_GET['token'];

       // Validate the token (you need to implement token generation and storage logic)
       // If valid, allow SSH access
       if (validate_token($token)) {
           $_SESSION['authenticated'] = true;
           // Redirect to SSH
           header('Location: ssh://your_username@your_server_ip');
           exit();
       } else {
           echo "Invalid token.";
       }
   }

   function validate_token($token) {
       // Implement your token validation logic here
       // Check against a database or a predefined list
       return true; // Placeholder, replace with actual validation
   }
   ?>
   ```

4. Save and exit the file.

### Step 3: Generate and Send the Authentication Link

1. You need a script to generate the email with a unique link:

   ```php
   // Send email function (simplified, use PHPMailer for production)
   function send_email($email, $token) {
       $subject = "Your SSH Login Link";
       $link = "http://your_domain/auth.php?token=" . $token;
       $message = "Click this link to authenticate: " . $link;

       mail($email, $subject, $message);
   }
   ```

2. Implement a way to generate a unique token, store it securely, and call the `send_email` function with the user’s email.

### Step 4: Configure SSH to Use `ForceCommand`

1. Open the SSH configuration file:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Add the following line at the end:

   ```plaintext
   Match User your_username
       ForceCommand /path/to/your/script.sh
   ```

   Replace `/path/to/your/script.sh` with the path to your script that handles the logic for checking if the user is authenticated. This could be a script that checks if the session variable is set.

3. Create the script `script.sh`:

   ```bash
   sudo nano /path/to/your/script.sh
   ```

   ```bash
   #!/bin/bash
   if [ ! -z "$SSH_CONNECTION" ]; then
       # Check if the user is authenticated via a mechanism you implemented
       if [ "$(curl -s http://your_domain/check_auth.php)" != "authenticated" ]; then
           echo "Authentication required."
           exit 1
       fi
   fi
   ```

4. Make the script executable:

   ```bash
   sudo chmod +x /path/to/your/script.sh
   ```

### Step 5: Restart SSH Service

After making changes to the SSH configuration, restart the SSH service:

```bash
sudo systemctl restart ssh
```

### Step 6: Testing

1. To test, try SSH into your server:

   ```bash
   ssh your_username@your_server_ip
   ```

2. If the ForceCommand checks fail (i.e., the session variable is not set), it should prompt for authentication via the email link.

### Important Security Considerations

- **Token Security**: Ensure that the token generation and validation logic is secure to prevent unauthorized access.
- **HTTPS**: Use HTTPS for your web server to secure the token transmission.
- **Session Management**: Implement session management properly to prevent session hijacking.
- **Firewall**: Ensure your server is secured with proper firewall rules.

This method requires thorough testing and security hardening before using in a production environment. Consider alternatives like MFA (Multi-Factor Authentication) if security is a primary concern.
😢😙😢😙😐😙😛😮‍💨😗😛😢😛😮‍💨😛😗😢😐😗😢😐😗😢😐😮‍💨😐😗😢😗😢😐😗😢😐😗😢😐😗😢😐😗😢😐😗😢😶🙃☹️😶☹️☹️😶😗☹️😶🙃☹️😶😗☹️😶😗☹️😶🙃☹️😶😗☹️😗🫠☹️😝😗☹️😶😗☹️😶😗😯🙃☹️😶😗😢😐😗☹️🙃🫠☹️😝😗😮‍💨😮‍💨😛😗😮‍💨😝😗😤😛😃😮‍💨😃😤😝😃😮‍💨


Setting up SSH login with SMS authentication on an Ubuntu server involves several steps, including configuring Twilio for SMS, updating the SSH configuration, and creating a script to handle the SMS authentication. Below is a guide to help you accomplish this without using a Python pipeline.

### Step 1: Install Required Packages

Make sure your server has `curl` and `twilio` CLI installed. You can install them with the following commands:

```bash
sudo apt update
sudo apt install curl
```

### Step 2: Set Up Twilio Account

1. **Sign Up for Twilio**: Create an account on the [Twilio website](https://www.twilio.com/).

2. **Get Twilio Credentials**: After signing up, note your `Account SID`, `Auth Token`, and a phone number that you will use to send SMS messages.

### Step 3: Create the SMS Authentication Script

Create a shell script that will handle the SMS authentication. For example, you can create a script called `sms_auth.sh`:

```bash
sudo nano /usr/local/bin/sms_auth.sh
```

Add the following code to the script:

```bash
#!/bin/bash

# Twilio credentials
TWILIO_ACCOUNT_SID="your_account_sid"
TWILIO_AUTH_TOKEN="your_auth_token"
TWILIO_PHONE_NUMBER="your_twilio_phone_number"
TO_PHONE_NUMBER="$1" # The user's phone number
SMS_CODE=$(shuf -i 1000-9999 -n 1) # Generate a random 4-digit code

# Send the SMS via Twilio
curl -X POST https://api.twilio.com/2010-04-01/Accounts/$TWILIO_ACCOUNT_SID/Messages.json \
--data-urlencode "To=$TO_PHONE_NUMBER" \
--data-urlencode "From=$TWILIO_PHONE_NUMBER" \
--data-urlencode "Body=Your authentication code is: $SMS_CODE" \
-u $TWILIO_ACCOUNT_SID:$TWILIO_AUTH_TOKEN

# Prompt for the code entered by the user
read -p "Enter the code sent to your phone: " USER_CODE

if [ "$USER_CODE" == "$SMS_CODE" ]; then
    echo "Authentication successful!"
    exit 0
else
    echo "Authentication failed!"
    exit 1
fi
```

### Step 4: Make the Script Executable

Make the script executable by running:

```bash
sudo chmod +x /usr/local/bin/sms_auth.sh
```

### Step 5: Configure SSHD to Use the Script

Edit the SSH configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```

Add the following line at the end of the file:

```plaintext
ForceCommand /usr/local/bin/sms_auth.sh %u
```

This configuration ensures that every SSH login attempts to invoke the SMS authentication script before proceeding.

### Step 6: Restart the SSH Service

After editing the SSH configuration file, restart the SSH service to apply the changes:

```bash
sudo systemctl restart sshd
```

### Step 7: Test the Configuration

Attempt to SSH into your server from a different terminal:

```bash
ssh your_username@your_server_ip
```

You should receive an SMS with a code on the phone number specified, and you'll be prompted to enter the code.

### Security Considerations

1. **Do Not Expose the Script**: Make sure your script is secured and not accessible by unauthorized users.
2. **Two-Factor Authentication**: This setup provides an extra layer of security but consider combining it with other forms of authentication (like SSH keys) for even better security.
3. **Twilio Costs**: Sending SMS via Twilio incurs costs; monitor your usage.

By following these steps, you will have a working SMS authentication system for SSH logins on your Ubuntu server.

