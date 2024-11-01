Setting up an Ubuntu SSH server with email verification and securing it with Apache and a domain name involves several steps. Here’s a guide to get you started:

### 1. **Install OpenSSH Server**

First, make sure the OpenSSH server is installed on your Ubuntu machine:

```bash
sudo apt update
sudo apt install openssh-server
```

After installation, you can check the SSH status to confirm it's running:

```bash
sudo systemctl status ssh
```

### 2. **Configure SSH for Security**

For additional security, consider modifying SSH configurations:

```bash
sudo nano /etc/ssh/sshd_config
```

Make the following changes:
- Change the default SSH port from 22 to a custom port (e.g., 2222).
- Disable root login by setting `PermitRootLogin no`.
- Allow only specific users with `AllowUsers <username>`.
- Disable password authentication and use key-based authentication by setting `PasswordAuthentication no`.

After making these changes, restart SSH:

```bash
sudo systemctl restart ssh
```

### 3. **Set Up Email Verification for SSH Logins**

To implement email verification, we can use `pam_exec` in the Pluggable Authentication Module (PAM) stack. Here’s an outline of the process:

1. **Install necessary packages:**

   ```bash
   sudo apt install mailutils
   ```

2. **Create a verification script:**

   Create a script that sends a verification code by email.

   ```bash
   sudo nano /usr/local/bin/email_verification.sh
   ```

   Add the following content (customize the email and user handling as necessary):

   ```bash
   #!/bin/bash

   EMAIL="user@example.com"  # Replace with user's email address
   CODE=$(shuf -i 1000-9999 -n 1)
   echo $CODE > /tmp/ssh_code_${PAM_USER}
   echo "Your SSH verification code is $CODE" | mail -s "SSH Verification Code" $EMAIL
   ```

3. **Make the script executable:**

   ```bash
   sudo chmod +x /usr/local/bin/email_verification.sh
   ```

4. **Configure PAM to use the script:**

   Edit `/etc/pam.d/sshd` to run the script during login.

   ```bash
   sudo nano /etc/pam.d/sshd
   ```

   Add this line at the beginning of the file:

   ```bash
   auth required pam_exec.so /usr/local/bin/email_verification.sh
   ```

5. **Prompt for code verification:**

   Create another PAM module that prompts users to enter the code sent via email:

   ```bash
   sudo nano /usr/local/bin/verify_code.sh
   ```

   Add:

   ```bash
   #!/bin/bash

   read -p "Enter the verification code: " input_code
   stored_code=$(cat /tmp/ssh_code_${PAM_USER})

   if [[ "$input_code" != "$stored_code" ]]; then
       echo "Verification failed."
       exit 1
   else
       echo "Verification successful."
       exit 0
   fi
   ```

   Make it executable:

   ```bash
   sudo chmod +x /usr/local/bin/verify_code.sh
   ```

   Then add this to `/etc/pam.d/sshd`:

   ```bash
   auth required pam_exec.so /usr/local/bin/verify_code.sh
   ```

**Note:** This is a basic verification setup. For production, consider more robust solutions like using 2FA or third-party libraries.

### 4. **Set Up Apache Web Server**

Install Apache:

```bash
sudo apt install apache2
```

To ensure Apache runs on boot, use:

```bash
sudo systemctl enable apache2
```

### 5. **Get a Domain Name and Secure with SSL**

1. **Set Up a Domain Name**: Register a domain name from a provider (e.g., GoDaddy, Namecheap).

2. **Point Your Domain to Your Server**: Update the DNS settings for your domain to point to your server’s IP address.

3. **Install Certbot for SSL**:

   ```bash
   sudo apt install certbot python3-certbot-apache
   ```

4. **Obtain an SSL Certificate**:

   ```bash
   sudo certbot --apache
   ```

   Follow the prompts to choose the domain and configure automatic HTTP to HTTPS redirection.

5. **Test the SSL Certificate**:

   After installation, test your configuration:

   ```bash
   sudo certbot renew --dry-run
   ```

### 6. **Configure SSH Access via Apache (Optional)**

For additional security, you could configure SSH access behind an Apache reverse proxy. This is more complex and typically involves proxying services through HTTPS to obfuscate your SSH port further.

### 7. **Configure Firewall**

Secure your server by allowing only the necessary ports:

```bash
sudo ufw allow 2222/tcp    # Custom SSH port
sudo ufw allow "Apache Full"
sudo ufw enable
```

### 8. **Test the Configuration**

After setup, test by logging in via SSH. You should receive an email with the verification code, and only upon entering the correct code will you gain access.

