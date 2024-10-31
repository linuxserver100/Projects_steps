
Setting up an Ubuntu SSH server that requires email verification for user logins involves several steps, including configuring the SSH server, installing necessary packages, setting up email verification, and securing the server with Apache. Below is a detailed guide to accomplish this task.

### Prerequisites
1. **Ubuntu Server**: Ensure you have an Ubuntu server installed and accessible.
2. **Root Access**: You will need root or sudo privileges.
3. **Domain Name**: Have a domain name ready for the Apache configuration.
4. **Email Server**: Set up an email server or use a service like SendGrid or Mailgun for sending verification emails.

### Step 1: Install SSH Server
1. **Update the Package Index**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Install OpenSSH Server**:
   ```bash
   sudo apt install openssh-server -y
   ```

3. **Check SSH Service**:
   ```bash
   sudo systemctl status ssh
   ```

### Step 2: Install Required Packages
Install the required packages for email sending and web server functionality:
```bash
sudo apt install apache2 php libapache2-mod-php php-mysql sendmail -y
```

### Step 3: Configure Apache
1. **Enable Required Apache Modules**:
   ```bash
   sudo a2enmod rewrite
   sudo systemctl restart apache2
   ```

2. **Set Up a Virtual Host**:
   Create a new configuration file for your domain:
   ```bash
   sudo nano /etc/apache2/sites-available/yourdomain.com.conf
   ```
   Add the following configuration:
   ```apache
   <VirtualHost *:80>
       ServerAdmin admin@yourdomain.com
       ServerName yourdomain.com
       DocumentRoot /var/www/yourdomain.com/public_html
       <Directory /var/www/yourdomain.com/public_html>
           AllowOverride All
       </Directory>
       ErrorLog ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```

3. **Create the Document Root**:
   ```bash
   sudo mkdir -p /var/www/yourdomain.com/public_html
   ```

4. **Enable the New Virtual Host**:
   ```bash
   sudo a2ensite yourdomain.com.conf
   sudo systemctl restart apache2
   ```

### Step 4: Create the Email Verification Script
1. **Create the PHP Verification Script**:
   Create a new PHP file for handling verification.
   ```bash
   sudo nano /var/www/yourdomain.com/public_html/verify.php
   ```

   Add the following code to send an email with a verification link:
   ```php
   <?php
   // Change these variables according to your setup
   $to = "user@example.com"; // Email address to send the verification link
   $subject = "Login Verification";
   $verification_link = "https://yourdomain.com/verify_user.php?token=your_token"; // Generate token dynamically

   $message = "Please verify your login by clicking the link: " . $verification_link;
   $headers = "From: admin@yourdomain.com\r\n";

   mail($to, $subject, $message, $headers);
   ?>
   ```

### Step 5: Generate Verification Tokens
You will need a way to generate and store verification tokens securely. You can use a database or a file to manage this. Here’s an example using a database.

1. **Install MySQL**:
   ```bash
   sudo apt install mysql-server -y
   ```

2. **Create a Database**:
   ```bash
   sudo mysql -u root -p
   ```
   ```sql
   CREATE DATABASE user_verification;
   USE user_verification;
   CREATE TABLE tokens (
       id INT AUTO_INCREMENT PRIMARY KEY,
       email VARCHAR(255) NOT NULL,
       token VARCHAR(255) NOT NULL,
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   ```

3. **Modify the PHP Script**:
   Update the PHP script to store tokens in the database.

### Step 6: Configure SSH for Email Verification
1. **Create a Custom SSH Login Script**:
   Create a script to handle login attempts and trigger email verification.
   ```bash
   sudo nano /usr/local/bin/ssh_login_verification.sh
   ```

   Add logic to send an email verification when a login attempt occurs.

2. **Modify the SSH Configuration**:
   Open the SSH configuration file:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

   Add or modify the following lines:
   ```bash
   ForceCommand /usr/local/bin/ssh_login_verification.sh
   ```

3. **Restart SSH**:
   ```bash
   sudo systemctl restart ssh
   ```

### Step 7: Test the Configuration
1. **Test SSH Login**: Attempt to log in to the server via SSH. You should receive an email with a verification link.

2. **Verify Email Link**: Clicking the verification link should complete the login process.

### Step 8: Secure Your Server
1. **Enable UFW Firewall**:
   ```bash
   sudo ufw allow OpenSSH
   sudo ufw allow 'Apache Full'
   sudo ufw enable
   ```

