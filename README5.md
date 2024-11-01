
Setting up SSH login on Ubuntu with email OTP (One-Time Password) verification can enhance your server's security. Below is a step-by-step guide to accomplish this without errors:

### Prerequisites
- An Ubuntu server with SSH access.
- A domain or email service to send OTPs.
- Python and the `smtplib` library for sending emails (often pre-installed).

### Step 1: Install Required Packages
First, make sure your system is updated and install the necessary packages.

```bash
sudo apt update
sudo apt install python3 python3-pip
pip3 install pyotp
```

### Step 2: Create a Python Script to Send OTP
Create a Python script that generates and sends the OTP to your email.

1. Create a directory for the script:
   ```bash
   mkdir ~/otp_script
   cd ~/otp_script
   ```

2. Create the script file:
   ```bash
   nano send_otp.py
   ```

3. Add the following code to `send_otp.py`:

   ```python
   import smtplib
   import pyotp
   from email.mime.text import MIMEText

   # Configuration
   EMAIL_ADDRESS = "your_email@example.com"
   EMAIL_PASSWORD = "your_email_password"
   TO_EMAIL = "recipient@example.com"  # Your own email or the user's email

   # Generate OTP
   totp = pyotp.TOTP('base32secret3232')  # Replace with a unique base32 secret
   otp = totp.now()

   # Email Content
   subject = "Your OTP Code"
   body = f"Your OTP code is: {otp}"

   # Send Email
   msg = MIMEText(body)
   msg['Subject'] = subject
   msg['From'] = EMAIL_ADDRESS
   msg['To'] = TO_EMAIL

   with smtplib.SMTP('smtp.gmail.com', 587) as server:
       server.starttls()
       server.login(EMAIL_ADDRESS, EMAIL_PASSWORD)
       server.send_message(msg)

   print("OTP sent to email.")
   ```

4. Replace `your_email@example.com`, `your_email_password`, and `recipient@example.com` with your actual email credentials.

5. Save and exit the editor (Ctrl + X, then Y, then Enter).

### Step 3: Set Up SSH to Use OTP
1. Open the SSH configuration file:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Find and set the following options:
   ```
   ChallengeResponseAuthentication yes
   PasswordAuthentication no
   ```

3. Save and exit the editor.

### Step 4: Restart SSH Service
To apply the changes, restart the SSH service:
```bash
sudo systemctl restart sshd
```

### Step 5: Create a Script for OTP Verification
1. Create a new script for OTP verification:
   ```bash
   nano ~/otp_verification.sh
   ```

2. Add the following content:

   ```bash
   #!/bin/bash

   # Generate OTP and send it
   python3 ~/otp_script/send_otp.py

   # Prompt user for OTP
   read -p "Enter the OTP sent to your email: " user_otp

   # Validate the OTP
   totp=$(python3 -c "import pyotp; print(pyotp.TOTP('base32secret3232').now())")

   if [ "$user_otp" == "$totp" ]; then
       echo "OTP verified! You are allowed to log in."
       exit 0
   else
       echo "Invalid OTP. Access denied."
       exit 1
   fi
   ```

3. Save and exit the editor.

4. Make the script executable:
   ```bash
   chmod +x ~/otp_verification.sh
   ```

### Step 6: Update PAM Configuration
1. Edit the PAM configuration for SSH:
   ```bash
   sudo nano /etc/pam.d/sshd
   ```

2. Add the following line at the end of the file:
   ```
   auth required pam_exec.so /path/to/your/otp_verification.sh
   ```
   Replace `/path/to/your/otp_verification.sh` with the actual path, e.g., `/home/yourusername/otp_verification.sh`.

3. Save and exit the editor.

### Step 7: Test the Setup
1. Open a new terminal and attempt to SSH into your server:
   ```bash
   ssh your_username@your_server_ip
   ```

2. You should receive an OTP in your email. Enter it when prompted.

### Conclusion
This setup will require the user to enter an OTP sent to their email whenever they try to log in via SSH. Be sure to test the entire process and adjust email server settings as needed. Always ensure your email service allows sending emails programmatically. 

If you encounter any issues, check the logs for errors:
```bash
sudo journalctl -u sshd
```

### Security Notes
- Use a dedicated email account for this purpose.
- Consider using environment variables for sensitive information like email passwords.
- Ensure that your server and scripts are kept secure to avoid unauthorized access.
- 😛😅☺️😛😜🥹🔑☺️🔑🥹😊🔑😅😊😅🔑😅😊😅😊🔑🥹🔑🥹🥰😄😄🔑🥰🔑😄🥰😝🔑🥰🔑😄🥰🔑😄🥰🔑😄🥰😝😝🔑☺️🔑😝😝🥴😍😍😝😂😍🥴😝😍😝🤣🥴🥴😍😝😍🥴😝😍🥴😝😍🥴😝😍😍😜🥰🥴😝🥰🤣😝😝🤣😍😍🤣😝🥰🤣😝🥰🔑😝🥰😍😝🔑😝🔑😍🔑😝🔑😊🔑😝☺️😝☺️😜😂😍🥴😝☺️😜😍🥴😜😍😂😝😍😂😝😍😂😝☺️😝😂😍😝🥴😍😂😝😊
Setting up SSH login with SMS OTP (One-Time Password) verification on an Ubuntu server involves several steps. This setup improves security by requiring a second form of authentication beyond just a password. Below are the detailed steps to configure this using `Google Authenticator`, which can send OTPs via SMS. 

