To set up email-based OTP (One-Time Password) login for SSH on an Ubuntu server, you'll need to set up a combination of tools, like **Flask** (a Python web framework) for generating and sending OTPs via email, and configuring SSH for PAM (Pluggable Authentication Modules) to use OTPs. Here’s a basic guide to help you get started:

### Step 1: Install Dependencies

You'll need **Python** and some packages to handle OTP generation and email sending. Start by installing the required packages.

```bash
sudo apt update
sudo apt install python3-pip python3-venv libpam-python
pip3 install flask pyotp smtplib
```

### Step 2: Create a Flask App for OTP Generation and Email Sending

1. Create a directory for your Flask application, such as `otp-server`.
   ```bash
   mkdir otp-server && cd otp-server
   ```
2. Create a Python virtual environment.
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```
3. Create a Flask app (`otp_server.py`) to generate and send OTPs.

```python
# otp_server.py
from flask import Flask, request, jsonify
import pyotp
import smtplib
from email.mime.text import MIMEText

app = Flask(__name__)
totp = pyotp.TOTP("base32secret3232")  # Replace this with a secure, random secret key

@app.route('/send_otp', methods=['POST'])
def send_otp():
    email = request.json.get('email')
    otp = totp.now()

    msg = MIMEText(f"Your OTP is: {otp}")
    msg['Subject'] = 'Your OTP Code'
    msg['From'] = 'your_email@example.com'
    msg['To'] = email

    try:
        with smtplib.SMTP('smtp.example.com', 587) as server:
            server.starttls()
            server.login("your_email@example.com", "your_email_password")
            server.sendmail("your_email@example.com", email, msg.as_string())
        return jsonify({"message": "OTP sent successfully"}), 200
    except Exception as e:
        return jsonify({"error": str(e)}), 500

@app.route('/verify_otp', methods=['POST'])
def verify_otp():
    otp = request.json.get('otp')
    if totp.verify(otp):
        return jsonify({"message": "OTP verified"}), 200
    else:
        return jsonify({"error": "Invalid OTP"}), 400

if __name__ == '__main__':
    app.run(port=5000)
```

Replace `"base32secret3232"`, SMTP server, and email credentials with your actual information.

### Step 3: Start the Flask App

```bash
flask run --port 5000
```

Ensure this is running in the background and accessible on the server.

### Step 4: Configure PAM for SSH

1. Open the PAM SSH configuration file.

   ```bash
   sudo nano /etc/pam.d/sshd
   ```

2. Add a PAM script to check OTPs via the Flask API. This requires a custom PAM module.

Create `/usr/share/pam-python/otp_pam.py`:

```python
# otp_pam.py
import requests

def pam_sm_authenticate(pamh, flags, argv):
    try:
        otp = pamh.conversation(pamh.Message(pamh.PAM_PROMPT_ECHO_OFF, "Enter OTP: ")).resp
        response = requests.post("http://127.0.0.1:5000/verify_otp", json={"otp": otp})
        if response.json().get("message") == "OTP verified":
            return pamh.PAM_SUCCESS
        else:
            return pamh.PAM_AUTH_ERR
    except Exception:
        return pamh.PAM_AUTH_ERR
```

3. Add the PAM module to `/etc/pam.d/sshd`:

   ```bash
   auth required pam_python.so /usr/share/pam-python/otp_pam.py
   ```

### Step 5: Configure SSH to Use PAM

1. Open the SSH daemon configuration file.

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Ensure the following lines are set:

   ```bash
   ChallengeResponseAuthentication yes
   UsePAM yes
   ```

3. Restart SSH:

   ```bash
   sudo systemctl restart ssh
   ```

### Step 6: Test OTP Login

1. Connect to your server via SSH. You should be prompted for an OTP after entering your password.
2. The Flask app should handle the OTP generation and email delivery.

This setup combines **Flask** for generating/sending OTPs, **pyotp** for OTP validation, and PAM scripting for SSH login integration.

🙃😙😘☺️☺️👇🤪👇☺️👇☺️👇😃👇😃☺️👇😀👇😃👇🙃👇😃🥲😙😝☺️🤩😝😙😝😝😃😊😝😙😝🙃😝😊😝😊😝😙😝😃😙😝😊😝😙😝😃😝🙃🥲😃🥲😃🥲😃🥲😁😁🥲😊🥲😊🥲😄🥲😄😄🥲☺️🥲☺️🥲😄🥲😊🥲😊🥲😄🥲😙🥲😙😌😃😌😙😌😙😙😌😙😌😊😌😊😌😙
To set up email OTP (One-Time Password) login on an Ubuntu server, you can use `PAM (Pluggable Authentication Module)` with a tool like `oathtool` to generate OTPs and send them via email. Here's a step-by-step guide:

### Step 1: Install Necessary Packages
1. **Install `oathtool`** (for generating OTPs) and **Postfix** (for sending emails). Replace `Postfix` with your preferred mail sending service if needed.

   ```bash
   sudo apt update
   sudo apt install libpam-oath oathtool postfix mailutils -y
   ```

### Step 2: Configure OATH
1. **Create a directory** for storing user OTP secrets.

   ```bash
   sudo mkdir /etc/libpam-oath
   sudo chmod 700 /etc/libpam-oath
   ```

2. **Create an OATH file** to store user secrets.

   ```bash
   sudo nano /etc/libpam-oath/users.oath
   ```

   Inside this file, add each user in this format:
   ```
   HOTP/T30/6 username - hex-secret
   ```

   - Replace `username` with the actual username.
   - Replace `hex-secret` with a generated secret. To create one, use:

     ```bash
     oathtool --verbose --totp <your_secret>
     ```

   This generates a TOTP secret.

3. **Set proper permissions**:

   ```bash
   sudo chmod 600 /etc/libpam-oath/users.oath
   sudo chown root:root /etc/libpam-oath/users.oath
   ```

### Step 3: Configure PAM to Use OTP
1. **Edit the PAM configuration** file for SSH.

   ```bash
   sudo nano /etc/pam.d/sshd
   ```

   Add the following line at the top of the file to enable PAM for OTP:

   ```text
   auth required pam_oath.so usersfile=/etc/libpam-oath/users.oath window=30 digits=6
   ```

2. **Edit the SSH configuration** to use password and OTP.

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

   Set these options:

   ```text
   ChallengeResponseAuthentication yes
   UsePAM yes
   ```

   **Restart SSH** for changes to take effect:

   ```bash
   sudo systemctl restart ssh
   ```

### Step 4: Set Up an Email Alert Script
1. **Create a script** to send the OTP to the user’s email.

   ```bash
   sudo nano /usr/local/bin/send-otp-email.sh
   ```

   Add the following content (update email settings as needed):

   ```bash
   #!/bin/bash
   USER=$1
   OTP=$2
   EMAIL="user@example.com"  # Replace with the actual user's email

   echo "Your OTP code is: $OTP" | mail -s "Your OTP Code" $EMAIL
   ```

2. **Make the script executable**:

   ```bash
   sudo chmod +x /usr/local/bin/send-otp-email.sh
   ```

3. **Modify the PAM configuration** to call this script.

   Edit the `/etc/pam.d/sshd` file again, adding the following line below the `pam_oath` line:

   ```text
   auth optional pam_exec.so /usr/local/bin/send-otp-email.sh
   ```

### Step 5: Test the Setup
1. Attempt to SSH into the server. After entering your password, you should receive an OTP via email.
2. Enter the OTP to complete the login.

### Note
Make sure to test carefully on a non-production server first, as locking yourself out is possible.

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



