
To set up SSH login with email OTP (One-Time Password) on an Ubuntu server **without additional installations**, you can leverage the built-in features of SSH and email. Here’s a step-by-step guide:

### Prerequisites

- Access to your Ubuntu server via SSH (with sudo privileges).
- An email account that can send emails via command line (such as Gmail or another email SMTP provider).
- Basic knowledge of SSH and configuring Ubuntu’s email.

### Steps

#### 1. Configure SSH Login with Public Key Authentication

Ensure the user’s public key is set up for SSH login. This adds a layer of security before implementing the OTP system.

1. **Generate a key pair (if not already done):**
   - Run on your local machine:
     ```bash
     ssh-keygen -t rsa -b 4096
     ```
   - Save the key in the default location (e.g., `~/.ssh/id_rsa`).

2. **Copy the public key to the server:**
   - Run this command to copy the key:
     ```bash
     ssh-copy-id username@your-server-ip
     ```
   - Replace `username` and `your-server-ip` with the appropriate values.

3. **Confirm SSH access:**
   - Attempt logging in to verify that you don’t need a password:
     ```bash
     ssh username@your-server-ip
     ```

#### 2. Set Up a Simple OTP System Using a Shell Script

Create a shell script on the server to generate and send an OTP via email each time you log in.

1. **Log in to the server and create a new script:**
   - Create the script in the user’s home directory (e.g., `~/send_otp.sh`):
     ```bash
     nano ~/send_otp.sh
     ```

2. **Add the following code to generate and email the OTP:**
   ```bash
   #!/bin/bash

   # Generate a random 6-digit OTP
   OTP=$(shuf -i 100000-999999 -n 1)
   
   # Store the OTP in a temporary file (for validation purposes)
   echo "$OTP" > /tmp/otp_code

   # Define the email details
   EMAIL="your-email@example.com"       # Replace with your email
   SUBJECT="Your OTP for SSH Login"
   BODY="Your OTP is: $OTP"

   # Send the email (using the `mail` command)
   echo "$BODY" | mail -s "$SUBJECT" "$EMAIL"
   ```

   - Replace `your-email@example.com` with the email address where you want to receive OTPs.

3. **Save and close the script file.**

4. **Make the script executable:**
   ```bash
   chmod +x ~/send_otp.sh
   ```

#### 3. Configure SSH to Run the OTP Script and Require OTP Validation

Edit the `.bashrc` or `.profile` file to prompt for the OTP each time the user logs in.

1. **Edit the user’s `.bashrc` file:**
   ```bash
   nano ~/.bashrc
   ```

2. **Add the following code to `.bashrc` to trigger the OTP script and prompt for OTP input:**
   ```bash
   # Run the OTP script to send an OTP via email
   ~/send_otp.sh

   # Prompt the user to enter the OTP
   read -p "Enter OTP: " entered_otp

   # Read the stored OTP from the temporary file
   stored_otp=$(cat /tmp/otp_code)

   # Check if the entered OTP matches the stored OTP
   if [ "$entered_otp" != "$stored_otp" ]; then
       echo "Invalid OTP. Access denied."
       exit 1
   fi

   # Remove the OTP file after successful login
   rm /tmp/otp_code
   ```

3. **Save and close `.bashrc`.**

4. **Test the SSH login:**
   - Disconnect and try to SSH into the server again:
     ```bash
     ssh username@your-server-ip
     ```
   - Upon login, you should receive an email with an OTP, and the terminal will prompt you to enter it.
   - Enter the OTP to complete the login.

#### Notes

- This method uses basic email commands and a script to manage OTPs, ensuring no additional packages are installed.
- If you use Gmail or a similar provider, you may need to configure your account to allow email sending from the server (such as by using an app password).
- Be cautious with email security as email OTPs are less secure than app-based OTPs.

This approach should give you a basic OTP system for SSH without extra installations.
