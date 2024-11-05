
To enforce an OTP check using SSH with `ForceCommand`, you can set up a script that generates a one-time password (OTP) and sends it via email using `ssmtp`. This guide assumes you have some familiarity with SSH, shell scripting, and basic server configuration. 

Here's how to set it up:

### Prerequisites
1. **Ensure that you have root access** to configure SSH and install necessary packages.
2. **Install `ssmtp`** to send emails. On Debian/Ubuntu, use:
   ```bash
   sudo apt update
   sudo apt install ssmtp
   ```
   Configure `/etc/ssmtp/ssmtp.conf` with your SMTP settings (e.g., Gmail or your mail server).

```bash
   root=your_email@example.com
   mailhub=smtp.example.com:587
   AuthUser=your_email@example.com
   AuthPass=your_email_password
   UseSTARTTLS=YES
   FromLineOverride=YES

   ```


### Steps to Implement SSH OTP Check

#### Step 1: Set Up an OTP Generation Script
Create a script that will generate an OTP, email it to the user, and check for user input.

1. **Create the OTP Script**:
   ```bash
   sudo nano /usr/local/bin/ssh-otp-check.sh
   ```

2. **Add the Following Script Content**:
   ```bash
   #!/bin/bash

   # Configuration: Set the user's email and OTP validity period in seconds
   EMAIL="user@example.com"
   OTP_VALIDITY=300  # OTP is valid for 5 minutes

   # Generate a random 6-digit OTP
   OTP=$(shuf -i 100000-999999 -n 1)

   # Send the OTP to the user's email using ssmtp
   echo -e "Subject: SSH OTP Authentication\n\nYour OTP is: $OTP" | ssmtp $EMAIL

   # Request OTP input from the user
   echo "Enter the OTP sent to your email:"
   read -t $OTP_VALIDITY -p "OTP: " USER_INPUT

   # Verify if the input matches the generated OTP
   if [[ "$USER_INPUT" == "$OTP" ]]; then
       echo "OTP verified successfully."
       exec $SHELL
   else
       echo "Invalid OTP or OTP expired."
       exit 1
   fi
   ```

3. **Make the Script Executable**:
   ```bash
   sudo chmod +x /usr/local/bin/ssh-otp-check.sh
   ```

#### Step 2: Configure SSH to Use `ForceCommand`
Now, configure SSH to use this script as a `ForceCommand` to enforce OTP verification on login.

1. **Edit the SSH Configuration File**:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. **Add/Modify the Following Lines**:
   ```bash
   Match User your_ssh_user  # Replace with the specific user or set conditions for OTP enforcement
       ForceCommand /usr/local/bin/ssh-otp-check.sh
   ```

   *Note*: This `Match User` section will enforce the OTP script only for the specified user(s). Adjust as needed.

3. **Restart the SSH Service**:
   ```bash
   sudo systemctl restart ssh
   ```

#### Step 3: Test the Configuration
1. Attempt an SSH connection as the specified user.
2. You should receive an OTP via email.
3. Enter the OTP when prompted in the SSH session.
4. If correct, you’ll be granted access. Otherwise, the session will terminate.

### Notes
- **Security Considerations**: This is a basic OTP implementation and is suitable for testing and internal use but may not be as secure as established OTP solutions (like Google Authenticator).
- **ssmtp Alternatives**: `ssmtp` is deprecated in some distributions, so you may consider `msmtp` or configuring `mailx` as an alternative for production environments.

This configuration should provide you with a basic OTP-enforced SSH login setup.

😯🥲🥱🤭🙂😢🙂😢🤭😢😐🙃😢😐🙃😢😐🙂😢🤭🙂😢🤭🙂😢🤭🙃😢🙃🤭😮🤭😥🤭🙃😦🙃😦🤭🙃🙃🤭😦🙃🤭🤭🙂😮🤭🙃🤭😮🤭🙂😮🤭🙃😮😮🤭🙂😢😢🤭🙂😥🤭🙂😥🙂😐😢🙂😐😢😙😐😮🤭🙂😮🤭🙂😮🤭🙂😮😮🤭🙂😮😮🤭🙂😮🤭😮🤭🙂😦😮🤭🙂😮🤭🙂😮🙂😮🤭🙂
Enforcing email link click authentication using SSH with `ForceCommand` and sending an email link for authentication involves several steps. Here’s a guide on how to set this up, including the necessary scripts and configurations.