😍🥹😘☺️😘☺️🥹😊☺️😁😊😍😘🥹☺️😘🥹😍😘😚😍😚😘☺️😃😍😚😊☺️😃☺️😚🥰☺️😃😚☺️🥰😚☺️😚🥰😍😚🥰☺️😚😃☺️😚🥰☺️☺️😚😊😚🥰☺️😚🥰😚😃☺️😚😃😚☺️😃😚☺️😜😃😜😃😜☺️☺️🥹😌😊😊🥹😊😉😊🥹😊🥹😅🙂😠😊😄😠😊😄😊😠😝😄😠😜😚😠😜😚😠😜😁😠😜😙😜😁🙁😠😜😙🙁😜😚🙁😜😄🙁😙🙁😜😙😠😜😄🙁😜😄🙁😄😜😠😄😝😄🙁😜😄😠😜😄🙁😜😄😠😝😄😠
Setting up an SSH server on Ubuntu with email verification is a unique and advanced configuration. Although SSH itself doesn't natively support email verification, it can be implemented through custom scripts and external email services. Here’s a general outline of how to set up such a system:

### 1. Configure SSH Server

Start by installing and configuring the OpenSSH server on your Ubuntu machine:

1. **Install OpenSSH Server:**
   ```bash
   sudo apt update
   sudo apt install openssh-server
   ```

2. **Edit SSH Configuration:**
   Open the SSH configuration file to set basic parameters.
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
   - Set `PermitRootLogin` to `no` to enhance security.
   - Consider adjusting `PasswordAuthentication` to `no` if using public keys.
   - Save and close the file, then restart the SSH service:
     ```bash
     sudo systemctl restart ssh
     ```

### 2. Set Up Email Verification Script

To require email verification, you can create a script that sends a verification email whenever a login is attempted. The user would need to enter the code or verify the email within a certain timeframe to gain access.

1. **Create the Verification Script:**

   Here’s a basic example using Python and `smtplib` for sending emails:

   ```python
   # /home/username/verify_login.py
   import smtplib
   import random
   from email.mime.text import MIMEText

   def send_verification_email(recipient_email, code):
       smtp_server = "smtp.example.com"
       smtp_port = 587
       smtp_user = "you@example.com"
       smtp_password = "yourpassword"

       msg = MIMEText(f"Your SSH verification code is: {code}")
       msg["Subject"] = "SSH Login Verification Code"
       msg["From"] = smtp_user
       msg["To"] = recipient_email

       with smtplib.SMTP(smtp_server, smtp_port) as server:
           server.starttls()
           server.login(smtp_user, smtp_password)
           server.sendmail(smtp_user, recipient_email, msg.as_string())

   if __name__ == "__main__":
       verification_code = random.randint(100000, 999999)
       send_verification_email("user@example.com", verification_code)
       print("Verification code sent!")
   ```

2. **Set Up PAM (Pluggable Authentication Module):**

   You can set up a PAM script that invokes the email verification script during the login process.

   - Create a PAM module (in `/etc/pam.d/sshd`) that calls the verification script.
   - Use `pam_exec` to run the Python verification script, checking for the code before allowing access.

3. **Verify Code Entry:**

   Modify the verification script to store the code temporarily (e.g., in a file or database) and create a mechanism for the user to enter this code as part of the SSH login.

### 3. Secure Server with Apache

Apache can act as a reverse proxy or a central access point, which can help protect the SSH service by restricting access to certain IPs or networks.

1. **Install Apache:**
   ```bash
   sudo apt install apache2
   ```

2. **Configure Reverse Proxy:**
   Enable required modules and configure Apache to proxy SSH connections if necessary:
   ```bash
   sudo a2enmod proxy
   sudo a2enmod proxy_http
   sudo nano /etc/apache2/sites-available/000-default.conf
   ```
   - Add proxy directives to route traffic appropriately.

3. **Enable SSL/TLS:**
   Set up HTTPS on Apache to secure the connection further.
   ```bash
   sudo a2enmod ssl
   sudo a2ensite default-ssl
   sudo systemctl reload apache2
   ```

### 4. Test the Setup

Attempt to log in via SSH, which should trigger the email verification process. Enter the code when prompted to complete the login.

---

This approach requires careful testing and tuning for your specific environment. Let me know if you need more details on any specific part of this setup!



