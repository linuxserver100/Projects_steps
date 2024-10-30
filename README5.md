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



