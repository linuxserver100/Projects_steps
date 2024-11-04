
Setting up SSH login on an Ubuntu server using email-based OTP (One-Time Password) authentication without using PAM (Pluggable Authentication Module) configuration prompts requires a custom approach. Below is a step-by-step guide to achieve this:

### Prerequisites
- An Ubuntu server with SSH access.
- A working email account to send OTPs.
- `ssh` and `sendmail` (or similar email sending tool) installed on the server.
- Basic knowledge of scripting and SSH.

### Steps

1. **Install Required Packages**:
   Ensure you have `sendmail` installed for sending emails. You can install it using:
   ```bash
   sudo apt update
   sudo apt install sendmail
   ```

2. **Generate an OTP**:
   You can use a simple script to generate a 6-digit OTP. Here’s an example using `bash` and `openssl`:

   ```bash
   #!/bin/bash

   OTP=$(openssl rand -base64 6 | tr -d /= | cut -c1-6)
   echo $OTP
   ```

   Save this script as `generate_otp.sh` and make it executable:
   ```bash
   chmod +x generate_otp.sh
   ```

3. **Send OTP via Email**:
   You will need a script to send the OTP via email. Create a script named `send_otp.sh`:

   ```bash
   #!/bin/bash

   EMAIL=$1
   OTP=$2

   SUBJECT="Your OTP Code"
   BODY="Your OTP code is: $OTP"

   echo "$BODY" | sendmail "$EMAIL"
   ```

   Make this script executable as well:
   ```bash
   chmod +x send_otp.sh
   ```

4. **Integrate OTP with SSH Login**:
   You can create a custom login script that handles the OTP generation and validation process. Here's a simple example script (`otp_login.sh`):

   ```bash
   #!/bin/bash

   EMAIL="your-email@example.com"  # Change this to the user's email
   OTP=$(./generate_otp.sh)

   # Send OTP to email
   ./send_otp.sh "$EMAIL" "$OTP"

   echo "An OTP has been sent to $EMAIL. Please enter the OTP:"
   read USER_OTP

   if [ "$USER_OTP" == "$OTP" ]; then
       echo "OTP validated. You are now logged in."
       exit 0
   else
       echo "Invalid OTP. Access denied."
       exit 1
   fi
   ```

   Make it executable:
   ```bash
   chmod +x otp_login.sh
   ```

5. **Modify SSH Configuration**:
   To allow users to run the OTP login script during their SSH session, edit the `/etc/ssh/sshd_config` file and set the `ForceCommand` directive. Add the following line:

   ```bash
   ForceCommand /path/to/otp_login.sh
   ```

   Replace `/path/to/otp_login.sh` with the actual path where your script is stored.

6. **Restart SSH Service**:
   After making changes to the SSH configuration, restart the SSH service:

   ```bash
   sudo systemctl restart ssh
   ```

### Important Considerations

- **Security**: This approach bypasses many security mechanisms in place with PAM. It is advisable to use this method with caution.
- **Email Delivery**: Ensure that the email server you are using can send emails without issues. You may need to configure `sendmail` properly or use an SMTP server.
- **Testing**: Test the script thoroughly to ensure that it works as expected without creating security vulnerabilities.

### Conclusion
This setup allows SSH login with OTP authentication through email without relying on PAM configurations. However, remember that this is a custom solution, and for production systems, consider using established methods and libraries for OTP management, like Google Authenticator or Authy.
