Setting up SSH login with email OTP (One-Time Password) verification on Ubuntu without installing any additional software can be a complex process, especially if you want it to start automatically after a reboot. The typical way to achieve OTP functionality usually involves using additional packages like `oath-toolkit`, `libpam-google-authenticator`, or similar. However, since you requested a method without installations, we can leverage existing tools with a focus on scripting and automation.

Here’s a detailed guide on how to configure SSH to use email OTP verification with minimal installations:

### Step 1: Configure SSH Server

1. **Open the SSH Configuration File:**
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
   
2. **Edit Configuration:**
   Make sure the following lines are set (you may need to uncomment them):
   ```bash
   PermitRootLogin no
   PasswordAuthentication no
   ChallengeResponseAuthentication yes
   UsePAM yes
   ```

3. **Save and Exit:**
   Press `CTRL + X`, then `Y`, and `Enter` to save and exit the editor.

4. **Restart SSH Service:**
   ```bash
   sudo systemctl restart ssh
   ```

### Step 2: Set Up Email Sending

1. **Configure a Mail Transfer Agent (MTA):**
   For simplicity, we can use `sendmail` (which is commonly pre-installed).
   Ensure that your MTA is set up correctly to send emails. This step will vary based on your environment, but here’s a simple way to test:

   ```bash
   echo "Test Email" | sendmail your_email@example.com
   ```

   Check your email to confirm that it works.

### Step 3: Generate and Store OTP

1. **Create an OTP Generation Script:**
   Create a shell script to generate and send OTP to your email.

   ```bash
   sudo nano /usr/local/bin/otp_send.sh
   ```

   **Add the following script:**
   ```bash
   #!/bin/bash

   # Generate a random OTP
   OTP=$(shuf -i 100000-999999 -n 1)

   # Send OTP to the user's email
   echo "Your OTP is: $OTP" | sendmail your_email@example.com

   # Save OTP to a temporary file
   echo $OTP > /tmp/otp.txt

   # Optional: set a timer to remove the OTP after some time
   sleep 300 && rm /tmp/otp.txt &
   ```

2. **Make the Script Executable:**
   ```bash
   sudo chmod +x /usr/local/bin/otp_send.sh
   ```

### Step 4: Configure PAM (Pluggable Authentication Module)

1. **Edit the PAM Configuration for SSH:**
   ```bash
   sudo nano /etc/pam.d/sshd
   ```

   **Add the following lines at the end:**
   ```bash
   auth required pam_exec.so /usr/local/bin/otp_send.sh
   auth required pam_exec.so /usr/local/bin/otp_verify.sh
   ```

### Step 5: Create OTP Verification Script

1. **Create the OTP Verification Script:**
   ```bash
   sudo nano /usr/local/bin/otp_verify.sh
   ```

   **Add the following script:**
   ```bash
   #!/bin/bash

   read -p "Enter the OTP sent to your email: " user_otp

   # Read the OTP from the temporary file
   stored_otp=$(cat /tmp/otp.txt 2>/dev/null)

   # Check if OTP is correct
   if [[ "$user_otp" == "$stored_otp" ]]; then
       exit 0
   else
       exit 1
   fi
   ```

2. **Make the Script Executable:**
   ```bash
   sudo chmod +x /usr/local/bin/otp_verify.sh
   ```

### Step 6: Testing the Setup

1. **Reboot the System:**
   ```bash
   sudo reboot
   ```

2. **Attempt to SSH:**
   From a remote machine, try to SSH into your Ubuntu server:
   ```bash
   ssh username@your-server-ip
   ```

3. **Check Email for OTP:**
   After initiating the SSH connection, check your email for the OTP.

4. **Enter OTP:**
   When prompted for the OTP, enter the code received in your email.

### Step 7: Automation after Reboot

The scripts you created will run automatically upon SSH login due to the PAM configuration. Ensure that the MTA is correctly configured to send emails on startup.

### Important Notes:

- **Security Concerns:** This method does not use encryption for the OTP in transit. Consider using a more secure method for production environments, such as two-factor authentication with Google Authenticator.
- **Dependencies:** Ensure that your MTA (e.g., `sendmail`, `postfix`) is properly configured to send emails, and that necessary permissions are granted to the scripts.
- **OTP Lifetime:** The OTP generated is stored temporarily; adjust the sleep time in the `otp_send.sh` as needed.
- **Testing:** Be sure to test this configuration thoroughly in a secure environment before deploying it in production.

This configuration uses existing tools and should work on a standard Ubuntu installation without any additional software installation.