😀😀😃☺️😀🙂‍↕️😍🙂‍↕️☺️🙂‍↕️😍☺️🙂‍↕️😀☺️😍🙂‍↕️☺️🙂‍↕️😍☺️😘☺️😘🙂‍↕️☺️🙂‍↕️😍☺️🙂‍↕️🥭😍☺️😍🥭🥭😌😌😍🥭😍🥭😌😌🥭😍🙂‍↕️☺️🙂‍↕️😀🙂‍↕️☺️🙂‍↕️😍🙂‍↕️😍😍😍🙂‍↕️😀😀☺️☺️🙂‍↕️😀☺️🙂‍↕️😀☺️😀🙂‍↕️😀🙂‍↕️☺️☺️🙂‍↕️😀😊😀🙂‍↕️😊🙂‍↕️😛🙂‍↕️😀😊🙂‍↕️😛😛🙂‍↕️☺️😀🙂‍↕️😛😊😊😛😊😛😛😛😊😀😜😊😜😀😊😜😀😊😜😀😛😊😛😊😜😀😊😜😉😊😛😜😊😀😜😊😜😉😊😊😜😉😊😜😀😊😉😜😊😜😉🥰😜😉🥰😉😜😉🥰😜😉🥰😊😉😜
To set up an Ubuntu SSH server that requires email verification for user logins, you can follow these steps:

### 1. Configure SSH Server
Start by setting up a basic SSH server on Ubuntu. 

1. **Install the OpenSSH Server**:
   ```bash
   sudo apt update
   sudo apt install openssh-server
   ```
   
2. **Edit SSH Configuration**:
   Open the SSH configuration file to make necessary changes:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
   
   Change some settings to improve security:
   - **Disable Root Login**:
     ```plaintext
     PermitRootLogin no
     ```
   - **Set a Limited Login Grace Period** (optional):
     ```plaintext
     LoginGraceTime 1m
     ```
   - **Restrict SSH to Key-based Authentication** (for added security):
     ```plaintext
     PasswordAuthentication no
     ```
   
   Save and exit, then restart the SSH server to apply changes:
   ```bash
   sudo systemctl restart ssh
   ```

### 2. Set Up Email Verification for SSH Access
Implementing email verification will require additional scripting. We’ll use a Python script to handle email verification by generating a one-time code and sending it to the user’s email address.

1. **Install Required Packages**:
   You’ll need Python and `ssmtp` or `sendmail` (for email sending).
   ```bash
   sudo apt install python3 python3-pip ssmtp
   pip3 install smtplib email
   ```

2. **Create the Verification Script**:
   Create a script that will:
   - Generate a one-time code
   - Send the code to the user’s email address
   - Prompt the user to enter the code to complete login
   
   Example script (`/usr/local/bin/email_verification.py`):
   ```python
   import smtplib
   import random
   from email.mime.text import MIMEText
   from getpass import getuser

   # Configuration - change these values
   SMTP_SERVER = 'smtp.example.com'
   SMTP_PORT = 587
   SMTP_USER = 'your_email@example.com'
   SMTP_PASSWORD = 'your_email_password'
   USER_EMAIL = 'user_email@example.com'  # User’s email address

   def send_verification_code(code):
       msg = MIMEText(f"Your verification code is: {code}")
       msg['Subject'] = 'SSH Login Verification'
       msg['From'] = SMTP_USER
       msg['To'] = USER_EMAIL

       with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
           server.starttls()
           server.login(SMTP_USER, SMTP_PASSWORD)
           server.sendmail(SMTP_USER, USER_EMAIL, msg.as_string())

   def main():
       code = random.randint(100000, 999999)
       send_verification_code(code)
       user_code = input("Enter the verification code sent to your email: ")
       if str(user_code) == str(code):
           print("Verification successful.")
       else:
           print("Verification failed.")
           exit(1)

   if __name__ == '__main__':
       main()
   ```

3. **Integrate Script with SSH Login**:
   Modify the `~/.bashrc` file to prompt users for the verification code after they log in:
   ```bash
   nano ~/.bashrc
   ```

   Add the following line at the end:
   ```bash
   python3 /usr/local/bin/email_verification.py
   ```
   This ensures that the email verification runs every time a user logs in over SSH.

### 3. Secure the Server with Apache (Optional Step for HTTPS Proxy)
If you want to secure communications further, you can use Apache as an HTTPS proxy. This is more common for applications needing web interfaces, but if you need this for other purposes, here’s how to set it up:

1. **Install Apache**:
   ```bash
   sudo apt install apache2
   ```

2. **Enable SSL Module**:
   ```bash
   sudo a2enmod ssl
   ```

