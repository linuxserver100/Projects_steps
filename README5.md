
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

😯🥲🥱🤭🙂😢🙂😢🤭😢😐🙃😢😐🙃😢😐🙂😢🤭🙂😢🤭🙂😢🤭🙃😢🙃🤭😮🤭😥🤭🙃😦🙃😦🤭🙃🙃🤭😦🙃🤭🤭🙂😮🤭🙃🤭🤧😬🤧😬😢🤧😢🤫🤧😮🤫😢🤫🤧🤫😮🤧😯🤫🤧☹️🤫🤧☹️🤫🤒🤫☹️🤒☹️🫢☹️🤒☹️🤫🤒🙁🫢🤒🤒☹️😑🤒☹️😑🤒😑☹️🤒☹️😑🤒😑🤒☹️�🫢🤒🫢☹️🤒🫢☹️🤒☹️🤫🤒🙁😑🤒☹️😑🤒☹️🫢🤒☹️😑🤒🙁🤒🙁😑🤒🙁😑🙁🤒😑🤒☹️😑🤒☹️🫢🤒☹️🫢☹️🤒☹️🤫🥸☹️🫢🤒☹️🫢
To implement SMS-based SSH authentication using **Twilio** and **Bash** (without relying on Python), you can use Twilio’s API to send the SMS messages. The process involves creating an account with Twilio, setting up your server to interact with Twilio, and using `curl` to send the SMS.