2. **Secure SSH Configuration**:
   Consider changing the default SSH port, disabling root login, and using key-based authentication.

### Summary
You have now set up an Ubuntu SSH server that requires email verification for login. This configuration ensures an additional layer of security for SSH access to your server. Always test thoroughly and ensure your email configurations are correct for the verification to work as intended.
☺️🥴😄😍🤣😘😊🥴😅😅😍😊😍😊😍😊😊😍🤣😊🤣😊😍😊😍😊😍😊😊☺️😊😍👇😍😅😉😊😉😉😊😀😊😊😛😍😊☺️👇👇☺️😍😍😍😉😊😉😅🥲😅🥲😅😌😅😅😅😅😍🥴😍😅😍😅🤣😅🤣😅🤣😅🤣😅🤣😅😄😅😄😅😄😄😊😉😊😉😊😉😅😉😅😉😅😉😅😄😅😄😅😄😅😄😅😄😅😄😅😄😅😉😅😉😅😄😄😅😉😅😉😅😉😅😍😅😍😅😉😅😄😅😄😅😉😅😉😅😉😅😀😅😉😅😉😅😀😅😉😅😄😅

Setting up an Ubuntu SSH server that requires users to confirm their login via secure email link verification, along with configuring Apache2 with a domain URL, involves multiple steps. Here’s a comprehensive guide to achieve this:

### Prerequisites

1. **Ubuntu Server**: Ensure you have an Ubuntu server installed (preferably 20.04 LTS or later).
2. **Domain Name**: A registered domain name pointing to your server’s IP address.
3. **Email Server**: A mail server or email-sending service (like SendGrid, Mailgun, etc.) configured.
4. **Basic Knowledge**: Familiarity with the command line and SSH.

### Step 1: Install OpenSSH Server

1. **Update Packages**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Install OpenSSH Server**:
   ```bash
   sudo apt install openssh-server -y
   ```

3. **Start and Enable SSH**:
   ```bash
   sudo systemctl start ssh
   sudo systemctl enable ssh
   ```

### Step 2: Install Apache2

1. **Install Apache2**:
   ```bash
   sudo apt install apache2 -y
   ```

2. **Start and Enable Apache2**:
   ```bash
   sudo systemctl start apache2
   sudo systemctl enable apache2
   ```

3. **Allow Apache through Firewall**:
   ```bash
   sudo ufw allow 'Apache Full'
   ```

### Step 3: Set Up a Domain with Apache

1. **Create a Directory for Your Domain**:
   ```bash
   sudo mkdir -p /var/www/yourdomain.com/html
   ```

2. **Set Permissions**:
   ```bash
   sudo chown -R $USER:$USER /var/www/yourdomain.com/html
   sudo chmod -R 755 /var/www
   ```

3. **Create a Sample Page**:
   ```bash
   echo "<html><head><title>Welcome to Your Domain!</title></head><body><h1>Hello, World!</h1></body></html>" | sudo tee /var/www/yourdomain.com/html/index.html
   ```

4. **Create a Virtual Host File**:
   ```bash
   sudo nano /etc/apache2/sites-available/yourdomain.com.conf
   ```

   Add the following configuration:
   ```apache
   <VirtualHost *:80>
       ServerAdmin admin@yourdomain.com
       ServerName yourdomain.com
       ServerAlias www.yourdomain.com
       DocumentRoot /var/www/yourdomain.com/html

       <Directory /var/www/yourdomain.com/html>
           Options Indexes FollowSymLinks
           AllowOverride All
           Require all granted
       </Directory>

       ErrorLog ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```

5. **Enable the New Site**:
   ```bash
   sudo a2ensite yourdomain.com.conf
   ```

6. **Disable the Default Site** (optional):
   ```bash
   sudo a2dissite 000-default.conf
   ```

7. **Test Apache Configuration**:
   ```bash
   sudo apache2ctl configtest
   ```

8. **Reload Apache**:
   ```bash
   sudo systemctl reload apache2
   ```

### Step 4: Install Required Packages for Email Verification

1. **Install Python and pip**:
   ```bash
   sudo apt install python3 python3-pip -y
   ```

2. **Install Flask and Flask-Mail**:
   ```bash
   pip3 install Flask Flask-Mail
   ```