3. **Create a Self-Signed SSL Certificate** (or use a certificate from a trusted CA like Let's Encrypt):
   ```bash
   sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
   ```

4. **Configure Apache for HTTPS**:
   Edit the default SSL configuration:
   ```bash
   sudo nano /etc/apache2/sites-available/default-ssl.conf
   ```
   
   Update the following lines:
   ```plaintext
   SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
   SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
   ```

5. **Enable the SSL Site and Restart Apache**:
   ```bash
   sudo a2ensite default-ssl.conf
   sudo systemctl reload apache2
   ```

### 4. Testing and Finalizing
Test the SSH login and email verification to ensure everything works smoothly. 

Make sure to adjust firewall settings (using `ufw` or `iptables`) to allow SSH and HTTPS traffic:

```bash
sudo ufw allow ssh
sudo ufw allow https
sudo ufw enable
```

This setup will require users to authenticate via SSH and then verify their identity with a code sent via email before fully logging in.

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
Setting up an email-based OTP (One-Time Password) login on an Ubuntu server without using `oathtool` involves a few steps, including configuring PAM, using an appropriate scripting method to generate OTPs, and setting up an email server or client to send the OTPs to users. Below is a general guide on how to achieve this.

### Prerequisites
- An Ubuntu server (16.04 or later).
- Access to the server with sudo privileges.
- An email server or a tool like `sendmail` or `ssmtp` configured for sending emails.

### Step 1: Install Required Packages

1. **Install Python** (if not already installed):

   ```bash
   sudo apt update
   sudo apt install python3 python3-pip
   ```

2. **Install the required email package**:

   You can use Python's built-in `smtplib` for sending emails, or you can install an additional package for more advanced email features, such as `yagmail`.

   ```bash
   pip3 install yagmail
   ```

### Step 2: Create a Script to Generate and Send OTP

1. **Create the OTP generation script**:

   Create a Python script named `send_otp.py`:

   ```bash
   sudo nano /usr/local/bin/send_otp.py
   ```

   Add the following code:

   ```python
   #!/usr/bin/env python3
   import smtplib
   import random
   import string
   import sys
   from email.mime.text import MIMEText

   def generate_otp(length=6):
       """Generate a random OTP."""
       digits = string.digits
       otp = ''.join(random.choice(digits) for _ in range(length))
       return otp

   def send_email(to_email, otp):
       """Send the OTP to the specified email address."""
       from_email = 'your_email@example.com'  # Your email address
       from_password = 'your_email_password'  # Your email password
       subject = 'Your OTP Code'
       body = f'Your OTP code is: {otp}'

       msg = MIMEText(body)
       msg['Subject'] = subject
       msg['From'] = from_email
       msg['To'] = to_email

       with smtplib.SMTP('smtp.example.com', 587) as server:
           server.starttls()
           server.login(from_email, from_password)
           server.sendmail(from_email, to_email, msg.as_string())

   if __name__ == '__main__':
       user_email = sys.argv[1]
       otp = generate_otp()
       send_email(user_email, otp)
       print(otp)  # Optionally, print the OTP for debugging
   ```

   Replace `your_email@example.com` and `your_email_password` with your actual email and password. Replace `smtp.example.com` with your SMTP server address.

2. **Make the script executable**:

   ```bash
   sudo chmod +x /usr/local/bin/send_otp.py
   ```

### Step 3: Configure PAM

1. **Edit the PAM configuration file** for the service you want to protect, for example, SSH:

   ```bash
   sudo nano /etc/pam.d/sshd
   ```

   Add the following line at the beginning of the file:

   ```plaintext
   auth required pam_exec.so /usr/local/bin/send_otp.py
   ```

2. **Make sure to adjust the PAM configuration for OTP** by including the following lines if they are not present:

   ```plaintext
   auth required pam_unix.so
   auth required pam_faildelay.so delay=2000000
   ```

### Step 4: Configure SSH for Challenge-Response Authentication

1. **Edit the SSH configuration file**:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

   Find and change (or add) the following settings:

   ```plaintext
   ChallengeResponseAuthentication yes
   PasswordAuthentication no
   ```

2. **Restart the SSH service**:

   ```bash
   sudo systemctl restart ssh
   ```

### Step 5: Test the Setup

1. **Attempt to log in via SSH**. You should receive an email with the OTP code. 
2. **Enter the OTP code** received in your email to complete the login.

### Note

- This setup provides a basic email-based OTP system. For production use, consider implementing additional security measures, such as rate limiting the number of OTP requests, using secure email handling, and ensuring sensitive information is not hardcoded.
- Depending on your email provider, you may need to enable "less secure apps" or use an app-specific password to allow the script to send emails.
- It's also recommended to log or store OTPs temporarily to validate the entered OTP against generated ones, possibly using a simple file or database.

### Conclusion

By following these steps, you should be able to set up email-based OTP login on your Ubuntu server without using `oathtool`. Adjust the implementation according to your needs, especially regarding security and usability.

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



