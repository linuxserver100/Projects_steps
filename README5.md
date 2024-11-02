To set up SSH login on Ubuntu with email OTP (One-Time Password) verification, we can use a method that involves a combination of SSH configuration and a simple script to send an OTP via email. Here’s a detailed guide, step by step:

### Prerequisites

1. **Ubuntu server** (with SSH access).
2. **Email account**: An email account configured to send emails via SMTP. This could be a Gmail account or any other email provider.
3. **Python 3**: The system should have Python 3 installed (most Ubuntu installations have it by default).

### Steps to Set Up Email OTP Verification for SSH

#### Step 1: Configure SSH

1. **Open SSH Configuration**:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
   
2. **Modify SSH Settings**:
   Ensure the following lines are present and uncommented:
   ```plaintext
   PasswordAuthentication yes
   ChallengeResponseAuthentication yes
   ```

3. **Restart SSH Service**:
   ```bash
   sudo systemctl restart ssh
   ```

#### Step 2: Create an OTP Generation and Email Script

1. **Install Required Packages** (if not installed):
   Make sure you have `mailutils` for sending emails:
   ```bash
   sudo apt-get install mailutils
   ```

2. **Create a Script**:
   Create a script that generates an OTP and sends it via email. Create a new file:
   ```bash
   sudo nano /usr/local/bin/send_otp.py
   ```

3. **Add the Following Python Script**:
   ```python
   #!/usr/bin/env python3

   import smtplib
   from email.mime.text import MIMEText
   import random
   import sys

   # Email Configuration
   EMAIL_ADDRESS = 'your_email@gmail.com'  # Change this to your email
   EMAIL_PASSWORD = 'your_email_password'   # Change this to your email password
   TO_EMAIL = 'recipient_email@example.com'  # Change this to the recipient's email (for OTP)

   # Generate OTP
   otp = random.randint(100000, 999999)

   # Send Email
   msg = MIMEText(f'Your OTP is: {otp}')
   msg['Subject'] = 'Your OTP Code'
   msg['From'] = EMAIL_ADDRESS
   msg['To'] = TO_EMAIL

   try:
       with smtplib.SMTP('smtp.gmail.com', 587) as server:
           server.starttls()
           server.login(EMAIL_ADDRESS, EMAIL_PASSWORD)
           server.sendmail(EMAIL_ADDRESS, TO_EMAIL, msg.as_string())
           print(otp)  # Print OTP for later verification
   except Exception as e:
       print(f"Error sending email: {e}")
       sys.exit(1)
   ```

   Replace:
   - `your_email@gmail.com` with your email address.
   - `your_email_password` with your email password (consider using an app password if using Gmail).
   - `recipient_email@example.com` with the email where you want to receive the OTP.

4. **Make the Script Executable**:
   ```bash
   sudo chmod +x /usr/local/bin/send_otp.py
   ```

#### Step 3: Create the OTP Verification Mechanism

1. **Create a Verification Script**:
   Create another script to handle the OTP verification. 
   ```bash
   sudo nano /usr/local/bin/verify_otp.py
   ```

2. **Add the Following Python Script**:
   ```python
   #!/usr/bin/env python3

   import sys

   # The OTP should be stored in a temporary file or some state
   with open('/tmp/otp.txt', 'r') as f:
       otp = f.read().strip()

   # Input from the user
   user_input = input('Enter the OTP sent to your email: ')

   if user_input == otp:
       print("OTP Verified Successfully!")
       sys.exit(0)
   else:
       print("Invalid OTP!")
       sys.exit(1)
   ```

3. **Make the Script Executable**:
   ```bash
   sudo chmod +x /usr/local/bin/verify_otp.py
   ```

#### Step 4: Modify SSH to Use the OTP System

1. **Create a PAM Configuration File**:
   Create a new PAM module configuration file:
   ```bash
   sudo nano /etc/pam.d/sshd
   ```

2. **Add the Following Lines**:
   ```plaintext
   auth required pam_exec.so /usr/local/bin/send_otp.py
   auth required pam_exec.so /usr/local/bin/verify_otp.py
   ```

#### Step 5: Store OTP in a Temporary File

Modify the `send_otp.py` script to store the OTP in a temporary file:

```python
# After generating the OTP
with open('/tmp/otp.txt', 'w') as f:
    f.write(str(otp))
```

Add this line right after generating the OTP (just before sending the email).

#### Step 6: Testing the Setup

1. **Reboot the System**:
   ```bash
   sudo reboot
   ```

2. **Log in via SSH**:
   - Connect to your server via SSH.
   - You should receive an email with the OTP.
   - Enter the OTP when prompted.

### Step 7: Make the System Start Automatically After Reboot

The above scripts will run automatically due to their configuration in PAM and the SSH service. The scripts do not need to be run on startup because PAM handles the execution during the SSH login process.

### Important Notes

- **Security**: Make sure to secure your email credentials and consider using environment variables or a more secure method for storing them.
- **Email Configuration**: If you're using Gmail, you might need to allow "less secure apps" or set up an App Password for better security.
- **File Permissions**: Ensure that the `/tmp/otp.txt` file is secured properly, as it contains sensitive information.

### Final Thoughts

This setup provides a basic method for OTP verification via email for SSH logins on Ubuntu. Depending on your security needs, consider further enhancements such as rate limiting, logging attempts, and possibly implementing a more robust OTP system using libraries or services designed for this purpose.