### Step 5: Set Up Flask Application for Email Verification

1. **Create a New Directory for Your Flask App**:
   ```bash
   mkdir ~/flask_app
   cd ~/flask_app
   ```

2. **Create `app.py`**:
   ```bash
   nano app.py
   ```

   Add the following code:
   ```python
   from flask import Flask, request, redirect, url_for, render_template
   from flask_mail import Mail, Message
   import os
   import random
   import string

   app = Flask(__name__)

   # Configure Flask-Mail
   app.config['MAIL_SERVER'] = 'smtp.your-email-provider.com'
   app.config['MAIL_PORT'] = 587
   app.config['MAIL_USE_TLS'] = True
   app.config['MAIL_USERNAME'] = 'your-email@example.com'
   app.config['MAIL_PASSWORD'] = 'your-email-password'
   app.config['MAIL_DEFAULT_SENDER'] = 'your-email@example.com'

   mail = Mail(app)

   # Function to generate random token
   def generate_token(length=16):
       return ''.join(random.choices(string.ascii_letters + string.digits, k=length))

   @app.route('/send_verification/<email>', methods=['GET'])
   def send_verification(email):
       token = generate_token()
       msg = Message('Login Verification', recipients=[email])
       link = url_for('verify', token=token, _external=True)
       msg.body = f'Click the link to verify your login: {link}'
       mail.send(msg)
       return 'Verification email sent!'

   @app.route('/verify/<token>', methods=['GET'])
   def verify(token):
       # Implement verification logic (e.g., mark user as verified)
       return 'Your login has been verified!'

   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000)
   ```

3. **Run the Flask Application**:
   ```bash
   python3 app.py
   ```

### Step 6: Set Up the Email Verification Process

1. **Adjust Your SSH Configuration**:
   Edit the SSH configuration file to restrict logins to users who have verified their email:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

   Add or modify the following lines:
   ```bash
   Match User <username>
       AuthenticationMethods publickey,keyboard-interactive
   ```

2. **Restart SSH Service**:
   ```bash
   sudo systemctl restart ssh
   ```

### Step 7: Firewall Configuration

Make sure your firewall allows traffic on the necessary ports:
```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Apache Full'
sudo ufw allow 5000/tcp
```

### Step 8: Secure Your Apache Server with SSL (Optional but Recommended)

1. **Install Certbot**:
   ```bash
   sudo apt install certbot python3-certbot-apache -y
   ```

2. **Obtain SSL Certificate**:
   ```bash
   sudo certbot --apache -d yourdomain.com -d www.yourdomain.com
   ```

3. **Follow the Prompts to Set Up SSL**.

### Step 9: Testing

1. **Test Email Sending**: Access the endpoint you set up for sending verification emails by navigating to `http://yourdomain.com:5000/send_verification/your-email@example.com`.

2. **Check Your Email**: You should receive an email with a verification link.

3. **Visit Verification Link**: Clicking the link should confirm the login process.

### Final Notes

- **Testing and Debugging**: Ensure your application is working by testing various scenarios (e.g., correct and incorrect email addresses).
- **Production Readiness**: Consider securing your Flask application further by using a production-ready server like Gunicorn and setting up a reverse proxy with Apache.

This setup provides a good foundation for an SSH server that uses email verification for user login along with an Apache web server. Adjust the configurations and security settings as needed for your specific use case.



😁🔗😁😄😍🥰😄🥰😍😍🥰😄😍🥰🙃🥰😍🙃🥰😍🙃😀😍🙃🥰😍🥰😉🙃🙃😍🙃😍🥰🙃😗😛😄🥰😛🙃🥰😍🙃🥰😍🙃😗😛🙃🥰😛🙃😗😛😄😗😛😄😄😗🙃😗🤪🙃🥰🤪🔗😍🔗🙃🥰😛🔗😛🔗😁😝🔗🥰😝🔗😁😝🙃🙃🤪🙃🙃🥰😍😄😍🥰😄🥰😍😄😗😛😄😗😛🙃🥰😛🙃😛🥰🙃🥰😛🙃🥰😛🔗🤪🤣🤩🤪🤣🤪😄🤩🤪🤩🤪🤣🤪🔗😁😝🔗😁🤪🔗🥰😛🙃🥰😛🙃🙃😗😛😗😄😗😛😄🤣😗🙃
😊To set up contact number-based OTP (One-Time Password) login for SSH on an Ubuntu server without using Django or Flask, you can utilize Python along with some libraries to handle OTP generation and sending. Here’s a step-by-step guide to achieve this:

