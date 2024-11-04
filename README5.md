
Setting up SSH login on an Ubuntu server using email OTP (One-Time Password) authentication can be a secure way to manage access. Below, I’ll outline the steps to accomplish this, including configuring SSH to force OTP authentication, using only standard Python libraries for SMTP email configuration, and setting up `ForceCommand` in the SSH configuration to enforce this.

### Steps

1. **Install Dependencies**:
   Ensure Python is installed. For Ubuntu, Python should already be present. You might also need `ssmtp` or `msmtp` for simpler SMTP handling (optional but useful).

   ```bash
   sudo apt update
   sudo apt install -y ssmtp  # or msmtp
   ```

2. **Configure SMTP**:
   Update your SMTP configuration to send email via a reliable service (like Gmail or your SMTP server). For example, with `ssmtp`, you could configure it at `/etc/ssmtp/ssmtp.conf`.

   Sample `ssmtp.conf` for Gmail:
   ```plaintext
   root=your-email@gmail.com
   mailhub=smtp.gmail.com:587
   AuthUser=your-email@gmail.com
   AuthPass=your-email-password
   UseSTARTTLS=YES
   ```

   If you don’t want to install extra SMTP utilities, you can also use Python’s built-in `smtplib` to send emails.

3. **Create OTP Script**:
   Write a Python script that generates and sends the OTP via email whenever someone tries to SSH into the server.

   Here’s a script (`/usr/local/bin/otp_auth.py`):

   ```python
   #!/usr/bin/env python3
   import smtplib
   import random
   import os
   import sys
   from email.message import EmailMessage

   # Configuration
   OTP_EMAIL = "your-email@gmail.com"  # The email to receive OTPs
   OTP_LENGTH = 6                      # Length of OTP

   def generate_otp(length=OTP_LENGTH):
       """Generate a random OTP."""
       return ''.join([str(random.randint(0, 9)) for _ in range(length)])

   def send_email(subject, body, to_email):
       """Send email with OTP."""
       msg = EmailMessage()
       msg.set_content(body)
       msg['Subject'] = subject
       msg['From'] = OTP_EMAIL
       msg['To'] = to_email

       with smtplib.SMTP("smtp.gmail.com", 587) as smtp:
           smtp.starttls()
           smtp.login(OTP_EMAIL, "your-email-password")
           smtp.send_message(msg)

   # Generate and send OTP
   otp = generate_otp()
   send_email("Your SSH OTP", f"Your OTP is: {otp}", OTP_EMAIL)

   # Save OTP for validation
   with open("/tmp/otp_code.txt", "w") as f:
       f.write(otp)

   print("An OTP has been sent to your email.")
   ```

   **Note:** Replace `your-email@gmail.com` and `your-email-password` with your actual email and password.

4. **Set up OTP Validation Script**:
   Create another script to check the OTP that the user enters. Place this script in a secure location, such as `/usr/local/bin/validate_otp.py`.

   ```python
   #!/usr/bin/env python3
   import sys

   # Read OTP from user
   user_otp = input("Enter OTP sent to your email: ")

   # Read the correct OTP
   try:
       with open("/tmp/otp_code.txt", "r") as f:
           correct_otp = f.read().strip()
   except FileNotFoundError:
       print("No OTP found. Please request a new OTP.")
       sys.exit(1)

   # Validate OTP
   if user_otp == correct_otp:
       print("Access granted.")
       os.remove("/tmp/otp_code.txt")  # Clean up after success
       sys.exit(0)  # Success
   else:
       print("Invalid OTP.")
       sys.exit(1)  # Failure
   ```

5. **Configure SSH to Use OTP Validation**:
   Configure SSH to run the OTP authentication script as a forced command. Open the SSH configuration file for editing:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

   Add the following at the end of the file to enforce OTP validation on login:

   ```plaintext
   Match User your-username
       ForceCommand /usr/local/bin/otp_auth.py && /usr/local/bin/validate_otp.py
   ```

   **Note**: Replace `your-username` with the specific username you want to require OTP for.

6. **Set Permissions and Make Scripts Executable**:
   Make sure the scripts are executable and properly secured.

   ```bash
   sudo chmod +x /usr/local/bin/otp_auth.py
   sudo chmod +x /usr/local/bin/validate_otp.py
   ```

7. **Restart SSH Service**:
   Restart the SSH service for changes to take effect.

   ```bash
   sudo systemctl restart ssh
   ```

8. **Testing**:
   Now, try logging in via SSH. You should receive an OTP email, and then you’ll be prompted to enter it. Access will be granted only if the correct OTP is provided.

### Important Notes

- Ensure the email and password used in the Python script are kept secure. Consider using environment variables or encrypted methods to store sensitive information.
- Adjust firewall and security rules accordingly to prevent unauthorized access.
- This setup is basic and can be further secured by adding logging, two-factor auth libraries, or additional configurations.

This approach should create a relatively secure SSH login using OTP without needing additional Python packages beyond the standard library.
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

