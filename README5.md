To set up SSH login on Ubuntu with email OTP (One-Time Password) verification without any additional installations and ensuring it starts automatically after a reboot, you can leverage `ssh` in combination with a script that sends an OTP to your email. Below are the steps to achieve this:

### Prerequisites
- An SMTP server or an email account that allows sending emails (e.g., Gmail, Outlook).
- Access to edit `~/.ssh/authorized_keys` and `~/.bashrc` or `/etc/profile`.

### Steps

1. **Create an Email Sending Script**:
   You'll need a script to send the OTP via email. Here’s a basic example using `sendmail`, which is often pre-installed on Ubuntu.

   Create a script, for example, `/usr/local/bin/send_otp.sh`:

   ```bash
   #!/bin/bash

   EMAIL="your_email@example.com"
   OTP=$(openssl rand -base64 6) # Generate a random OTP
   echo "$OTP" > /tmp/otp.txt # Store OTP in a temporary file
   SUBJECT="Your OTP Code"
   BODY="Your OTP code is: $OTP"

   # Send email using the sendmail command
   echo -e "Subject:$SUBJECT\n\n$BODY" | /usr/sbin/sendmail -t "$EMAIL"
   ```

   Make the script executable:
   ```bash
   sudo chmod +x /usr/local/bin/send_otp.sh
   ```

2. **Modify SSH Login Script**:
   You need to configure the SSH session to run the OTP verification script. This can be done in your `~/.bashrc` or `/etc/profile` file to execute the OTP check on login.

   Edit `~/.bashrc` (for a specific user) or `/etc/profile` (for all users) and add:

   ```bash
   # Send OTP on SSH login
   /usr/local/bin/send_otp.sh

   # Read the OTP from the temporary file
   read -p "Enter the OTP sent to your email: " USER_OTP

   # Check if the entered OTP matches the generated one
   if [ "$USER_OTP" != "$(cat /tmp/otp.txt)" ]; then
       echo "Invalid OTP. Login denied."
       exit 1
   fi

   # Clean up the OTP file
   rm /tmp/otp.txt
   ```

3. **Configure SSH**:
   Ensure your SSH server is configured to allow this setup. Check `/etc/ssh/sshd_config` and confirm that the following line is present:

   ```plaintext
   PermitRootLogin no
   ```

   (This is a good security practice to avoid direct root login via SSH.)

   Restart the SSH service:
   ```bash
   sudo systemctl restart ssh
   ```

4. **Testing**:
   - Now, when you SSH into the server, the script should send an OTP to your email.
   - After that, it will prompt you to enter the OTP. If the OTP is correct, you will log in; otherwise, it will deny access.

5. **Ensure Auto-Execution on Reboot**:
   The above setup should automatically work after a reboot since the script runs every time a user logs in.

### Notes
- This method relies on having a working email configuration on your server. If you're using Gmail, you may need to set up app passwords or allow less secure apps.
- Make sure your server has proper security measures, as this implementation has some limitations and can be improved further with better error handling and security practices.
- Consider logging and monitoring for any failed login attempts for security purposes. 

This method allows for basic OTP verification via email without installing any additional software.
