Configuring email-based One-Time Passwords (OTP) for SSH login on an Ubuntu server involves several steps. Below is a guide to help you set it up:

### Prerequisites
- An Ubuntu server with SSH access.
- An email account that will be used to send the OTPs (e.g., Gmail).
- `sendmail` or another MTA (Mail Transfer Agent) installed for sending emails.

### Step 1: Install Required Packages
First, ensure that you have the necessary packages installed. Open a terminal and run:

```bash
sudo apt update
sudo apt install mailutils python3-pip
```

### Step 2: Install `oathtool`
You’ll need `oathtool` to generate the OTPs:

```bash
sudo apt install oathtool
```

### Step 3: Create a Python Script to Send OTP
Create a Python script that will generate and send the OTP via email.

1. Create a new directory for the script:

   ```bash
   mkdir ~/otp
   cd ~/otp
   ```

2. Create the script `send_otp.py`:

   ```bash
   nano send_otp.py
   ```

   Paste the following code into the file:

   ```python
   #!/usr/bin/env python3

   import smtplib
   from email.mime.text import MIMEText
   from email.mime.multipart import MIMEMultipart
   import secrets
   import sys

   def send_email(to_email, otp):
       from_email = 'your_email@gmail.com'
       password = 'your_email_password'
       
       subject = "Your OTP Code"
       body = f"Your OTP code is: {otp}"

       msg = MIMEMultipart()
       msg['From'] = from_email
       msg['To'] = to_email
       msg['Subject'] = subject

       msg.attach(MIMEText(body, 'plain'))

       try:
           server = smtplib.SMTP('smtp.gmail.com', 587)
           server.starttls()
           server.login(from_email, password)
           server.sendmail(from_email, to_email, msg.as_string())
           server.quit()
           print("OTP sent successfully")
       except Exception as e:
           print(f"Failed to send OTP: {e}")

   if __name__ == "__main__":
       if len(sys.argv) != 2:
           print("Usage: send_otp.py <email>")
           sys.exit(1)

       email = sys.argv[1]
       otp = secrets.token_hex(4)  # Generate a 8-character OTP
       send_email(email, otp)
       print(otp)  # Print OTP for verification
   ```

3. Replace `your_email@gmail.com` and `your_email_password` with your actual email credentials. If you're using Gmail, make sure to allow "Less secure apps" or use an App Password if you have 2FA enabled.

4. Make the script executable:

   ```bash
   chmod +x send_otp.py
   ```

### Step 4: Configure SSH
1. Open the SSH configuration file:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Add the following line to enable challenge-response authentication:

   ```bash
   ChallengeResponseAuthentication yes
   ```

3. Save the file and exit.

4. Restart the SSH service:

   ```bash
   sudo systemctl restart sshd
   ```

### Step 5: Set Up PAM to Use OTP
1. Open the PAM SSH configuration file:

   ```bash
   sudo nano /etc/pam.d/sshd
   ```

2. Add the following line at the end:

   ```bash
   auth required pam_exec.so /path/to/otp_script.sh
   ```

### Step 6: Create OTP Script
Create the script that will handle the OTP generation and verification.

1. Create the OTP script:

   ```bash
   sudo nano /usr/local/bin/otp_script.sh
   ```

   Paste the following code into the file:

   ```bash
   #!/bin/bash

   EMAIL="$PAM_USER@example.com"  # Replace with a method to fetch user email

   # Generate OTP and send email
   OTP=$(python3 /path/to/send_otp.py "$EMAIL")

   echo "Please check your email for the OTP."
   read -p "Enter the OTP: " input_otp

   if [ "$input_otp" == "$OTP" ]; then
       exit 0  # Successful authentication
   else
       exit 1  # Failed authentication
   fi
   ```

2. Make the script executable:

   ```bash
   sudo chmod +x /usr/local/bin/otp_script.sh
   ```

### Step 7: Test the Configuration
- Try logging in via SSH. You should receive an OTP via email and be prompted to enter it.
- If the OTP is correct, you should gain access. If not, the login will be denied.

### Important Notes
- Ensure that your email server settings allow sending emails from the script.
- For production environments, consider using a more secure and robust solution for OTP, like Google Authenticator.
- Always test configurations in a safe environment before deploying them to production.

This setup should provide a basic email-based OTP authentication mechanism for SSH access on your Ubuntu server. Adjust the scripts and configurations as necessary for your specific requirements.
