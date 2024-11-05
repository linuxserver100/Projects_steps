
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

😯🥲🥱🤭🙂😢🙂😢🤭😢😐🙃😢😐🙃😢😐🙂😢🤭🙂😢🤭🙂😢🤭🙃😢🙃🤭😮🤭😥🤭🙃😦🙃😦🤭🙃🙃🤭😦🙃🤭🤭🙂😮🤭🙃🤭🤧😬🤧😬😢🤧😢🤫🤧😮🤫😢🤫🤧🤫😮🤧😯🤫🤧☹️🤫🤧☹️🤫🤒🤫☹️🤒☹️🫢☹️🤒☹️🤫🤒🙁🫢🤒🤒☹️😑🤒☹️😑🤒😑☹️🤒☹️😑🤒😑🤒☹️😑🤒☹️🫢🤒🫢☹️🤒🫢☹️🤒☹️🤫🤒🙁😑🤒☹️😑🤒☹️🫢🤒☹️😑🤒🙁🤒🙁😑🤒🙁😑🙁🤒😑🤒☹️😑🤒☹️🫢🤒☹️🫢☹️🤒☹️🤫🥸☹️🫢🤒☹️🫢
To enforce an email link click authentication using SSH and `ForceCommand`, you can use a combination of a custom script, email generation, and an SSH `ForceCommand` directive. Here's a step-by-step outline on how you could implement this:

### 1. **Prerequisites**
Ensure that the following tools and configurations are available on your system:
- **ssmtp or msmtp**: For sending emails via SMTP from your server.
- **SSH server**: Running and configured properly.
- **Web server**: A simple HTTP server to handle the link validation (you can use something lightweight like `nginx` or `apache2`).
- **SSH client with `ForceCommand`**: This allows you to specify a specific command to run when a user logs in via SSH.

### 2. **Setup Email Script**
The main idea is to send a unique authentication link to the user’s email address when they try to authenticate. This link should contain a token that is only valid for a limited time.

#### 2.1 **Install Required Tools**
First, make sure you have `ssmtp` or `msmtp` configured to send emails.

```bash
sudo apt-get install ssmtp
```

Make sure to configure `ssmtp` or `msmtp` with your SMTP credentials. For example, edit `/etc/ssmtp/ssmtp.conf` to include your SMTP server settings:

```bash
root=postmaster@example.com
mailhub=smtp.example.com:587
AuthUser=username@example.com
AuthPass=yourpassword
FromLineOverride=YES
UseSTARTTLS=YES
```

#### 2.2 **Script for Sending Email**
Write a script that generates a random token and sends an email with a validation link.

```bash
#!/bin/bash

# User email
USER_EMAIL=$1

# Generate a random token (e.g., 32 characters)
TOKEN=$(openssl rand -base64 32)

# Store the token for later verification (you can use a database or flat file)
echo "$USER_EMAIL:$TOKEN" > /tmp/valid_tokens.txt

# Define the validation link
VALIDATION_URL="https://yourserver.com/validate?token=$TOKEN"

# Send email with the validation link
echo -e "Subject: SSH Login Verification\n\nClick the link to authenticate your login: $VALIDATION_URL" | ssmtp $USER_EMAIL
```

Make sure this script is executable:

```bash
chmod +x /path/to/send_auth_email.sh
```

### 3. **Create a Validation Endpoint**
You need to create an HTTP endpoint on your server that will handle the token validation. This can be a simple web application.

#### Example using a simple PHP script:

```php
<?php
$valid_tokens = file_get_contents('/tmp/valid_tokens.txt');
$valid_tokens = explode("\n", $valid_tokens);

$token = $_GET['token'];

foreach ($valid_tokens as $line) {
    list($email, $valid_token) = explode(":", $line);
    if ($token === $valid_token) {
        // Token is valid, allow SSH login
        echo "Token valid. You may now log in.";
        exit(0);
    }
}

echo "Invalid or expired token.";
exit(1);
?>
```

Place this script on your web server, e.g., `/var/www/html/validate.php`.

### 4. **ForceCommand Setup in SSH Configuration**
Now, modify the SSH server configuration (`/etc/ssh/sshd_config`) to enforce the use of a custom script via `ForceCommand`.

#### 4.1 **Configure `ForceCommand`**
Modify `/etc/ssh/sshd_config` to force the execution of a script that will handle the token check:

```bash
ForceCommand /path/to/authentication_check.sh
```

This script will be executed whenever a user logs in via SSH. 

#### 4.2 **Script for Authentication Check**
Write a script (`/path/to/authentication_check.sh`) that will verify whether the user clicked the link before allowing SSH access.

```bash
#!/bin/bash

USER_EMAIL=$(whoami)  # or use a method to get the user email
VALIDATION_URL="https://yourserver.com/validate?token=$TOKEN"

# Check if the user clicked the link and if the token is valid
if curl --silent --fail $VALIDATION_URL | grep -q "Token valid"; then
    # Token is valid, proceed with SSH login
    exec $SHELL
else
    echo "Invalid or expired token. Please check your email and click the link to authenticate."
    exit 1
fi
```

This script checks if the user clicked the link and whether the token is valid. If valid, it allows the user to proceed with the SSH session (`exec $SHELL`). If invalid, it denies access.

### 5. **Restart SSH Service**
Finally, restart the SSH service to apply the changes:

```bash
sudo systemctl restart sshd
```

### 6. **Testing**
- When a user tries to log in via SSH, the `ForceCommand` will trigger the `authentication_check.sh` script.
- The script will send an email with the unique authentication link.
- The user will need to click the link, which will verify their identity before allowing the SSH session.

### Caveats and Considerations
- **Security**: Tokens should be time-limited, and you should ensure that they are only valid for a short period of time (e.g., 10-15 minutes).
- **Scalability**: For a larger setup, using a database to store tokens and manage sessions might be more appropriate than using a flat file.
- **Error Handling**: Ensure proper error handling and logging in your scripts for better debugging and auditing.

This approach can be a good way to add an additional layer of authentication, but keep in mind that it introduces potential delays and complexity to the login process.



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

