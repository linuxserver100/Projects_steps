To configure the same SSH private key for a new user to log in to an Ubuntu server as an existing user, follow these steps:

### 1. Copy the Existing User's SSH Configuration

1. **Log in to the server** as the existing user that has SSH access.

2. **Check the existing user's SSH directory**:

   ```bash
   cd ~/.ssh
   ```

3. **List the contents** to see the authorized keys:

   ```bash
   ls -l
   ```

### 2. Create the New User (if not already created)

If you haven't created the new user yet, do so:

```bash
sudo adduser newuser
```

### 3. Copy SSH Keys to the New User

1. **Switch to the new user**:

   ```bash
   sudo su - newuser
   ```

2. **Create the `.ssh` directory** if it doesn’t exist:

   ```bash
   mkdir -p ~/.ssh
   chmod 700 ~/.ssh
   ```

3. **Copy the `authorized_keys` file from the existing user**:

   Exit from the new user session back to the existing user, then run:

   ```bash
   sudo cp ~/.ssh/authorized_keys /home/newuser/.ssh/
   sudo chown newuser:newuser /home/newuser/.ssh/authorized_keys
   ```

4. **Set the correct permissions** for the new user’s `authorized_keys` file:

   ```bash
   sudo chmod 600 /home/newuser/.ssh/authorized_keys
   ```

### 4. Set Permissions for the New User's Home Directory

Ensure the home directory and `.ssh` folder permissions are correct:

```bash
sudo chmod 700 /home/newuser
sudo chmod 700 /home/newuser/.ssh
```

### 5. Test SSH Access

Now, test the SSH access with the new user:

```bash
ssh newuser@server_ip
```

You should be able to log in using the same private key as the existing user.
