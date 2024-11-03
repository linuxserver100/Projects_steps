
To configure SSH login on your Ubuntu server with email-based OTP (One-Time Password) authentication, you’ll need to follow a few steps. We’ll use `pam_exec` and a simple script to send an OTP to an email. Then, we’ll configure your SSH to require this OTP on login. Finally, I’ll guide you on adding a file to `.bashrc`.

Here’s the step-by-step guide:

### Step 1: Install Required Packages
First, ensure you have `ssmtp` or `mail` for sending emails. You can install `ssmtp` for this:

```bash
sudo apt update
sudo apt install ssmtp mailutils
```

Then, configure `ssmtp` to be able to send emails by editing its configuration file:

```bash
sudo nano /etc/ssmtp/ssmtp.conf
```

Add the following configuration (replace with your SMTP details):

```bash
mailhub=smtp.gmail.com:587
AuthUser=your-email@gmail.com
AuthPass=your-email-password
UseSTARTTLS=YES
FromLineOverride=YES
```

### Step 2: Create the OTP Script
Create a script that generates an OTP, emails it to the user, and verifies it on login.

1. **Create the script:**

   ```bash
   sudo nano /usr/local/bin/email_otp.sh
   ```

2. **Paste the following code:**

   ```bash
   #!/bin/bash

   # Generate a random OTP
   OTP=$(shuf -i 100000-999999 -n 1)

   # Email the OTP
   echo "Your OTP is: $OTP" | mail -s "SSH Login OTP" your-email@gmail.com

   # Prompt the user for the OTP
   read -p "Enter OTP sent to your email: " input_otp

   # Check if the OTP matches
   if [[ "$input_otp" == "$OTP" ]]; then
       exit 0  # Success
   else
       echo "Invalid OTP"
       exit 1  # Failure
   fi
   ```

   Replace `your-email@gmail.com` with the actual email where you want to receive the OTP.

3. **Make the script executable:**

   ```bash
   sudo chmod +x /usr/local/bin/email_otp.sh
   ```

### Step 3: Configure PAM for SSH with OTP
Edit the SSH PAM configuration to include the OTP script as a required step.

1. Open the PAM SSH configuration:

   ```bash
   sudo nano /etc/pam.d/sshd
   ```

2. Add this line at the beginning of the file:

   ```bash
   auth required pam_exec.so /usr/local/bin/email_otp.sh
   ```

### Step 4: Update SSHD Configuration
Now, make sure SSH is set to use password authentication (so PAM can handle OTP checks).

1. Edit the SSH daemon configuration file:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Ensure the following options are set:

   ```bash
   UsePAM yes
   ChallengeResponseAuthentication yes
   PasswordAuthentication yes
   ```

3. Save and close the file, then restart SSH:

   ```bash
   sudo systemctl restart sshd
   ```

### Step 5: Test the Setup
Try logging in via SSH. You should now be prompted for both your password and an OTP sent to your email.

### Step 6: Add File to `.bashrc`
If you want to source a file every time you open a terminal, you can add it to `.bashrc`.

1. Edit the `.bashrc` file:

   ```bash
   nano ~/.bashrc
   ```

2. At the end, add:

   ```bash
   source /path/to/your/file.sh
   ```

3. Save and close the file, and apply changes with:

   ```bash
   source ~/.bashrc
   ```

Replace `/path/to/your/file.sh` with the actual path of the file you want to add. This will run the file’s content every time you open a terminal session.