### Prerequisites:
1. **Twilio Account**: You need a Twilio account. Create one at [Twilio](https://www.twilio.com/).
2. **Twilio API Credentials**: After signing up, get your **Account SID** and **Auth Token** from the [Twilio console](https://www.twilio.com/console).
3. **Twilio Phone Number**: Purchase a phone number from Twilio that can send SMS.
4. **SSH Access**: Basic SSH access set up on your server.

### Step-by-Step Implementation

#### Step 1: Install `curl` (if not already installed)
`curl` is used to make API requests to Twilio. Most Linux distributions have `curl` installed by default, but you can install it if it's missing:

```bash
sudo apt update
sudo apt install curl
```

#### Step 2: Configure Twilio for SMS
1. **Get Twilio API credentials**:
   - Sign up for a free Twilio account.
   - Get your **Account SID** and **Auth Token** from the Twilio console.
   - Buy a Twilio phone number that is capable of sending SMS (this is required for sending SMS).

#### Step 3: Create the SMS Sending Script Using Twilio

Create a script that will send an SMS using Twilio’s API.

```bash
sudo nano /usr/local/bin/send_sms.sh
```

Add the following content to the script, replacing the placeholders with your actual Twilio details.

```bash
#!/bin/bash

# Twilio Account SID, Auth Token, and Twilio phone number
TWILIO_SID="your_account_sid"
TWILIO_AUTH_TOKEN="your_auth_token"
TWILIO_PHONE_NUMBER="your_twilio_phone_number"

# The recipient's phone number
TO_NUMBER="$1"
# The SMS message content
MESSAGE="$2"

# Send the SMS via Twilio using curl
curl -X POST https://api.twilio.com/2010-04-01/Accounts/$TWILIO_SID/Messages.json \
--data-urlencode "To=$TO_NUMBER" \
--data-urlencode "From=$TWILIO_PHONE_NUMBER" \
--data-urlencode "Body=$MESSAGE" \
-u "$TWILIO_SID:$TWILIO_AUTH_TOKEN" > /dev/null 2>&1
```

Make the script executable:

```bash
sudo chmod +x /usr/local/bin/send_sms.sh
```

This script will send an SMS to the number provided (`$1` is the recipient's number) with the message provided (`$2` is the message content).

#### Step 4: Create the SSH Authentication Script
This script will handle the SMS-based authentication process. It will generate a random code, send it via SMS using the `send_sms.sh` script, and then prompt the user to enter the code.

```bash
sudo nano /usr/local/bin/authenticate.sh
```

Add the following content to the authentication script:

```bash
#!/bin/bash

# Replace this with the actual user's phone number
PHONE_NUMBER="user_phone_number"

# Generate a random 6-digit code
CODE=$(shuf -i 100000-999999 -n 1)

# Send the SMS code to the user using the send_sms.sh script
/usr/local/bin/send_sms.sh "$PHONE_NUMBER" "Your SSH authentication code is: $CODE"

# Prompt the user to enter the code they received
echo -n "Enter the SMS code: "
read INPUT_CODE

# Validate the code entered by the user
if [[ "$INPUT_CODE" == "$CODE" ]]; then
    echo "Authentication successful. Starting shell..."
    exec $SHELL
else
    echo "Authentication failed."
    exit 1
fi
```

Make the script executable:

```bash
sudo chmod +x /usr/local/bin/authenticate.sh
```

This script does the following:
1. It generates a random 6-digit code.
2. Sends the code to the specified phone number using Twilio.
3. Prompts the user to enter the code they received.
4. If the entered code matches the generated code, the user is authenticated and granted access to the shell.

#### Step 5: Modify SSH Configuration
Next, you’ll modify the SSH server configuration to use the authentication script.

Edit the SSH configuration:

```bash
sudo nano /etc/ssh/sshd_config
```

Add the following lines to force SSH to use the custom authentication script for a specific user:

```plaintext
Match User your_username
    ForceCommand /usr/local/bin/authenticate.sh
```

Replace `your_username` with the actual username you want to enable SMS authentication for.

#### Step 6: Restart SSH Service
After making these changes, restart the SSH service to apply the new configuration:

```bash
sudo systemctl restart ssh
```

#### Step 7: Testing the Setup
1. Try to SSH into the server as the specified user.
2. You will receive an SMS with a code.
3. Enter the code when prompted to authenticate and gain access.

### Important Considerations:
- **Security**: As mentioned, SMS-based authentication is not the most secure method because of vulnerabilities like SIM swapping. For higher security, consider using TOTP (Time-Based One-Time Password) or hardware-based authentication methods.
- **Twilio Costs**: Sending SMS via Twilio incurs costs, even if using a free account. Check Twilio’s pricing to ensure it fits your budget.
- **Error Handling**: You should add error handling to account for issues like network failures or API errors (e.g., Twilio service disruptions).
- **Phone Number Management**: It's better to store phone numbers securely, such as in a database or secure configuration file, rather than hardcoding them in the script.

### Conclusion
This solution uses Twilio to send SMS-based authentication codes for SSH login. It bypasses the need for complex dependencies like Python, relying solely on Bash and `curl` to interact with Twilio's API.


🫥🫥🥰🫥😊🫥😊🫥🫥😊🫥☺️🫥☺️🫥☺️🫥🫥☺️🫥☺️🫥☺️🫥🫥☺️🫥☺️🫥☺️🫥☺️🫥🫥☺️🫥😌🫥🫥🫥🫥😌🫥😌🫥😌🫥😌🫥😌🫥🫥😌🫥😌🫥😌🫥😌🫥😌🫥🫥😌🫥😌🫥😌🫥😌😌🫥😌🫥😌🫥🫥😌🫥😌🫥😌🫥😌🫥🫥😌🫥😌🫥😌🫥😌🫥🫥😌🫥😌🫥😌🫥😌🫥😌🫥😌🫥😌🫥🫥😌🫥😌🫥😌🫥😌🫥😌🫥😌🫥😌🫥🫥😌🫥😌🫥😌🫥😌🫥😌🫥🫥😌🫥😌🫥😌🫥😌🫥😌🫥😌🫥😌🫥😌🫥🫥😌🫥😌🫥😌🫥😌🫥😌🫥😌🫥🫥😌🫥😌🫥😌🫥😌🫥😌🫥🫥😌🫥😌🫥😌🫥😌🫥😌🫥😌😌🫥😌🫥🫥😌🫥😌🫥😌🫥😌🫥😌🫥😌
To implement phone SMS authentication for SSH without Twilio, you can use an alternative service like **Nexmo** (now called **Vonage**), which offers a similar API for sending SMS. Below is a revised implementation using Nexmo (Vonage).

### Requirements
1. **Vonage (Nexmo) Account**: Sign up for Vonage and obtain your API key, API secret, and a Vonage phone number.
2. **ssmtp**: For email functionality (optional, in case you wish to send notifications via email).
3. **SSH Server**: Ensure your server is running SSH (OpenSSH).
4. **Bash or Python**: A scripting language to create the custom authentication script.

### Step-by-Step Implementation

#### Step 1: Install Required Packages
You will need `curl` (if not already installed) to make HTTP requests:

```bash
sudo apt update
sudo apt install curl
```

#### Step 2: Create the SMS Sending Script
Create a script that sends an SMS using the Vonage API.

```bash
sudo nano /usr/local/bin/send_sms.sh
```

Add the following content to the script (replace placeholders with your Vonage credentials):

```bash
#!/bin/bash

# Vonage API Credentials
API_KEY="your_vonage_api_key"
API_SECRET="your_vonage_api_secret"
FROM_NUMBER="your_vonage_number"
TO_NUMBER="$1"
MESSAGE="$2"

# Send SMS using Vonage API
curl -X POST "https://rest.nexmo.com/sms/json" \
--data-urlencode "api_key=$API_KEY" \
--data-urlencode "api_secret=$API_SECRET" \
--data-urlencode "from=$FROM_NUMBER" \
--data-urlencode "to=$TO_NUMBER" \
--data-urlencode "text=$MESSAGE"
```

Make the script executable:

```bash
sudo chmod +x /usr/local/bin/send_sms.sh
```

#### Step 3: Set Up the Custom SSH Command
Edit the SSH configuration file to use `ForceCommand` and execute your custom authentication script.

```bash
sudo nano /etc/ssh/sshd_config
```

Add or modify the following lines to include `ForceCommand`:

```plaintext
Match User your_username
    ForceCommand /usr/local/bin/authenticate.sh
```

#### Step 4: Create the Authentication Script
Now, create the `authenticate.sh` script that will handle the SMS authentication.

```bash
sudo nano /usr/local/bin/authenticate.sh
```

Here’s an example of what this script might look like:

```bash
#!/bin/bash

# User's phone number
PHONE_NUMBER="user_phone_number"

# Generate a random 6-digit code
CODE=$(shuf -i 100000-999999 -n 1)

# Send the SMS code to the user
/usr/local/bin/send_sms.sh "$PHONE_NUMBER" "Your SSH authentication code is: $CODE"

# Prompt for the code from the user
echo -n "Enter the SMS code: "
read INPUT_CODE

# Validate the code entered by the user
if [[ "$INPUT_CODE" == "$CODE" ]]; then
    echo "Authentication successful. Starting shell..."
    exec $SHELL
else
    echo "Authentication failed."
    exit 1
fi
```

Make this script executable:

```bash
sudo chmod +x /usr/local/bin/authenticate.sh
```

#### Step 5: Restart SSH Service
After making these changes, restart the SSH service to apply the new configuration:

```bash
sudo systemctl restart ssh
```

### Step 6: Testing
1. Attempt to SSH into the server as the specified user.
2. You should receive an SMS with the authentication code.
3. Enter the code when prompted to gain access.

### Important Considerations
- **Security**: SMS-based authentication is inherently less secure than other methods, such as time-based one-time passwords (TOTP) or hardware tokens. It's vulnerable to SIM swapping attacks, interception, etc.
- **Rate Limiting**: To avoid abuse, implement rate limiting in your script to restrict the number of SMS messages sent in a short time.
- **Logging**: It’s recommended to log all access attempts and failures for audit purposes.

### Additional Enhancements
- **Database or File Storage**: You can enhance the security by storing phone numbers in a secure database or file, rather than hardcoding them in scripts.
- **Error Handling**: Add more robust error handling to account for network failures, invalid API responses, or other issues.
- **Environment Variables**: Instead of hardcoding sensitive information like the API key, secret, and phone number, store them in environment variables or a configuration file.

### Conclusion
This solution provides a basic implementation for adding SMS-based two-factor authentication to SSH using the Vonage API. While it is a simple and straightforward method, remember that SMS-based authentication can be vulnerable to various attacks. For higher security, you may want to consider using more robust authentication methods such as TOTP (Time-based One-Time Passwords) or hardware security keys like YubiKey.


😢😙😢😙😐😙😛😮‍💨😗😛😢😛😮‍💨😛😗😢😐😗😢😐😗😢😐😮‍💨😐😗😢😗😢😐😗😢😐😗😢😐😗😢😐😗😢😐😗😢😶🙃☹️😶☹️☹️😶😗☹️😶🙃☹️😶😗☹️😶😗☹️😶🙃☹️😶😗☹️😗🫠☹️😝😗☹️😶😗☹️😶😗😯🙃☹️😶😗😢😐😗☹️🙃🫠☹️😝😗😮‍💨😮‍💨😛😗😮‍💨😝😗😤😛😃😮‍💨😃😤😝😃😮‍💨
.
To .To configure your system to send an SMS when an SSH login is triggered, and only allow the user to proceed after verifying the SMS code, we need to update the scripts accordingly. Below is a complete guide with each step reset, updated, and more clearly explained.

### Prerequisites:
1. **Gammu**: Used to send SMS via a GSM device (e.g., USB dongle or mobile phone).
2. **Cron Jobs**: To handle periodic tasks (though we will focus on SSH-based triggering in this case).
3. **SSH Configuration**: SSH is configured to require a verification step (via SMS) before shell access is granted.
4. **`ForceCommand`**: To ensure SSH sessions are restricted to your custom SMS verification script.

---

### Step-by-Step Guide:

### Part 1: Set up SMS Sending with Gammu

#### Step 1: Install Gammu

To send SMS from your Linux server, you need to install **Gammu**. This tool interacts with your GSM device and sends the SMS messages.

```bash
sudo apt update
sudo apt install gammu gammu-smsd
```

#### Step 2: Configure Gammu

You need to configure Gammu to interface with your GSM device (e.g., USB modem or mobile phone). This typically involves creating a `gammu` configuration file. You can find the configuration steps in the [Gammu documentation](https://wammu.eu/docs/).

Run the following command to generate a `gammurc` configuration file:

```bash
gammu-config
```

This will guide you through the setup to ensure Gammu is able to send SMS messages.

#### Step 3: Create a Send SMS Script

Create a script that will send an SMS using **Gammu**. The script should accept a phone number and a message as parameters.

```bash
sudo nano /usr/local/bin/send_sms.sh
```

Add the following content:

```bash
#!/bin/bash

# First parameter: Phone number to send SMS to
TO_NUMBER="$1"
# Second parameter: Message to send
MESSAGE="$2"

# Send the SMS using Gammu
gammu sendsms TEXT "$TO_NUMBER" -text "$MESSAGE"
```

Make this script executable:

```bash
sudo chmod +x /usr/local/bin/send_sms.sh
```

This script will allow you to send SMS messages via Gammu.

---

### Part 2: Force SMS Verification on SSH Login

The key step here is using `ForceCommand` in the SSH configuration to make sure that SSH logins are forced to execute a script (which will handle SMS verification) before a user gets shell access.

#### Step 1: Create an SMS Authentication Script

This script will trigger an SMS message with a verification code when an SSH login occurs, and it will ask the user to enter the code before proceeding.

```bash
sudo nano /usr/local/bin/ssh_sms_auth.sh
```

Add the following content:

```bash
#!/bin/bash

# Define the phone number (change this to your desired phone number)
PHONE_NUMBER="your_phone_number"  # Replace with the actual phone number

# Generate a random 6-digit code for SMS authentication
CODE=$(shuf -i 100000-999999 -n 1)

# Send the authentication code to the user's phone number
/usr/local/bin/send_sms.sh "$PHONE_NUMBER" "Your SSH authentication code is: $CODE"

# Prompt the user to enter the code
echo -n "Enter the SMS code: "
read INPUT_CODE

# Validate the entered code against the generated code
if [[ "$INPUT_CODE" == "$CODE" ]]; then
    echo "Authentication successful. Granting shell access..."
    exec $SHELL  # Allow the user to continue with shell access
else
    echo "Authentication failed. Please try again."
    exit 1  # Deny access if the code doesn't match
fi
```

Make this script executable:

```bash
sudo chmod +x /usr/local/bin/ssh_sms_auth.sh
```

This script does the following:
1. It generates a random 6-digit authentication code.
2. Sends the code via SMS to the user’s phone number.
3. Prompts the user to enter the code.
4. Verifies the code. If correct, it allows shell access; otherwise, it denies access.

#### Step 2: Configure SSH to Use the SMS Script

Now, we need to configure SSH to enforce this SMS authentication script upon login.

Edit the SSH configuration file (`/etc/ssh/sshd_config`):

```bash
sudo nano /etc/ssh/sshd_config
```

Add or modify the following line:

```bash
ForceCommand /usr/local/bin/ssh_sms_auth.sh
```

This directive forces every SSH session to run `/usr/local/bin/ssh_sms_auth.sh` instead of a regular shell. Therefore, users must pass the SMS authentication before being granted shell access.

#### Step 3: Restart SSH Service

After modifying the SSH configuration, restart the SSH service to apply the changes:

```bash
sudo systemctl restart ssh
```

Now, whenever someone logs in via SSH, they will be forced to run the SMS authentication script.

---

### Part 3: Test the SSH Login and SMS Verification

1. **Login via SSH**: Try logging in to the server via SSH from another terminal or machine.

```bash
ssh your_user@your_server_ip
```

2. **SMS Code**: You should receive an SMS with a 6-digit authentication code on the phone number you configured in `/usr/local/bin/ssh_sms_auth.sh`.

3. **Enter the Code**: Enter the 6-digit code when prompted. If the code is correct, you will gain shell access.

   If the code is incorrect, the system will deny access.

---

### Troubleshooting

- **No SMS received**: Ensure that Gammu is properly configured and can send SMS. You can test sending an SMS manually using:

  ```bash
  gammu sendsms TEXT "your_phone_number" -text "Test message"
  ```

- **SSH not working as expected**: If users can't log in after these changes, check `/var/log/auth.log` for SSH errors, and ensure that the `ForceCommand` directive is correctly set.

- **Permissions**: Ensure that the scripts `/usr/local/bin/ssh_sms_auth.sh` and `/usr/local/bin/send_sms.sh` are executable and have the right permissions.

---

### Conclusion

In this setup, we've configured the server to send an SMS message with a verification code whenever an SSH login is triggered. The user must enter the code to gain access to the shell. By using `ForceCommand`, we’ve ensured that the login process is restricted to running only the SMS authentication script, enhancing security.


🥰🫥🥰🫥🥰🫥🥰🫥🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰☺️😊🫥😊🫥😊🫥🥰🫥🫥🥰🫥😊🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥😊🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥🥰🫥
Sure! Here's a revised version of your SSH authentication setup with **SMS verification** using **Textbelt** and **ForceCommand** to enforce the authentication process before allowing access to the shell. I'll walk you through the full setup, with the necessary steps and command details:

---

### **Setting Up SSH Authentication with SMS Verification (Using ForceCommand)**

In this updated guide, we’ll configure an SSH login system that requires SMS verification via **Textbelt**, a free SMS service. We will use `ForceCommand` in SSH to ensure the system only proceeds with a login after SMS verification.

---

### **Prerequisites**:

1. **Curl**: For sending HTTP requests to the Textbelt API.
2. **SSH**: A properly configured SSH server that will enforce SMS-based login via `ForceCommand`.
3. **Textbelt API**: A free SMS API to send the verification code.
4. **Linux Server**: A server running Ubuntu (or a similar Linux distribution).

---

### **Part 1: Setting Up the SMS Sending Script**

#### Step 1: Install `curl`

First, make sure you have `curl` installed to interact with Textbelt’s API.

```bash
sudo apt update
sudo apt install curl
```

#### Step 2: Create a Script to Send SMS

Now, we will create a script that sends an SMS via the Textbelt API when called.

Create the script:

```bash
sudo nano /usr/local/bin/send_sms.sh
```

Add the following content to the script:

```bash
#!/bin/bash

# First parameter: Phone number to send SMS to
TO_NUMBER="$1"
# Second parameter: Message to send
MESSAGE="$2"

# Textbelt API endpoint
TEXTBELT_API="https://textbelt.com/text"

# Your Textbelt API key (use 'textbelt' for the free key, or replace it with a paid key)
API_KEY="textbelt"  # 'textbelt' is the free API key for limited use

# Send SMS via Textbelt API
curl -X POST "$TEXTBELT_API" \
    -d phone="$TO_NUMBER" \
    -d message="$MESSAGE" \
    -d key="$API_KEY"
```

Make this script executable:

```bash
sudo chmod +x /usr/local/bin/send_sms.sh
```

This script sends an SMS using the Textbelt API to a specified phone number with a custom message.

---

### **Part 2: Create the SMS Authentication Script for SSH Login**

#### Step 1: Create the Authentication Script

Next, we create the script that will handle the SMS verification process when a user attempts to log in via SSH.

```bash
sudo nano /usr/local/bin/ssh_sms_auth.sh
```

Add the following content to the script:

```bash
#!/bin/bash

# Define the phone number for SMS authentication (replace with your own number)
PHONE_NUMBER="your_phone_number"  # Example: "+1234567890"

# Generate a random 6-digit authentication code
CODE=$(shuf -i 100000-999999 -n 1)

# Send the authentication code via SMS
/usr/local/bin/send_sms.sh "$PHONE_NUMBER" "Your SSH authentication code is: $CODE"

# Prompt the user to enter the code
echo -n "Enter the SMS code sent to your phone: "
read INPUT_CODE

# Validate the entered code
if [[ "$INPUT_CODE" == "$CODE" ]]; then
    echo "Authentication successful. Granting shell access..."
    exec $SHELL  # Allow the user to continue with shell access
else
    echo "Authentication failed. Please try again."
    exit 1  # Deny access if the code doesn't match
fi
```

Make the script executable:

```bash
sudo chmod +x /usr/local/bin/ssh_sms_auth.sh
```

This script works as follows:
- **Generates a random 6-digit code**.
- **Sends the code via SMS** to the specified phone number.
- **Prompts the user** to enter the code they received.
- If the entered code matches the sent code, the user gains shell access. Otherwise, access is denied.

---

### **Part 3: Configure SSH to Use the SMS Authentication Script**

#### Step 1: Modify the SSH Configuration

Now, we need to configure SSH to **force the execution** of the SMS authentication script on every login attempt using the `ForceCommand` directive.

Open the SSH configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```

Add the following line to the configuration file:

```bash
ForceCommand /usr/local/bin/ssh_sms_auth.sh
```

This line ensures that every SSH login attempt is forced to run the `/usr/local/bin/ssh_sms_auth.sh` script. Without passing the SMS verification, users will not be granted shell access.

#### Step 2: Restart SSH Service

After making changes to the `sshd_config` file, you need to restart the SSH service to apply the changes:

```bash
sudo systemctl restart ssh
```

Now, SSH will invoke the SMS authentication script every time a login is attempted.

---

### **Part 4: Test the SSH Login and SMS Verification**

1. **Login via SSH**: Attempt to log in to the server from a terminal or another machine:

   ```bash
   ssh your_user@your_server_ip
   ```

2. **Receive SMS**: You should receive an SMS with a 6-digit code sent to the phone number defined in the `/usr/local/bin/ssh_sms_auth.sh` script.

3. **Enter the Code**: Enter the code when prompted. If the code is correct, you’ll gain access to the shell. If the code is incorrect, access will be denied.

---

### **Troubleshooting**

1. **No SMS Received**:
   - Ensure that the `curl` command is working correctly. You can manually test the SMS script by running:

     ```bash
     /usr/local/bin/send_sms.sh "+1234567890" "Test message"
     ```

2. **SSH Login Issues**:
   - If you can’t log in, check if the `ForceCommand` directive is properly set in the `/etc/ssh/sshd_config` file. Ensure the path to the script is correct and that the script is executable (`chmod +x /usr/local/bin/ssh_sms_auth.sh`).

3. **Textbelt API Limits**:
   - Textbelt has limitations on free usage. If you reach these limits, you may need to switch to a paid service like **Twilio** or **Nexmo** for a larger volume of SMS messages.

4. **Ensure Correct Permissions**:
   - Check if the script has proper permissions to execute. Ensure the script is owned by root and executable:

     ```bash
     sudo chmod +x /usr/local/bin/send_sms.sh
     sudo chmod +x /usr/local/bin/ssh_sms_auth.sh
     ```

---

### **Conclusion**

This setup ensures that **SMS verification** is enforced for every SSH login, enhancing the security of your server. The **`ForceCommand`** directive in SSH ensures that users cannot bypass the SMS authentication process, making the server more secure.

Using **Textbelt** for sending SMS is a cost-effective way to implement this, though keep in mind that the free service has limitations. For more extensive use, consider using paid SMS providers like **Twilio** or **Nexmo** for more reliability and higher SMS limits.

🫥🫥🙃🫥😚🫥😚🫥🫥😚🫥😚🫥😚🫥😚🫥😚🫥🥳🫥🫥🥳🫥🥳🫥🥳🫥😚🫥😚🫥🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥🫥😚😚🫥😚🫥🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥🫥😚🫥😚🫥😚🫥😚🫥😚😚🫥😚🫥😚🫥😚🫥🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚🫥😚
To modify the existing SSH authentication setup to work with **Fast2SMS** instead of **Textbelt**, you will need to adjust the SMS sending script and use Fast2SMS's API for sending the SMS.

Here’s the updated version of the setup with Fast2SMS:

---

### **Setting Up SSH Authentication with SMS Verification (Using Fast2SMS)**

This guide configures an SSH login system that requires SMS verification via **Fast2SMS**, a popular SMS service provider in India. We will use `ForceCommand` in SSH to enforce the authentication process before allowing access to the shell.

---

### **Prerequisites**:

1. **Curl**: For sending HTTP requests to the Fast2SMS API.
2. **SSH**: A properly configured SSH server that will enforce SMS-based login via `ForceCommand`.
3. **Fast2SMS API**: Fast2SMS account with API access to send SMS.
4. **Linux Server**: A server running Ubuntu (or a similar Linux distribution).

---

### **Part 1: Setting Up the SMS Sending Script with Fast2SMS**

#### Step 1: Install `curl`

Ensure `curl` is installed to interact with the Fast2SMS API.

```bash
sudo apt update
sudo apt install curl
```

#### Step 2: Create a Script to Send SMS

Create the script that will send an SMS via the Fast2SMS API.

```bash
sudo nano /usr/local/bin/send_sms.sh
```

Add the following content to the script:

```bash
#!/bin/bash

# First parameter: Phone number to send SMS to
TO_NUMBER="$1"
# Second parameter: Message to send
MESSAGE="$2"

# Fast2SMS API endpoint
FAST2SMS_API="https://www.fast2sms.com/dev/bulk"

# Your Fast2SMS API key (replace with your actual API key)
API_KEY="your_fast2sms_api_key"  # Replace with your Fast2SMS API key

# Send SMS via Fast2SMS API
curl -X POST "$FAST2SMS_API" \
    -d sender_id="FSTSMS" \
    -d message="$MESSAGE" \
    -d language="english" \
    -d route="p" \
    -d numbers="$TO_NUMBER" \
    -d api_key="$API_KEY"
```

Make this script executable:

```bash
sudo chmod +x /usr/local/bin/send_sms.sh
```

This script sends an SMS using the Fast2SMS API to a specified phone number with a custom message.

---

### **Part 2: Create the SMS Authentication Script for SSH Login**

#### Step 1: Create the Authentication Script

Next, we will create the script that handles SMS verification when a user attempts to log in via SSH.

```bash
sudo nano /usr/local/bin/ssh_sms_auth.sh
```

Add the following content to the script:

```bash
#!/bin/bash

# Define the phone number for SMS authentication (replace with your own number)
PHONE_NUMBER="your_phone_number"  # Example: "+1234567890"

# Generate a random 6-digit authentication code
CODE=$(shuf -i 100000-999999 -n 1)

# Send the authentication code via SMS
/usr/local/bin/send_sms.sh "$PHONE_NUMBER" "Your SSH authentication code is: $CODE"

# Prompt the user to enter the code
echo -n "Enter the SMS code sent to your phone: "
read INPUT_CODE

# Validate the entered code
if [[ "$INPUT_CODE" == "$CODE" ]]; then
    echo "Authentication successful. Granting shell access..."
    exec $SHELL  # Allow the user to continue with shell access
else
    echo "Authentication failed. Please try again."
    exit 1  # Deny access if the code doesn't match
fi
```

Make the script executable:

```bash
sudo chmod +x /usr/local/bin/ssh_sms_auth.sh
```

This script:
- **Generates a random 6-digit code**.
- **Sends the code via SMS** to the phone number defined in the script.
- **Prompts the user** to enter the code.
- If the entered code matches the sent code, the user gains shell access; otherwise, access is denied.

---

### **Part 3: Configure SSH to Use the SMS Authentication Script**

#### Step 1: Modify the SSH Configuration

Now, we need to configure SSH to **force the execution** of the SMS authentication script on every login attempt using the `ForceCommand` directive.

Open the SSH configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```

Add the following line to the configuration file:

```bash
ForceCommand /usr/local/bin/ssh_sms_auth.sh
```

This line ensures that every SSH login attempt is forced to run the `/usr/local/bin/ssh_sms_auth.sh` script. Without passing the SMS verification, users will not be granted shell access.

#### Step 2: Restart SSH Service

After making changes to the `sshd_config` file, restart the SSH service to apply the changes:

```bash
sudo systemctl restart ssh
```

Now, SSH will invoke the SMS authentication script every time a login is attempted.

---

### **Part 4: Test the SSH Login and SMS Verification**

1. **Login via SSH**: Attempt to log in to the server from a terminal or another machine:

   ```bash
   ssh your_user@your_server_ip
   ```

2. **Receive SMS**: You should receive an SMS with a 6-digit code sent to the phone number defined in the `/usr/local/bin/ssh_sms_auth.sh` script.

3. **Enter the Code**: Enter the code when prompted. If the code is correct, you’ll gain access to the shell. If the code is incorrect, access will be denied.

---

### **Troubleshooting**

1. **No SMS Received**:
   - Ensure that the `curl` command is working correctly. You can manually test the SMS script by running:

     ```bash
     /usr/local/bin/send_sms.sh "+1234567890" "Test message"
     ```

2. **SSH Login Issues**:
   - If you can’t log in, check if the `ForceCommand` directive is properly set in the `/etc/ssh/sshd_config` file. Ensure the path to the script is correct and that the script is executable (`chmod +x /usr/local/bin/ssh_sms_auth.sh`).

3. **Fast2SMS API Limits**:
   - Fast2SMS has usage limits on free accounts. If you exceed those limits, you may need to upgrade to a paid plan for more SMS credits.

4. **Ensure Correct Permissions**:
   - Ensure the script has proper permissions to execute. Check that the script is owned by root and executable:

     ```bash
     sudo chmod +x /usr/local/bin/send_sms.sh
     sudo chmod +x /usr/local/bin/ssh_sms_auth.sh
     ```

---

### **Conclusion**

This setup ensures that **SMS verification** is enforced for every SSH login, enhancing the security of your server. The **`ForceCommand`** directive in SSH ensures that users cannot bypass the SMS authentication process, making the server more secure.

Using **Fast2SMS** for sending SMS is an effective solution, though keep in mind that free accounts come with some limitations. For higher usage, consider upgrading your Fast2SMS plan or using alternative SMS providers like **Twilio** or **Nexmo**.

---

By following the steps above, you can successfully implement SMS-based SSH authentication with Fast2SMS. Let me know if you need any further assistance with this!

🫥😊🫥😊🫥🫥😊🫥😊🫥😊🫥😊🫥😊😊🫥🫥😊🫥😊🫥☺️🫥☺️🫥🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥☺️🫥🫥☺️🫥☺️🫥☺️To modify the SSH authentication setup to use **SMS Gateway Hub** instead of **Fast2SMS**, we'll follow similar steps but adjust the API calls to match the **SMS Gateway Hub** API format. Below is the updated version of the script and configuration for SMS authentication with SSH using SMS Gateway Hub.

---

### **Setting Up SSH Authentication with SMS Verification (Using SMS Gateway Hub)**

This guide configures an SSH login system that requires SMS verification via **SMS Gateway Hub**, a popular SMS service provider.

---

### **Prerequisites**:

1. **Curl**: For sending HTTP requests to the SMS Gateway Hub API.
2. **SSH**: A properly configured SSH server that will enforce SMS-based login via `ForceCommand`.
3. **SMS Gateway Hub API**: SMS Gateway Hub account with API access to send SMS.
4. **Linux Server**: A server running Ubuntu (or a similar Linux distribution).

---

### **Part 1: Setting Up the SMS Sending Script with SMS Gateway Hub**

#### Step 1: Install `curl`

Ensure `curl` is installed to interact with the SMS Gateway Hub API.

```bash
sudo apt update
sudo apt install curl
```

#### Step 2: Create a Script to Send SMS

Create the script that will send an SMS via the SMS Gateway Hub API.

```bash
sudo nano /usr/local/bin/send_sms.sh
```

Add the following content to the script:

```bash
#!/bin/bash

# First parameter: Phone number to send SMS to
TO_NUMBER="$1"
# Second parameter: Message to send
MESSAGE="$2"

# SMS Gateway Hub API endpoint
SMSGATEWAYHUB_API="https://www.smsgatewayhub.com/api/mt/SendSMS"

# Your SMS Gateway Hub API credentials
API_KEY="your_sms_gateway_hub_api_key"  # Replace with your API key
SENDER_ID="your_sender_id"  # Replace with your sender ID (like "FSTSMS" or your custom sender ID)

# Send SMS via SMS Gateway Hub API
curl -X POST "$SMSGATEWAYHUB_API" \
    -d "user=$API_KEY" \
    -d "password=your_password" \  # Replace with your API password
    -d "senderid=$SENDER_ID" \
    -d "channel=2" \
    -d "DCS=0" \
    -d "flashsms=0" \
    -d "number=$TO_NUMBER" \
    -d "message=$MESSAGE" \
    -d "route=4"
```

Make this script executable:

```bash
sudo chmod +x /usr/local/bin/send_sms.sh
```

This script sends an SMS using the SMS Gateway Hub API to a specified phone number with a custom message.

---

### **Part 2: Create the SMS Authentication Script for SSH Login**

#### Step 1: Create the Authentication Script

Next, we will create the script that handles SMS verification when a user attempts to log in via SSH.

```bash
sudo nano /usr/local/bin/ssh_sms_auth.sh
```

Add the following content to the script:

```bash
#!/bin/bash

# Define the phone number for SMS authentication (replace with your own number)
PHONE_NUMBER="your_phone_number"  # Example: "+1234567890"

# Generate a random 6-digit authentication code
CODE=$(shuf -i 100000-999999 -n 1)

# Send the authentication code via SMS
/usr/local/bin/send_sms.sh "$PHONE_NUMBER" "Your SSH authentication code is: $CODE"

# Prompt the user to enter the code
echo -n "Enter the SMS code sent to your phone: "
read INPUT_CODE

# Validate the entered code
if [[ "$INPUT_CODE" == "$CODE" ]]; then
    echo "Authentication successful. Granting shell access..."
    exec $SHELL  # Allow the user to continue with shell access
else
    echo "Authentication failed. Please try again."
    exit 1  # Deny access if the code doesn't match
fi
```

Make the script executable:

```bash
sudo chmod +x /usr/local/bin/ssh_sms_auth.sh
```

This script:
- **Generates a random 6-digit code**.
- **Sends the code via SMS** to the phone number defined in the script.
- **Prompts the user** to enter the code.
- If the entered code matches the sent code, the user gains shell access; otherwise, access is denied.

---

### **Part 3: Configure SSH to Use the SMS Authentication Script**

#### Step 1: Modify the SSH Configuration

Now, we need to configure SSH to **force the execution** of the SMS authentication script on every login attempt using the `ForceCommand` directive.

Open the SSH configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```

Add the following line to the configuration file:

```bash
ForceCommand /usr/local/bin/ssh_sms_auth.sh
```

This line ensures that every SSH login attempt is forced to run the `/usr/local/bin/ssh_sms_auth.sh` script. Without passing the SMS verification, users will not be granted shell access.

#### Step 2: Restart SSH Service

After making changes to the `sshd_config` file, restart the SSH service to apply the changes:

```bash
sudo systemctl restart ssh
```

Now, SSH will invoke the SMS authentication script every time a login is attempted.

---

### **Part 4: Test the SSH Login and SMS Verification**

1. **Login via SSH**: Attempt to log in to the server from a terminal or another machine:

   ```bash
   ssh your_user@your_server_ip
   ```

2. **Receive SMS**: You should receive an SMS with a 6-digit code sent to the phone number defined in the `/usr/local/bin/ssh_sms_auth.sh` script.

3. **Enter the Code**: Enter the code when prompted. If the code is correct, you’ll gain access to the shell. If the code is incorrect, access will be denied.

---

### **Troubleshooting**

1. **No SMS Received**:
   - Ensure that the `curl` command is working correctly. You can manually test the SMS script by running:

     ```bash
     /usr/local/bin/send_sms.sh "+1234567890" "Test message"
     ```

2. **SSH Login Issues**:
   - If you can’t log in, check if the `ForceCommand` directive is properly set in the `/etc/ssh/sshd_config` file. Ensure the path to the script is correct and that the script is executable (`chmod +x /usr/local/bin/ssh_sms_auth.sh`).

3. **SMS Gateway Hub API Limits**:
   - SMS Gateway Hub has usage limits. If you exceed those limits, you may need to upgrade to a paid plan for more SMS credits.

4. **Ensure Correct Permissions**:
   - Ensure the script has proper permissions to execute. Check that the script is owned by root and executable:

     ```bash
     sudo chmod +x /usr/local/bin/send_sms.sh
     sudo chmod +x /usr/local/bin/ssh_sms_auth.sh
     ```

---

### **Conclusion**

This setup ensures that **SMS verification** is enforced for every SSH login, enhancing the security of your server. The **`ForceCommand`** directive in SSH ensures that users cannot bypass the SMS authentication process, making the server more secure.

Using **SMS Gateway Hub** for sending SMS is an effective solution. For higher usage, consider upgrading your SMS Gateway Hub plan.

---

By following the steps above, you can successfully implement SMS-based SSH authentication with SMS Gateway Hub. Let me know if you need any further assistance with this!






