Setting up SSH login with One-Time Password (OTP) verification on an Ubuntu system typically requires additional software, like `Google Authenticator`, to manage the OTP process. However, if you want to create a simple OTP mechanism using a shell script and SMTP configuration without installing additional packages, I can guide you through that. 

**Disclaimer:** This method is more of a workaround and may not be as secure or robust as using dedicated tools like `Google Authenticator`.

### Prerequisites
1. **SSH Access**: You need to have SSH access to your Ubuntu machine.
2. **Email Account**: You need access to an SMTP server to send OTPs via email.

### Steps

#### Step 1: Create the OTP Generation Script

1. **Open your terminal** and create a script that will generate the OTP and send it via email.

   ```bash
   nano ~/otp_script.sh
   ```

2. **Add the following content** to the script. Replace the placeholder values for SMTP configuration with your actual SMTP details:

   ```bash
   #!/bin/bash

   # SMTP Configuration
   SMTP_SERVER="smtp.example.com"
   SMTP_PORT="587"
   SMTP_USER="your_email@example.com"
   SMTP_PASS="your_password"
   RECIPIENT_EMAIL="recipient@example.com"

   # Generate a 6-digit OTP
   OTP=$(shuf -i 100000-999999 -n 1)

   # Send OTP via Email
   echo "Your OTP is: $OTP" | sendmail -S $SMTP_SERVER:$SMTP_PORT -au$SMTP_USER -ap$SMTP_PASS $RECIPIENT_EMAIL

   # Store OTP in a file for later verification
   echo $OTP > /tmp/otp_value.txt
   ```

   - This script generates a random 6-digit OTP, sends it via email using `sendmail`, and stores it in a temporary file.

3. **Make the script executable**:

   ```bash
   chmod +x ~/otp_script.sh
   ```

#### Step 2: Configure the `.bashrc` for OTP Verification

1. **Open your `.bashrc` file**:

   ```bash
   nano ~/.bashrc
   ```

2. **Add the following lines** at the end of the file to call the OTP script after login:

   ```bash
   # Call OTP Script
   ~/otp_script.sh

   # Prompt for OTP
   read -p "Enter the OTP sent to your email: " input_otp

   # Read the stored OTP
   stored_otp=$(cat /tmp/otp_value.txt)

   # Verify OTP
   if [ "$input_otp" != "$stored_otp" ]; then
       echo "Invalid OTP. Access denied."
       exit 1
   else
       echo "OTP verified. Welcome!"
   fi
   ```

   - This will execute the OTP script upon SSH login, prompt the user to enter the OTP received via email, and verify it against the stored OTP.

#### Step 3: Set Up Sendmail (If Not Already Installed)

1. If `sendmail` is not installed, you will need to install it (there's no way to send emails without an SMTP client). Install it using:

   ```bash
   sudo apt-get update
   sudo apt-get install sendmail
   ```

2. Configure `sendmail` to work with your SMTP server (this can vary based on your SMTP provider; you might need to refer to the provider's documentation).

#### Step 4: Ensure Everything Works

1. **Test your setup**:
   - Reboot the system or log out and back in to trigger the OTP generation.
   - Check your email for the OTP and enter it when prompted.

2. **Ensure the permissions are correct**:
   - Make sure that the `/tmp/otp_value.txt` file is not accessible to other users for security purposes:

   ```bash
   chmod 600 /tmp/otp_value.txt
   ```

#### Step 5: Make the Script Run on Reboot (Optional)

1. If you want the OTP generation to run automatically after a reboot, consider adding the following line to your crontab:

   ```bash
   crontab -e
   ```

   Then add:

   ```bash
   @reboot /path/to/your/otp_script.sh
   ```

### Important Notes

- This method relies on `sendmail`, which may require configuration to connect to your SMTP server securely.
- Security concerns: Storing OTP in a file can be insecure; it’s better to use an actual OTP generator like `Google Authenticator` or `Authy` for production environments.
- Ensure your email credentials are secured and not exposed in scripts.

By following these steps, you should be able to set up SSH login with OTP verification via email on your Ubuntu machine without installing additional software beyond what's necessary to send emails.
