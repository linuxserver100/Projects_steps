
Setting up an SSH server on Ubuntu that requires users to confirm their login via an email link involves several steps, including installing necessary software, configuring SSH, and setting up email notifications. Here’s a step-by-step guide to achieve this:

### Prerequisites
- An Ubuntu server (18.04 or later).
- Root or sudo access to the server.
- A domain name (for email confirmation) and an email service set up (e.g., SMTP server).

### Step 1: Install SSH Server
1. **Update Package Lists:**
   ```bash
   sudo apt update
   ```

2. **Install OpenSSH Server:**
   ```bash
   sudo apt install openssh-server
   ```

3. **Enable and Start SSH Service:**
   ```bash
   sudo systemctl enable ssh
   sudo systemctl start ssh
   ```

4. **Check SSH Service Status:**
   ```bash
   sudo systemctl status ssh
   ```

### Step 2: Set Up User Accounts
1. **Add a New User:**
   ```bash
   sudo adduser username
   ```

   Replace `username` with the desired username. Follow the prompts to set a password and provide additional information.

2. **Add Users to the Sudo Group (Optional):**
   ```bash
   sudo usermod -aG sudo username
   ```

### Step 3: Install and Configure Required Software
You will need a web server (like Apache or Nginx) to send confirmation emails, as well as a mailing library. Here we will use **Postfix** for sending emails.

1. **Install Postfix:**
   ```bash
   sudo apt install postfix
   ```

   During installation, you will be prompted to configure Postfix. Select "Internet Site" and set your server’s domain name.

2. **Install PHP and Necessary Modules (if using PHP):**
   If you choose to create a simple PHP application for email confirmation, install PHP:
   ```bash
   sudo apt install php libapache2-mod-php
   ```

3. **Install Other Required Software:**
   If you plan to use a database to manage users, install MySQL or PostgreSQL:
   ```bash
   sudo apt install mysql-server
   ```

### Step 4: Create a Confirmation System
You need a system to generate unique tokens and send email confirmations. This example will create a simple PHP script.

1. **Set Up a Web Directory:**
   ```bash
   sudo mkdir /var/www/html/ssh_confirmation
   sudo chown -R www-data:www-data /var/www/html/ssh_confirmation
   ```

2. **Create a PHP Script:**
   Create a file named `confirm.php` in `/var/www/html/ssh_confirmation`:
   ```bash
   sudo nano /var/www/html/ssh_confirmation/confirm.php
   ```

   Add the following code to handle token verification:
   ```php
   <?php
   $token = $_GET['token'];

   // Database connection
   $conn = new mysqli('localhost', 'db_user', 'db_pass', 'db_name');

   if ($conn->connect_error) {
       die("Connection failed: " . $conn->connect_error);
   }

   // Check if token is valid
   $sql = "SELECT * FROM users WHERE token='$token'";
   $result = $conn->query($sql);

   if ($result->num_rows > 0) {
       // Token is valid
       // Update user's status
       $sql = "UPDATE users SET confirmed=1 WHERE token='$token'";
       $conn->query($sql);
       echo "Your email has been confirmed!";
   } else {
       echo "Invalid token!";
   }

   $conn->close();
   ?>
   ```

3. **Create User Registration and Token Generation Script:**
   Create another PHP script named `register.php`:
   ```bash
   sudo nano /var/www/html/ssh_confirmation/register.php
   ```

   Add code to register users and send confirmation emails:
   ```php
   <?php
   if ($_SERVER["REQUEST_METHOD"] == "POST") {
       $email = $_POST['email'];
       $token = bin2hex(random_bytes(16)); // Generate a token

       // Database connection
       $conn = new mysqli('localhost', 'db_user', 'db_pass', 'db_name');

       if ($conn->connect_error) {
           die("Connection failed: " . $conn->connect_error);
       }

       // Insert user with token
       $sql = "INSERT INTO users (email, token, confirmed) VALUES ('$email', '$token', 0)";
       $conn->query($sql);

       // Send confirmation email
       $to = $email;
       $subject = "Email Confirmation";
       $message = "Click the link to confirm your email: http://your_domain.com/ssh_confirmation/confirm.php?token=$token";
       mail($to, $subject, $message);

       echo "Confirmation email sent!";
       $conn->close();
   }
   ?>
   ```

4. **Create the Users Table in Your Database:**
   Run the following commands in MySQL to create the users table:
   ```sql
   CREATE DATABASE db_name;
   USE db_name;

   CREATE TABLE users (
       id INT AUTO_INCREMENT PRIMARY KEY,
       email VARCHAR(255) NOT NULL,
       token VARCHAR(255) NOT NULL,
       confirmed TINYINT(1) DEFAULT 0
   );
   ```

### Step 5: Configure SSH to Require Confirmation
1. **Install a PAM Module for Authentication:**
   You can use PAM (Pluggable Authentication Module) to add email confirmation. However, implementing a full PAM solution can be complex.

   Alternatively, you could create a custom script that checks user confirmation before allowing SSH access.

2. **Example of a Custom Check:**
   Create a script that checks the user’s confirmation status before allowing SSH login:
   ```bash
   sudo nano /usr/local/bin/check_confirmation.sh
   ```

   Add the following:
   ```bash
   #!/bin/bash
   username=$1
   confirmed=$(mysql -u db_user -p'db_pass' -D db_name -se "SELECT confirmed FROM users WHERE email='$username'")

   if [ "$confirmed" != "1" ]; then
       echo "User not confirmed"
       exit 1
   fi

   exit 0
   ```

   Make it executable:
   ```bash
   sudo chmod +x /usr/local/bin/check_confirmation.sh
   ```

3. **Modify SSHD Configuration:**
   Edit the SSH daemon configuration:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

   Add the following line to the end:
   ```bash
   ForceCommand /usr/local/bin/check_confirmation.sh
   ```

4. **Restart SSH Service:**
   ```bash
   sudo systemctl restart ssh
   ```

### Step 6: Testing
1. **Register a User:**
   Access `register.php` through your web browser and enter an email to receive a confirmation link.

2. **Confirm the Email:**
   Click the link in the confirmation email. This should set the user’s confirmation status in the database.

3. **Attempt to SSH Login:**
   Try logging in via SSH with the registered email. If confirmation has not been completed, access should be denied.

### Notes
- This is a basic implementation and should be expanded for production use (e.g., add input validation, better error handling, secure the PHP scripts).
- Ensure your server has proper firewall settings to allow SSH traffic.
- Regularly update your software and manage user accounts securely.

### Conclusion
You now have an SSH server set up on Ubuntu that requires email confirmation for user logins. Be sure to test and validate the security measures in place.
Always remember to keep your system updated and regularly review your security practices.
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



