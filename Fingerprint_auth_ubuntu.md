Setting up fingerprint login on an Ubuntu server can be achieved through a series of steps. The approach will vary slightly depending on whether you're setting it up for a PC (with a built-in or USB fingerprint sensor) or configuring remote authentication (for Android or other devices). Here's a step-by-step guide for both configurations:

### **Step 1: Install Fingerprint Utilities on Ubuntu Server**

Ubuntu has support for fingerprint authentication using PAM (Pluggable Authentication Modules). You will first need to install fingerprint software that interfaces with your fingerprint sensor.

1. **Update your system packages:**

   ```bash
   sudo apt update
   sudo apt upgrade
   ```

2. **Install the required packages:**

   For fingerprint authentication, you need the `fprintd` package.

   ```bash
   sudo apt install fprintd libpam-fprintd
   ```

3. **Install support for fingerprint sensors** (if your sensor is not supported by default):
   
   Some sensors might require specific drivers or software such as the `libfprint` library or external drivers from the sensor manufacturer.

   ```bash
   sudo apt install libfprint-2-2 libfprint-2-tod1
   ```

### **Step 2: Enroll Fingerprint on Ubuntu Server**

1. **Check if your fingerprint sensor is recognized:**

   Run the following command to list available fingerprint devices:

   ```bash
   fprintd-list
   ```

   If the fingerprint sensor is correctly installed and detected, it will list available devices.

2. **Enroll your fingerprint:**

   Use the `fprintd-enroll` command to enroll a new fingerprint. You will be asked to swipe or press your finger several times.

   ```bash
   fprintd-enroll
   ```

   Follow the on-screen instructions to successfully enroll your fingerprint.

### **Step 3: Configure PAM for Fingerprint Authentication**

1. **Edit the PAM configuration file:**

   Open the `/etc/pam.d/common-auth` file:

   ```bash
   sudo nano /etc/pam.d/common-auth
   ```

2. **Add the following line to enable fingerprint authentication:**

   Add this line at the top of the file:

   ```bash
   auth   [success=1 default=ignore]   pam_fprintd.so
   ```

   This line ensures that the system will check for fingerprint authentication before falling back to other methods like password.

3. **Save and exit:**

   Press `Ctrl+O` to save, then `Ctrl+X` to exit.

### **Step 4: Test Fingerprint Login on the Server**

1. Lock your screen or restart the server, and attempt to log in using your fingerprint.
   
2. You should be prompted to authenticate with your fingerprint first. If it fails, you will be asked to provide your password.

### **Optional: Enable Sudo Authentication with Fingerprint**

To use your fingerprint for `sudo` authentication, follow these steps:

1. **Edit the sudo PAM configuration file:**

   ```bash
   sudo nano /etc/pam.d/sudo
   ```

2. **Add the same line:**

   Add the following line to the top:

   ```bash
   auth   sufficient   pam_fprintd.so
   ```

3. **Save and exit.**

### **Step 5: Set Up Remote Fingerprint Authentication (For Android)**

To use fingerprint authentication from an Android device to log into your Ubuntu server remotely, you will need to install an app that supports SSH fingerprint login, and configure key-based SSH authentication.

#### 1. **Install Termux on Android**

- Termux is a terminal emulator for Android that allows you to use Linux commands.
- You can find it on the Play Store or F-Droid.

#### 2. **Set Up SSH Key Pair**

Generate an SSH key pair on your Android device (within Termux) to use for authentication:

```bash
ssh-keygen -t rsa -b 4096
```

This will generate a private key (`~/.ssh/id_rsa`) and a public key (`~/.ssh/id_rsa.pub`).

#### 3. **Copy the Public Key to the Ubuntu Server**

Transfer the public key to the server using `ssh-copy-id`:

```bash
ssh-copy-id user@server-ip
```

Ensure that password-based SSH login is disabled for better security:

```bash
sudo nano /etc/ssh/sshd_config
```

Look for the line `PasswordAuthentication` and set it to `no`.

#### 4. **Install OpenSSH on Termux**

Install OpenSSH on your Android device using:

```bash
pkg install openssh
```

You can now log into your Ubuntu server using SSH and use Android’s biometric authentication as part of the SSH login process by combining it with password-less (key-based) SSH login.

### **Step 6: Troubleshooting and Notes**

- If the fingerprint sensor isn't recognized, ensure you have the correct drivers installed. Some USB fingerprint scanners might require additional drivers or software.
- Some Android devices may allow more advanced biometric authentication solutions for remote logins via specific apps that support fingerprint-based login.
- Ensure your Ubuntu server version supports the biometric modules and libraries installed (test with Ubuntu 20.04 or newer).

### Summary
- **For PC**: Install `fprintd`, enroll fingerprint, and configure PAM for both login and sudo.
- **For Android**: Use Termux with SSH key authentication for remote access, securing it with Android’s biometric system for added security.

Let me know if you need any additional details!