### Prerequisites

1. **Ubuntu Server**: Ensure you have SSH access to your Ubuntu server.
2. **Python 3**: Python should be installed (Python 3.6 or higher recommended).
3. **pip**: Ensure `pip` is installed for package management.

### Step 1: Install Required Packages

Install the necessary packages using `pip`. You’ll need `pyotp` for OTP generation and `twilio` or `smtplib` for sending SMS or email.

```bash
sudo apt update
sudo apt install python3-pip
pip3 install pyotp twilio
```

### Step 2: Configure Twilio for SMS (or use Email)

If you choose to send OTPs via SMS, sign up for a [Twilio account](https://www.twilio.com/) and set up a phone number. Note down your **Account SID**, **Auth Token**, and the **Twilio phone number**.

Alternatively, for email, you can use Python’s built-in `smtplib` to send OTPs via email.

### Step 3: Create the OTP Generation and Sending Script

Create a Python script named `otp_server.py` that will handle OTP generation and sending.

```python
import pyotp
import os
import time
from twilio.rest import Client

# Twilio configuration
TWILIO_ACCOUNT_SID = 'your_account_sid'
TWILIO_AUTH_TOKEN = 'your_auth_token'
TWILIO_PHONE_NUMBER = 'your_twilio_number'
client = Client(TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN)

def send_otp(phone_number, otp):
    message = client.messages.create(
        body=f'Your OTP is: {otp}',
        from_=TWILIO_PHONE_NUMBER,
        to=phone_number
    )
    return message.sid

def generate_otp(secret):
    totp = pyotp.TOTP(secret)
    return totp.now()

if __name__ == '__main__':
    phone_number = input("Enter your phone number: ")
    secret = pyotp.random_base32()  # Store this securely for user authentication later

    print("Your OTP secret (store this securely):", secret)

    # Generate and send OTP
    otp = generate_otp(secret)
    send_otp(phone_number, otp)
    print(f"OTP sent to {phone_number}: {otp}")

    # Wait for user input to verify
    user_otp = input("Enter the OTP you received: ")
    totp = pyotp.TOTP(secret)
    if totp.verify(user_otp):
        print("OTP verified successfully!")
    else:
        print("Invalid OTP.")
```

### Step 4: Configure PAM for SSH

To configure SSH to use PAM for OTP verification, you need to modify the PAM configuration files.

1. **Edit the SSH PAM Configuration**:

   Open the SSH PAM configuration file:

   ```bash
   sudo nano /etc/pam.d/sshd
   ```

   Add the following line to the top of the file to include your OTP verification script:

   ```bash
   auth required pam_exec.so /path/to/your/otp_server.py
   ```

   Ensure that `pam_exec.so` is pointing to your OTP script. The path must be executable by the SSH service.

2. **Edit the SSH Configuration**:

   Open the SSH configuration file:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

   Ensure the following lines are present and uncommented:

   ```bash
   ChallengeResponseAuthentication yes
   UsePAM yes
   ```

### Step 5: Restart SSH Service

After making changes, restart the SSH service:

```bash
sudo systemctl restart ssh
```

### Step 6: Testing the OTP Login

1. Open a terminal on a different machine and try to SSH into your Ubuntu server.
   
   ```bash
   ssh username@your_server_ip
   ```

2. You should be prompted to enter your OTP. Follow the instructions in your `otp_server.py` script to enter the OTP sent to your phone.

### Notes:

- Ensure that your script runs properly without errors. You may need to manage where the OTP secret is stored and ensure secure storage.
- This setup is for basic OTP handling. Consider implementing additional security measures, such as logging, rate limiting, and error handling, for production use.
- Always test your setup in a safe environment before deploying it on a live server.

With this setup, you should have a basic contact number-based OTP login for SSH configured on your Ubuntu server!


😁🤩🙂😂🥰👇😂😍🤩😂👇🥰😗👇🥰😂😍👇😂😍😉😂😍😉😂😍😉😂😍😉😂😍👇😂🥰👇😂🥰👇😂🥰😗🔑😗🔑😀😗🔑😀😗🥰😀😗😀😂😀🥰😗🥰😀😂🥰😀😗😀🥰😗🔑😀😗🥰😀😂🥰😀😗🥰😀😗🥰😀😗🥰😀😗😀😗🥰😀😗🔑😀😗🥰😀😂🥰😀😗🥰😊😂🥰😊😗🥰😊🔑😊😂🥰😊😛🔑😊😗🔑😊😊😂🙂😃😗😃😀😗🔑😀😗🔑😀😗👇😗🔑😀😗🔑😀😂😗🥰😊😗🔑😊😗🔑😊😂🔑😂🔑
To set up email-based OTP login for SSH on an Ubuntu server, you can follow these steps. We’ll use Python to generate and send OTPs via email, and configure SSH to work with PAM for OTP-based authentication. Here’s how:

### Step 1: Install Required Packages
First, ensure your system has the necessary tools:

```bash
sudo apt update
sudo apt install python3 python3-pip libpam-python libpam-google-authenticator
```

The `libpam-python` library will allow us to write a custom PAM module in Python, and `libpam-google-authenticator` can be used to generate OTPs.

### Step 2: Set Up Python Script to Generate and Send OTPs
Create a Python script that will generate and send OTPs to a user’s email. We’ll use Python’s `smtplib` to send the email and `random` to generate OTPs.

1. **Create the Python Script:**

   ```bash
   sudo nano /usr/local/bin/send_otp.py
   ```

2. **Add the Following Code:**

   ```python
   import smtplib
   import random
   import sys
   from email.mime.text import MIMEText

   # Replace these values with your email provider’s details
   SMTP_SERVER = "smtp.example.com"
   SMTP_PORT = 587
   SMTP_USER = "your_email@example.com"
   SMTP_PASSWORD = "your_password"

   def send_otp(email):
       otp = str(random.randint(100000, 999999))
       message = MIMEText(f"Your OTP code is: {otp}")
       message["Subject"] = "Your One-Time Password"
       message["From"] = SMTP_USER
       message["To"] = email

       with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
           server.starttls()
           server.login(SMTP_USER, SMTP_PASSWORD)
           server.sendmail(SMTP_USER, email, message.as_string())

       return otp

   if __name__ == "__main__":
       email = sys.argv[1]
       otp = send_otp(email)
       print(otp)
   ```

3. **Make the Script Executable:**

   ```bash
   sudo chmod +x /usr/local/bin/send_otp.py
   ```

### Step 3: Configure PAM to Use the OTP Script

1. **Create a PAM Module Script:**

   Create a script that calls `send_otp.py` and verifies the OTP entered by the user:

   ```bash
   sudo nano /etc/security/otp_pam.py
   ```

2. **Add the Following Code:**

   ```python
   import pam
   import subprocess

   def pam_sm_authenticate(pamh, flags, argv):
       user_email = "user_email@example.com"  # Replace with actual email

       otp = subprocess.check_output(["/usr/local/bin/send_otp.py", user_email]).strip()
       user_input = pamh.conversation(pamh.Message(pamh.PAM_PROMPT_ECHO_ON, "Enter OTP: ")).resp

       if user_input == otp.decode():
           return pamh.PAM_SUCCESS
       return pamh.PAM_AUTH_ERR

   def pam_sm_setcred(pamh, flags, argv):
       return pamh.PAM_SUCCESS

   def pam_sm_acct_mgmt(pamh, flags, argv):
       return pamh.PAM_SUCCESS

   def pam_sm_open_session(pamh, flags, argv):
       return pamh.PAM_SUCCESS

   def pam_sm_close_session(pamh, flags, argv):
       return pamh.PAM_SUCCESS

   def pam_sm_chauthtok(pamh, flags, argv):
       return pamh.PAM_SUCCESS
   ```

3. **Make the Script Executable:**

   ```bash
   sudo chmod +x /etc/security/otp_pam.py
   ```

### Step 4: Configure SSH to Use PAM

1. **Edit SSH Configuration:**

   Open the SSH configuration file and enable PAM:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

   Find the line:

   ```bash
   #UsePAM no
   ```

   Change it to:

   ```bash
   UsePAM yes
   ```

2. **Edit PAM SSH Configuration:**

   Add your custom PAM module to SSH’s PAM configuration file:

   ```bash
   sudo nano /etc/pam.d/sshd
   ```

   Add the following line at the top:

   ```bash
   auth required pam_python.so /etc/security/otp_pam.py
   ```

### Step 5: Restart SSH Service

```bash
sudo systemctl restart ssh
```

### Step 6: Test the Setup

Now, when you try to SSH into the server, you should receive an OTP to your specified email, which you can then use to log in.

**Note:** This setup sends the OTP to a hardcoded email in the script. In a production environment, you'd likely implement a method to dynamically fetch each user's email. Additionally, for security, avoid storing plaintext credentials in scripts. Instead, use environment variables or other secure methods to store sensitive information.

🙃😙😘☺️☺️👇🤪👇☺️👇☺️👇😃👇😃☺️👇😀👇😃👇🙃👇😃🥲😙😝☺️🤩😝😙😝😝😃😊😝😙😝🙃😝😊😝😊😝😙😝😃😙😝😊😝😙😝😃😝🙃🥲😃🥲😃🥲😃🥲😁😁🥲😊🥲😊🥲😄🥲😄😄🥲☺️🥲☺️🥲😄🥲😊🥲😊🥲😄🥲😙🥲😙😌😃😌😙😌😙😙😌😙😌😊😌😊😌😙

Setting up email-based OTP (One-Time Password) login on an Ubuntu server involves configuring PAM (Pluggable Authentication Module) to use `oathtool` for OTP generation and integrating it with an email tool to send the OTPs to the user's email. Here’s a detailed step-by-step guide:

### 1. Install Required Packages

You need `oathtool` for OTP generation and a mail utility (e.g., `mailutils`) to send emails.

```bash
sudo apt update
sudo apt install oathtool mailutils -y
```

### 2. Create a Script to Send OTP via Email

You'll need a script that will generate an OTP, send it via email, and log the OTP for verification.

1. **Create the script**:

    ```bash
    sudo nano /usr/local/bin/send-otp.sh
    ```

2. **Add the following code** to generate and email the OTP:

    ```bash
    #!/bin/bash

    # Define variables
    USER_EMAIL="user@example.com"  # Replace with the user's email
    OTP_LENGTH=6  # Set the length of the OTP

    # Generate OTP using oathtool
    OTP=$(oathtool --totp -d $OTP_LENGTH)

    # Send OTP via email
    echo "Your OTP for login is: $OTP" | mail -s "Your Login OTP" "$USER_EMAIL"

    # Log the OTP to a file with restricted permissions (for testing)
    echo "$OTP" > /tmp/otp.txt
    chmod 600 /tmp/otp.txt
    ```

    **Replace** `user@example.com` with the actual email address you want to receive the OTP.

3. **Make the script executable**:

    ```bash
    sudo chmod +x /usr/local/bin/send-otp.sh
    ```

### 3. Set Up a Custom PAM Module to Call the Script

1. **Create a new PAM configuration file**:

    ```bash
    sudo nano /etc/pam.d/otp-login
    ```

2. **Add the following PAM configuration**:

    ```plaintext
    auth required pam_exec.so /usr/local/bin/send-otp.sh
    auth sufficient pam_exec.so /usr/local/bin/check-otp.sh
    ```

This configuration will:

- Run the `send-otp.sh` script when a user attempts to authenticate.
- Call `check-otp.sh` (which we’ll create next) to verify the OTP entered by the user.

### 4. Create a Script to Check the OTP Entered by the User

1. **Create the `check-otp.sh` script**:

    ```bash
    sudo nano /usr/local/bin/check-otp.sh
    ```

2. **Add the following code**:

    ```bash
    #!/bin/bash

    # Prompt the user for the OTP
    read -p "Enter OTP: " user_otp

    # Read the generated OTP from the file
    correct_otp=$(cat /tmp/otp.txt)

    # Compare user input with generated OTP
    if [[ "$user_otp" == "$correct_otp" ]]; then
        exit 0  # Success
    else
        echo "Invalid OTP."
        exit 1  # Failure
    fi
    ```

3. **Make the script executable**:

    ```bash
    sudo chmod +x /usr/local/bin/check-otp.sh
    ```

### 5. Apply OTP-based Login to a Specific User or Service

To apply OTP-based login for SSH:

1. **Edit the SSH PAM configuration**:

    ```bash
    sudo nano /etc/pam.d/sshd
    ```

2. **Add the following line** at the top of the file to include the OTP module:

    ```plaintext
    auth required pam_exec.so /etc/pam.d/otp-login
    ```

3. **Restart SSH** to apply the changes:

    ```bash
    sudo systemctl restart sshd
    ```

### 6. Test the OTP Login

1. Attempt to log in via SSH. After entering your username and password, you should receive an OTP email.
2. Enter the OTP as prompted by the `check-otp.sh` script.
3. If the OTP is correct, the login should succeed; otherwise, it will deny access.

### Notes and Security Considerations

- **Security**: OTPs should be securely deleted or managed in production. Avoid storing OTPs in plain text.
- **Permissions**: Ensure only authorized users can modify the PAM scripts and configuration files.
😆😅🥴🥴😅🥴😛😀👇🔑😀🔑🤪😝🔑🤪🤩🔑😘🤩🤩😘😘🤩😘🤩😘🤩👇😉😌😀🔑😌😀🔑😌😀🔑😌🔑😀😌😀🔑😌🔑😌🔑😃☺️😃🔑☺️🥲🔑☺️🔑🥲☺️🔑🥲☺️😃🔑☺️😃🔑☺️😃🔑☺️😃🔑😃😃☺️☺️😃🔑☺️🔑😃☺️😃🔑☺️😃🔑😍😃🔑😍😃🔑🔑🥰😊😉😃😊🔑😃😊😃😉😊😉😃😊😃😉😉😊
To set up email OTP (One-Time Password) login on an Ubuntu server, you can use tools like `PAM` (Pluggable Authentication Modules), `otpw` (OTP generator), and `ssmtp` (for email sending). Here’s a general guide to get it up and running:

### Prerequisites

1. **Ubuntu server** with administrative privileges.
2. **SMTP relay** credentials (e.g., Gmail SMTP) for sending OTPs via email.

### Step 1: Install Required Packages

```bash
sudo apt update
sudo apt install libpam-otpw ssmtp mailutils
```

- **libpam-otpw**: Provides PAM module for OTP-based login.
- **ssmtp**: A simple mail transfer program to send emails.
- **mailutils**: Provides the `mail` command-line tool to send email.

### Step 2: Configure `ssmtp` to Send Emails

1. **Edit `ssmtp.conf`:**

   ```bash
   sudo nano /etc/ssmtp/ssmtp.conf
   ```

2. **Configure SMTP settings** (example with Gmail):

   ```plaintext
   mailhub=smtp.gmail.com:587
   AuthUser=your-email@gmail.com
   AuthPass=your-email-password
   UseSTARTTLS=YES
   FromLineOverride=YES
   ```

3. **Test email sending:**

   ```bash
   echo "Test email" | mail -s "Test" recipient@example.com
   ```

### Step 3: Configure OTPW for OTP Generation

1. **Generate OTP Seeds for a User:**

   As the target user, run the following to create OTP entries:

   ```bash
   otpw-gen
   ```

   This will create `~/.otpw` containing OTP seeds. It also generates a list of OTPs, each with a unique number.

2. **Store OTP List Securely:** Save this list for reference, as each OTP can be used only once.

### Step 4: Configure PAM to Use OTP for SSH Login

1. **Edit PAM Configuration for SSH:** Open the SSH PAM configuration:

   ```bash
   sudo nano /etc/pam.d/sshd
   ```

2. **Add PAM OTPW Module:** Add the following line at the top:

   ```plaintext
   auth required pam_otpw.so
   ```

3. **Restrict SSH to OTP-based Authentication:** 

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

   Set the following parameters:

   ```plaintext
   ChallengeResponseAuthentication yes
   PasswordAuthentication no
   ```

4. **Restart SSH Service:**

   ```bash
   sudo systemctl restart ssh
   ```

### Step 5: Set Up a Script to Email the OTP

Create a script to email the OTP to users, then add this script to `/etc/otpw-send-mail.sh`:

```bash
#!/bin/bash
OTP=$(head -1 ~/.otpw | cut -d ' ' -f 2)
echo "Your OTP is: $OTP" | mail -s "OTP Login Code" recipient@example.com
```

Make the script executable:

```bash
sudo chmod +x /etc/otpw-send-mail.sh
```

### Step 6: Test the OTP-Based SSH Login

1. **SSH to the server:**

   ```bash
   ssh your_user@server_ip
   ```

2. **Receive an OTP email**, and enter the OTP when prompted.

This setup should now allow you to use email OTPs for SSH logins on your Ubuntu server.