### Prerequisites

1. **Ubuntu Server**: Ensure you have an Ubuntu server (version 18.04 or later).
2. **Root or Sudo Access**: You will need root privileges or sudo access to install packages and configure settings.
3. **SMS Gateway**: You will need an SMS gateway (like Twilio, Nexmo, etc.) that can send SMS messages via API.

### Steps to Set Up SSH with SMS OTP Verification

#### Step 1: Install Required Packages

1. **Update your package list**:

   ```bash
   sudo apt update
   ```

2. **Install `libpam-google-authenticator`**:

   ```bash
   sudo apt install libpam-google-authenticator
   ```

#### Step 2: Configure Google Authenticator

1. **Run the Google Authenticator setup**:

   ```bash
   google-authenticator
   ```

2. You will be prompted with several questions:

   - **Do you want authentication tokens to be time-based?** (Answer `y`)
   - This will generate a QR code and a secret key. You can use a TOTP app (like Google Authenticator) to scan the QR code or save the secret key.

3. **Backup the QR code and secret key** somewhere safe. 

4. **Save the emergency scratch codes** provided; these are important if you lose access to your TOTP app.

#### Step 3: Configure PAM for SSH

1. **Edit the PAM configuration file for SSH**:

   ```bash
   sudo nano /etc/pam.d/sshd
   ```

2. **Add the following line at the top**:

   ```plaintext
   auth required pam_google_authenticator.so
   ```

3. **Save and exit** (Ctrl + X, then Y, then Enter).

#### Step 4: Configure SSHD

1. **Edit the SSH daemon configuration file**:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. **Modify the following parameters**:

   ```plaintext
   ChallengeResponseAuthentication yes
   PasswordAuthentication yes
   ```

3. **You may also want to set the following for additional security**:

   ```plaintext
   UsePAM yes
   ```

4. **Save and exit** (Ctrl + X, then Y, then Enter).

#### Step 5: Restart SSH Service

To apply the changes, restart the SSH service:

```bash
sudo systemctl restart sshd
```

#### Step 6: Setting Up SMS OTP

To send OTPs via SMS, you'll need to set up an SMS gateway.

1. **Choose an SMS service provider** (e.g., Twilio, Nexmo) and sign up.
2. **Get your API credentials** (API key, API secret, and phone number).
3. **Install a tool like `curl` if it’s not already installed**:

   ```bash
   sudo apt install curl
   ```

4. **Create a script to send SMS**. Create a file `send_sms.sh`:

   ```bash
   nano ~/send_sms.sh
   ```

   Add the following script (example for Twilio):

   ```bash
   #!/bin/bash

   # Twilio API credentials
   ACCOUNT_SID="your_account_sid"
   AUTH_TOKEN="your_auth_token"
   FROM_NUMBER="your_twilio_number"
   TO_NUMBER="$1"
   OTP="$2"

   curl -X POST "https://api.twilio.com/2010-04-01/Accounts/$ACCOUNT_SID/Messages.json" \
   --data-urlencode "Body=Your OTP is: $OTP" \
   --data-urlencode "From=$FROM_NUMBER" \
   --data-urlencode "To=$TO_NUMBER" \
   -u "$ACCOUNT_SID:$AUTH_TOKEN"
   ```

5. **Make the script executable**:

   ```bash
   chmod +x ~/send_sms.sh
   ```

#### Step 7: Integrate SMS with Google Authenticator

You will need to modify your `~/.google_authenticator` file to send an OTP via SMS. This will require modifying the PAM configuration to trigger the SMS script after generating an OTP.

1. **Edit PAM configuration** again:

   ```bash
   sudo nano /etc/pam.d/sshd
   ```

   Add a line to run the SMS script:

   ```plaintext
   auth required pam_exec.so /path/to/your/send_sms.sh $USER $(google-authenticator -g)
   ```

   (Ensure that the Google Authenticator app is already installed and properly configured.)

#### Step 8: Test Your Setup

1. **Log out of your SSH session**.
2. **Attempt to log back in**. You should first enter your password and then receive an SMS with the OTP.
3. **Enter the OTP to gain access**.

### Troubleshooting Tips

- **Check your SMS gateway settings** if you do not receive SMS.
- **Check logs**: Look at `/var/log/auth.log` for any authentication errors.
- Ensure your SMS script has the correct permissions to execute and access network resources.

### Conclusion

With these steps, you should have a functional SSH setup with SMS OTP verification for added security. Always ensure your SMS gateway is configured properly to avoid issues with receiving OTPs.
- 



