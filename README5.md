To set up SSH login on an Ubuntu server with email OTP (One-Time Password) authentication without modifying PAM or `.bashrc`, we can follow these general steps. Here, we'll use Google Authenticator (or `libpam-google-authenticator`) to generate OTP codes and use a Python script to send them via email. These instructions assume you already have SSH access to your server.

### Prerequisites
- Ensure Python is installed on your server (for sending the email).
- A working SMTP server for sending emails (like Gmail SMTP).
- `libpam-google-authenticator` package installed for OTP generation.

### Steps

1. **Install Required Packages**
   ```bash
   sudo apt update
   sudo apt install -y libpam-google-authenticator python3-pip
   pip3 install yagmail pyotp
   ```

2. **Generate OTP Secret with Google Authenticator**

   Run the following command for the user account you want to configure OTP for (e.g., your own user account).
   ```bash
   google-authenticator
   ```
   - Answer the prompts according to your preference.
   - Record the secret key or QR code generated. This will be used to generate OTPs.
   - When asked if you'd like to "disallow multiple uses of the same authentication token," answer "yes."

   This will generate a `~/.google_authenticator` file that holds the OTP secret and configuration.

3. **Create a Python Script to Send OTPs via Email**

   Create a new script called `send_otp.py` in your user's home directory or another convenient location.
   ```bash
   nano ~/send_otp.py
   ```

   Add the following code to the script:
   ```python
   import yagmail
   import pyotp
   import sys

   # Email configuration
   EMAIL = "your_email@gmail.com"      # Sender email
   APP_PASSWORD = "your_app_password"  # App password (use Google app password if using Gmail)
   RECIPIENT_EMAIL = "recipient_email@gmail.com"  # Recipient email for OTP

   # Initialize OTP generator
   otp_secret = "YOUR_OTP_SECRET"  # Replace this with the secret from Google Authenticator
   totp = pyotp.TOTP(otp_secret)

   # Generate OTP
   otp = totp.now()

   # Send OTP via email
   yag = yagmail.SMTP(EMAIL, APP_PASSWORD)
   subject = "Your SSH One-Time Password"
   contents = f"Your OTP is: {otp}"
   yag.send(RECIPIENT_EMAIL, subject, contents)

   print("OTP sent successfully.")
   ```

   - Replace `your_email@gmail.com` with the sender email address.
   - Replace `your_app_password` with an app-specific password (if using Gmail or other secure email).
   - Replace `recipient_email@gmail.com` with the email that should receive the OTP.
   - Replace `YOUR_OTP_SECRET` with the secret from Google Authenticator (from Step 2).

   Save and exit the file.

4. **Make the Script Executable**

   ```bash
   chmod +x ~/send_otp.py
   ```

5. **Configure SSH to Use the OTP for Authentication**

   Update the `~/.ssh/authorized_keys` file for the user with a command to verify OTP before allowing SSH access.

   ```bash
   nano ~/.ssh/authorized_keys
   ```

   For each SSH key in this file, prepend it with a command to run `send_otp.py` and prompt for OTP verification. For example:
   ```bash
   command="~/send_otp.py && read -p 'Enter OTP: ' input_otp && python3 -c 'import pyotp; print(pyotp.TOTP(\"YOUR_OTP_SECRET\").verify(input_otp))' | grep -q '^True$'" ssh-rsa AAAAB3Nza...
   ```

   - Replace `"YOUR_OTP_SECRET"` in the command with your actual OTP secret.
   - The `command` directive:
     - Sends the OTP email.
     - Prompts the user to enter the OTP during SSH login.
     - Verifies the OTP using `pyotp` (requires it to match the current OTP).

6. **Test the SSH Login**

   From a client machine, try to SSH into the server:
   ```bash
   ssh username@your-server-ip
   ```

   You should:
   - Receive an OTP via email.
   - Be prompted to enter the OTP.
   - Gain access only if the OTP is correct.

This setup provides email-based OTP without modifying PAM or `.bashrc` files, though it relies on Google Authenticator and a custom Python script. Adjustments can be made to enhance security or customize email content further.
