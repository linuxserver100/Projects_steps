To add a new user to an Ubuntu server and set up SSH key login, follow these steps:

### Step 1: Add a New User

1. **Log into your server** via SSH as a user with sudo privileges.
   
   ```bash
   ssh username@server_ip
   ```

2. **Add a new user** (replace `newuser` with your desired username):

   ```bash
   sudo adduser newuser
   ```

   Follow the prompts to set the password and user details.

### Step 2: Create SSH Keys (if you don’t have them)

On your local machine, generate SSH keys if you haven't already:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

You can press Enter to accept the default file location and set a passphrase if desired.

### Step 3: Copy the Public Key to the Server

You can copy the public key to the server using the `ssh-copy-id` command:

```bash
ssh-copy-id newuser@server_ip
```

Alternatively, you can manually copy the key:

1. **Display the public key**:

   ```bash
   cat ~/.ssh/id_rsa.pub
   ```

2. **Log into the server** as the new user:

   ```bash
   ssh newuser@server_ip
   ```

3. **Create the `.ssh` directory** (if it doesn’t exist):

   ```bash
   mkdir -p ~/.ssh
   ```

4. **Add the public key to the `authorized_keys` file**:

   ```bash
   echo "your_public_key_here" >> ~/.ssh/authorized_keys
   ```

   Replace `your_public_key_here` with the actual contents of `id_rsa.pub`.

5. **Set the correct permissions**:

   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

### Step 4: Test SSH Login

Log out from the server and attempt to SSH into the new user account:

```bash
ssh newuser@server_ip
```

If everything is set up correctly, you should log in without needing a password.

### Step 5: (Optional) Disable Password Authentication

To enhance security, you might want to disable password authentication:

1. Open the SSH configuration file:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Find and set the following parameters:

   ```bash
   PasswordAuthentication no
   ChallengeResponseAuthentication no
   ```

3. **Restart the SSH service**:

   ```bash
   sudo systemctl restart ssh
   ```

Now your new user can log in using SSH keys only.