☺️☺️🤪😍☺️😍🤪😅🥰🤪😅🥰😊🤪😅😊🤪😅😊😅😊😅🥰🤩😅😊😅🤩😅😅😅😅☺️😅😅😊😊😅😊😅😊😊😅🤩😊😊😍😊😅😊😅☺️😅☺️😅🤩😊😍🤩😅😊🥴🤩☺️☺️🤩😍☺️😍🤩☺️🤩😍☺️🤩😅☺️🤩😅☺️😅🤩😊😅☺️😅😊😅😊😊😅😊🥹🤩😅🤩😊😊😊😅😊😅🤩😊🥹😊🤪🥹😊😊😍🤩😊🤩😍😊😂🤩😂😊😍😊🤪🥹😊😅😊🥹😊😅🤩😊🤩🥹😊😅☺️😅😊😅☺️😅☺️😅😝😅😜😜😅😝😅
Setting up SSH login with email link verification on an Ubuntu server involves several steps. Here’s a detailed guide to achieve this using **SSH** with **One-Time Passwords (OTP)** via email, using tools like **`ssmtp`** and **`google-authenticator`**. 

### Prerequisites
1. **Ubuntu Server**: Ensure you have a server running Ubuntu.
2. **SSH Access**: You need SSH access to the server.
3. **Email Account**: An email account to send verification links (like Gmail).
4. **Install Required Packages**: Make sure you have `mailutils` (or `ssmtp` for Gmail) and `google-authenticator` installed.

### Step-by-Step Setup

#### Step 1: Update Your System
First, update your package list and upgrade installed packages:
```bash
sudo apt update
sudo apt upgrade -y
```

#### Step 2: Install Required Packages
Install the necessary packages for sending emails and OTP generation:
```bash
sudo apt install mailutils libpam-google-authenticator -y
```

#### Step 3: Configure Google Authenticator
Run the `google-authenticator` command for the user you want to set up for SSH access. This will create a secret key, emergency scratch codes, and a QR code for OTP.
```bash
google-authenticator
```
- Answer the prompts:
  - **Do you want authentication tokens to be time-based (y/n)**? **y**
  - Make sure to save the emergency scratch codes.
  - You’ll see a QR code, scan it with your OTP app (like Google Authenticator).

#### Step 4: Configure PAM (Pluggable Authentication Module)
Edit the PAM configuration for SSH to include the Google Authenticator module:
```bash
sudo nano /etc/pam.d/sshd
```
Add the following line at the top:
```
auth required pam_google_authenticator.so
```

#### Step 5: Configure SSH Daemon
Now, modify the SSH configuration to allow challenge-response authentication:
```bash
sudo nano /etc/ssh/sshd_config
```
Make sure to set the following options:
```
ChallengeResponseAuthentication yes
PasswordAuthentication no
UsePAM yes
```
- Optionally, you may want to change `PermitRootLogin` to `no` for security.

#### Step 6: Restart SSH Service
After making the changes, restart the SSH service:
```bash
sudo systemctl restart ssh
```

#### Step 7: Configure Email Sending
If you are using Gmail to send emails, configure `ssmtp`:
1. Install `ssmtp`:
    ```bash
    sudo apt install ssmtp -y
    ```
2. Edit the `ssmtp` configuration:
    ```bash
    sudo nano /etc/ssmtp/ssmtp.conf
    ```
    Add or modify the following lines (use your actual Gmail credentials):
    ```
    root=your_email@gmail.com
    mailhub=smtp.gmail.com:587
    AuthUser=your_email@gmail.com
    AuthPass=your_email_password
    UseSTARTTLS=YES
    ```
3. Enable “Less secure app access” for your Gmail account.

#### Step 8: Script for Email Verification
Create a simple script to send the email verification link (you can customize the link):
```bash
sudo nano /usr/local/bin/send_verification_email.sh
```
Add the following content:
```bash
#!/bin/bash
TO="$1"
SUBJECT="SSH Login Verification"
BODY="Click the link to verify your SSH login: http://your-server-ip/verify?token=$2"
echo -e "Subject:$SUBJECT\n\n$BODY" | ssmtp $TO
```
Make the script executable:
```bash
sudo chmod +x /usr/local/bin/send_verification_email.sh
```

#### Step 9: Integrate Email Verification with SSH
Edit the `~/.ssh/authorized_keys` for the user, prepend the command to the public key to run your email script:
```bash
command="/usr/local/bin/send_verification_email.sh your_email@gmail.com $(google-authenticator -o -t -d -s /path/to/your/secret/file)" ssh-rsa AAAAB3... user@host
```

### Testing
1. Try to SSH into your server:
   ```bash
   ssh your_user@your_server_ip
   ```
2. Check your email for the verification link.
3. Click the link to verify.

### Important Notes
- Ensure your server allows outbound connections to the SMTP server (e.g., Gmail).
- For production use, consider using secure email handling and proper user management.
- Regularly update your server for security.

### Conclusion
You have now configured SSH login on Ubuntu with email link verification using OTP. Ensure to test thoroughly to confirm that the entire process works as expected.