### Prerequisites

1. **SSH Server**: Ensure you have an SSH server running (e.g., OpenSSH).
2. **Email Sending Utility**: You’ll need a utility like `ssmtp` or `msmtp` to send emails from the server.
3. **Web Server**: A web server to handle the link click (could be Nginx, Apache, etc.).
4. **Database or File Storage**: To store authentication tokens (optional).

### Steps to Implement Email Link Click Authentication

#### Step 1: Install `ssmtp`

If `ssmtp` is not installed, you can install it using the following commands:

For Debian/Ubuntu:
```bash
sudo apt-get update
sudo apt-get install ssmtp
```

For CentOS/RHEL:
```bash
sudo yum install ssmtp
```

#### Step 2: Configure `ssmtp`

Edit the configuration file `/etc/ssmtp/ssmtp.conf`:

```conf
root=your_email@example.com
mailhub=smtp.example.com:587
AuthUser=your_email@example.com
AuthPass=your_email_password
UseSTARTTLS=YES
UseTLS=YES
```

Make sure to replace the placeholder values with your actual email settings.

#### Step 3: Create the Email Link Script

Create a script named `send_auth_email.sh` that sends an authentication email with a unique link.

```bash
#!/bin/bash

# Variables
recipient=$1
token=$(openssl rand -hex 16)  # Generate a random token
link="http://yourdomain.com/verify.php?token=$token"

# Store the token (for example, in a file or database)
echo "$recipient:$token" >> /tmp/auth_tokens.txt  # Change to a secure method for production

# Send the email
subject="SSH Authentication Link"
body="Click the link to authenticate your SSH session: $link"

echo -e "Subject:$subject\n\n$body" | ssmtp $recipient
```

Make the script executable:

```bash
chmod +x send_auth_email.sh
```

#### Step 4: Create the Verification Script

Create a verification script named `verify.php` to handle link clicks. This script should check the token and allow SSH access if valid.

```php
<?php
$token = $_GET['token'];

// Read the stored tokens (make this secure in production)
$lines = file('/tmp/auth_tokens.txt');
foreach ($lines as $line) {
    list($email, $stored_token) = explode(':', trim($line));
    if ($stored_token === $token) {
        // Allow access, e.g., create a session or write to a file
        file_put_contents('/tmp/auth_success.txt', "$email\n", FILE_APPEND);
        echo "Authentication successful. You may now use SSH.";
        exit;
    }
}
echo "Invalid token.";
?>
```

#### Step 5: Configure SSH with `ForceCommand`

Edit your SSH server configuration file (usually `/etc/ssh/sshd_config`) and add a `ForceCommand` for the users you want to restrict.

```conf
Match User your_ssh_user
    ForceCommand /path/to/send_auth_email.sh %u@example.com
```

This will trigger the email script upon an SSH login attempt.

#### Step 6: Restart SSH Service

After editing the SSH configuration, restart the SSH service to apply the changes:

```bash
sudo systemctl restart sshd
```

#### Step 7: Implementing Authentication Check in SSH

You’ll need to modify the `ForceCommand` to check for the successful link click before allowing SSH access. This can be achieved through a simple script that checks for the success file.

Create a script named `check_auth.sh`:

```bash
#!/bin/bash

if [ -f /tmp/auth_success.txt ]; then
    echo "Access granted."
    exit 0  # Allow access
else
    echo "Access denied. Please check your email for the authentication link."
    exit 1  # Deny access
fi
```

Make this script executable:

```bash
chmod +x check_auth.sh
```

Then, update your `sshd_config`:

```conf
Match User your_ssh_user
    ForceCommand /path/to/check_auth.sh
```

#### Step 8: Final Steps

1. **Security Considerations**: Make sure to secure the token storage and web script properly to prevent unauthorized access.
2. **Testing**: Test the entire flow, from attempting an SSH login to clicking the email link.

### Summary

You have now set up an email link click authentication mechanism for SSH logins using `ForceCommand` and `ssmtp`. Users will receive an email with a unique link, and their SSH access will be validated upon clicking that link.


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

