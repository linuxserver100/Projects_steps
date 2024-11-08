Implementing email link click authentication for SSH using a combination of `ForceCommand`, a custom script, and email generation involves several steps. This approach requires SSH to execute a specific command when a user connects, and that command includes authentication via a link sent to their email. Here’s a detailed guide to achieving this:

### Step 1: Setup Email Sending with `ssmtp`

1. **Install `ssmtp`:**
   Ensure that `ssmtp` is installed on your server. You can install it using:

   ```bash
   sudo apt-get install ssmtp
   ```

2. **Configure `ssmtp`:**
   Configure the `/etc/ssmtp/ssmtp.conf` file for your SMTP server. For example:

   ```plaintext
   root=postmaster
   mailhub=smtp.example.com:587
   AuthUser=your_email@example.com
   AuthPass=your_email_password
   UseSTARTTLS=YES
   ```

3. **Set Up Email Sending Script:**
   Create a script that sends an email with a unique link. Here’s an example script (`send_auth_email.sh`):

   ```bash
   #!/bin/bash

   EMAIL="$1"
   AUTH_CODE="$(openssl rand -hex 12)"
   LINK="http://yourdomain.com/authenticate?code=$AUTH_CODE"

   echo -e "Subject: SSH Authentication\n\nClick the link to authenticate your SSH session: $LINK" | ssmtp "$EMAIL"

   # Store AUTH_CODE and associate it with the user
   #echo "$AUTH_CODE" > "/tmp/auth_code_$USER"
   # echo "$AUTH_CODE" | sudo tee "/tmp/auth_code_$USER" > /dev/null
   #echo "$AUTH_CODE" | sudo tee "/tmp/auth_code_root" > /dev/null
   #echo "$AUTH_CODE" | sudo tee "/tmp/auth_code_root" > /dev/null && sudo chmod +x /tmp/auth_code_root
    echo "$AUTH_CODE" | sudo -u <username> tee "/tmp/auth_code_root" > /dev/null && sudo chmod +x /tmp/auth_code_root && chmod 644 /tmp/auth_code_root
   
   
   ```

   Make the script executable:

   ```bash
   chmod +x send_auth_email.sh
   ```

### Step 2: Create the Authentication Script

This script will handle the authentication when the user clicks the link.

```bash
#!/bin/bash

# auth_link.sh
AUTH_CODE="$1"
USER_AUTH_CODE_FILE="/tmp/auth_code_$USER"

# Debugging: Check if file exists
echo "Looking for auth code file: $USER_AUTH_CODE_FILE"

# Create a test auth code if the file doesn't exist
if [ ! -f "$USER_AUTH_CODE_FILE" ]; then
    echo "Creating auth code file with a test code."
    echo "12345" > "$USER_AUTH_CODE_FILE"
fi

# Check if the code matches the stored one
if [ -f "$USER_AUTH_CODE_FILE" ]; then
    STORED_CODE="$(cat $USER_AUTH_CODE_FILE)"
    if [ "$AUTH_CODE" == "$STORED_CODE" ]; then
        echo "Authentication successful."
        rm "$USER_AUTH_CODE_FILE"  # Cleanup
        exec $SHELL  # Open a shell session
    else
        echo "Authentication failed."
        exit 1
    fi
else
    echo "No authentication code found."
    exit 1
fi

```

Make this script executable:

```bash
chmod +x auth_link.sh
```

### Step 3: Update SSH Configuration

1. **Edit SSHD Configuration:**
   Open your SSH daemon configuration file (`/etc/ssh/sshd_config`) and add or modify the following line:

   ```plaintext
   ForceCommand /path/to/send_auth_email.sh $USER; exec /path/to/auth_link.sh
   ```

   Ensure to replace `/path/to/` with the actual path to your scripts.

2. **Set Up the Shell Environment:**
   In the same configuration file, set the default shell to your desired shell if needed:

   ```plaintext
   Subsystem sftp internal-sftp
   ```

### Step 4: Create the Link Handling Mechanism

You'll need a web server to handle the GET request from the link that the user clicks. Here’s a simple example using PHP:

```php
<?php
if (isset($_GET['code'])) {
    $code = $_GET['code'];
    // Call the authentication script with the code
    $output = shell_exec("/path/to/auth_link.sh $code 2>&1");
    echo "<pre>$output</pre>";
} else {
    echo "No code provided.";
}
?>
```

### Step 5: Restart SSH Service

After configuring everything, restart the SSH service:

```bash
sudo systemctl restart ssh
```

### Final Considerations

- **Security:** Ensure your scripts are secured and only accessible to authorized users.
- **Cleanup:** Implement a mechanism to clean up expired or used authentication codes.
- **Testing:** Thoroughly test the entire workflow to ensure users can authenticate smoothly.
- **Logging:** Consider adding logging for both successful and failed authentication attempts.

This setup combines SSH with email authentication, adding a layer of security for remote access.

🙂☹️🙂☹️🙂☹️😶🙂😯🙂😯🥱🤤😯🤤🥱☹️🤤🥱☹️🤤🥱😯😶🤤🤤😯🥱☹️🤤😶☹️😶🤤😯😶🤤😯🥱🤤😯😯🥱🤤😯😯🥱🤤😯😯🥱🤤😯😯🥱🤤😯🥱🙂😮🥱🙂😯🥱🙂😯🙂😯🥱🙂😢😮😐🙃😯😯🤭🙃😮🙂😯🥱🙃😮🙂😯🙂😶😯🙂😶😯😶🙂😯😶🙂😯😶🙂😯🥱🙂😶😯🙂😯😶🙂
To implement email-based link-click authentication for SSH access using `ForceCommand`, you can combine several tools and steps, including generating an authentication email, creating a custom script to verify authentication, and utilizing `ForceCommand` to control the SSH session. Here’s an outline of how you can set this up on a Linux server:

### Prerequisites

1. **ssmtp (or another email-sending tool)**: To send emails.
2. **A web server**: To host the authentication link.
3. **A custom shell script**: To verify the authentication link.
4. **SSH ForceCommand**: To control SSH command execution.

### Step-by-Step Setup

#### 1. Set Up `ssmtp` (or Similar) to Send Emails

Install `ssmtp` and configure it to work with an email provider (e.g., Gmail, Postfix, etc.). If `ssmtp` is not suitable for your needs, you could also use `sendmail` or `mail`.

Example configuration for `ssmtp`:
```bash
sudo apt-get install ssmtp
```

Configure `ssmtp` by editing `/etc/ssmtp/ssmtp.conf`:
```conf
root=username@gmail.com
mailhub=smtp.gmail.com:587
AuthUser=username@gmail.com
AuthPass=your_password
UseTLS=YES
UseSTARTTLS=YES
```

#### 2. Create the Custom Authentication Script

This script will generate a unique token for each SSH session and send it as part of a clickable link in an email.

- Create a directory for temporary tokens, like `/tmp/auth_tokens/`.
- Create a custom script, `auth_check.sh`, to generate the link and send an email with the token.

Here's an example of `auth_check.sh`:

```bash
#!/bin/bash

# Variables
EMAIL="user@example.com"  # Email to send the link
TOKEN_FILE="/tmp/auth_tokens"
TOKEN=$(uuidgen)  # Generate a unique token
URL="http://yourserver.com/verify?token=$TOKEN"  # Verification URL

# Store the token in a file with the user's name
#echo "$TOKEN" > "$TOKEN_FILE/${USER}.token"
 echo "$TOKEN" | tee "/tmp/auth_tokens/$USER.token" > /dev/null


# Send an email with the token link using ssmtp
echo -e "Subject: SSH Authentication\n\nClick the link to authenticate:\n$URL" | ssmtp "$EMAIL"

echo "An authentication link has been sent to your email. Please click the link to complete the SSH login."
echo "Waiting for authentication..."

# Wait for the file to be updated or deleted as a sign of authentication
while [ -f "$TOKEN_FILE/${USER}.token" ]; do
  sleep 1
done

# If the file is deleted, authentication succeeded
echo "Authentication successful. Welcome!"
exec $SHELL  # Proceed to the user's shell
```

#### 3. Configure the Web Server to Verify the Token

Set up a simple server-side script (like in PHP or Python) to delete the token file when the user clicks the link. For example, a PHP script `verify.php` could look like this:

```php
<?php
if (isset($_GET['token'])) {
    $token = $_GET['token'];
    $user_token_file = "/tmp/auth_tokens/{$_SERVER['REMOTE_USER']}.token";

    // Debug: Output token and file path
    echo "Token from URL: " . htmlspecialchars($token) . "<br>";
    echo "File path: " . htmlspecialchars($user_token_file) . "<br>";
    
    if (file_exists($user_token_file)) {
        $file_token = trim(file_get_contents($user_token_file));
        echo "Token from file: " . htmlspecialchars($file_token) . "<br>";
        
        if ($file_token === $token) {
            unlink($user_token_file);
            echo "Authentication successful!";
        } else {
            echo "Invalid or expired token.";
        }
    } else {
        echo "Token file does not exist.";
    }
} else {
    echo "No token provided.";
}
?>

```

Place `verify.php` on your web server and ensure it’s accessible at the URL defined in `auth_check.sh`.

#### 4. Set Up `ForceCommand` in SSH Configuration

Configure SSH to use `ForceCommand` to run the authentication script `auth_check.sh` whenever a user logs in.

1. Open `/etc/ssh/sshd_config`.
2. Add a `ForceCommand` entry for the users who need email link-click authentication. For example:

```conf
Match User your_ssh_user
    ForceCommand /path/to/auth_check.sh
```

3. Reload SSH to apply the changes:

```bash
sudo systemctl reload sshd
```

### How It Works

1. When the specified user logs in over SSH, `ForceCommand` runs `auth_check.sh`.
2. `auth_check.sh` generates a token, saves it, and sends an email with a verification link.
3. The user clicks the link in their email, triggering the server-side script (`verify.php`), which deletes the token file if the token is valid.
4. `auth_check.sh` detects the file deletion and grants access by launching the user’s default shell.

### Important Security Considerations

- **SSL**: Ensure your web server uses HTTPS to protect token transmission.
- **Permissions**: Restrict permissions for `/tmp/auth_tokens` and the scripts to prevent unauthorized access.
- **Token Expiration**: Consider implementing a timeout mechanism in `auth_check.sh` to delete stale tokens.

With this setup, users must verify each SSH session via email link click, adding an extra layer of security.


🥺🥺😠🤤😠🙂😤🥲🥲🥲😮‍💨🥹😮‍💨😙😮‍💨😗😮‍💨😗🙂😤🙃😤🤤😤🤫😤🫢😤😬😤🤤😤🙃😤😗😤😗😤🙃🙂😤🙂🙂😤🤤😤🤤😤🤤😤🤤😤🤤😤🤤🤤😤😤😋😋😤😋😤😤🤤😗😋🙄😋😗🙄😗😋🙄🙄😉😗😮‍💨😮‍💨😛😮‍💨😮‍💨😄😗😄😮‍💨😁😁😛😗😮‍💨😁😮‍💨😮‍💨😮‍💨😙😝😤😤😤😝😙😤😘😤😚😁😝😤😙😜😁😤😜😤😁😜😤😁😝😤😝😤😝😤😝😙😁😤😤😁😤😁😤😝😁😤😤😤😤😤😜😚😙😤😤😚😤😤😤😤😜😁😤😁

To implement email-based link-click authentication for SSH access and store tokens securely in a new directory with proper permissions, follow the steps below. We'll set up a dedicated directory to store tokens and ensure that it has proper permissions for security. The new folder will help segregate token storage and make it easier to manage.

### Steps to Implement Token Authentication with Token Storage Directory

#### 1. **Create a Token Storage Directory**

We will create a directory, e.g., `/var/auth_tokens`, to store the tokens. This directory will have full permissions for the web server and SSH server to read and write tokens.

```bash
sudo mkdir -p /var/auth_tokens
sudo chmod 700 /var/auth_tokens
sudo chown root:root /var/auth_tokens
```

- `700` permission ensures that only the owner (root) has read, write, and execute permissions on the folder.
- This directory will be used to store the authentication tokens generated for each session.

#### 2. **Update Token Generation Script**

We will update the token generation script to store the token in this new directory.

```bash
#!/bin/bash

# auth_email.sh
TOKEN=$(openssl rand -hex 16)
USER=$(echo $1)  # Get the username

# Define token storage path
TOKEN_FILE="/var/auth_tokens/$USER.token"

# Save the token and username in the file
echo "$TOKEN" > $TOKEN_FILE

# Create the authentication link
LINK="http://yourserver.com/authenticate.php?token=$TOKEN&user=$USER"

# Send the email
echo -e "Subject: SSH Authentication\n\nClick the link to authenticate your SSH session:" $LINK | ssmtp $USER
```

Ensure the script is executable:

```bash
chmod +x auth_email.sh
```

#### 3. **Create the Verification Script**

Now, we need to update the verification script that will validate the token and ensure proper session handling. This script will check if the token exists and is valid by reading the token from the file in `/var/auth_tokens`.

```php
<?php
// authenticate.php
session_start();

// Sanitize user inputs
$user = filter_input(INPUT_GET, 'user', FILTER_SANITIZE_STRING);
$token = filter_input(INPUT_GET, 'token', FILTER_SANITIZE_STRING);

// Check if user or token are empty
if (empty($user) || empty($token)) {
    echo "User or token is missing.";
    exit;
}

// Define token storage path
$tokenFile = '/var/auth_tokens/' . $user . '.token';

// Check if the token file exists
if (!file_exists($tokenFile) || !is_readable($tokenFile)) {
    echo "Error: Token file is missing or unreadable.";
    exit;
}

// Read the stored token from the file
$storedToken = file_get_contents($tokenFile);

// Validate the token
if ($storedToken === $token) {
    // Token is valid, log the user in
    $_SESSION['authenticated_user'] = $user;

    // Regenerate session ID to prevent session fixation
    session_regenerate_id(true);

    // Remove the token file after successful authentication
    unlink($tokenFile);

    echo "Authenticated. You can now access the SSH.";
    exit;
} else {
    echo "Invalid token.";
    exit;
}
?>
```

This PHP script will:

- Sanitize user input for security.
- Check if the token is valid by comparing it with the stored token.
- If valid, it will authenticate the user and delete the token to prevent reuse.
- If the token is invalid, it will print an error message.

#### 4. **Update the SSH ForceCommand Script**

You also need to ensure that SSH is properly using a script to check for authentication. Here's an updated version of the `verify_auth.sh` script that checks if the user is authenticated.

```bash
#!/bin/bash

# verify_auth.sh
if [[ -z "$SSH_CONNECTION" ]]; then
    echo "SSH connection not established."
    exit 1
fi

# Define token storage path
TOKEN_DIR="/var/auth_tokens"

# Check if the user has an associated token file
if [[ ! -f "$TOKEN_DIR/$USER.token" ]]; then
    echo "Not authenticated. Please check your email for the authentication link."
    exit 1
fi

# If the user has an authentication token, proceed to their shell
echo "Authenticated. You can now access the shell."
exec $SHELL  # Replace with the user's shell (e.g., /bin/bash)
```

This script checks if the token exists for the user in the `/var/auth_tokens` directory. If the token file is missing, it prevents SSH access, indicating that the user needs to authenticate by clicking the email link.

#### 5. **Modify SSH Configuration**

Ensure that `ForceCommand` is properly configured in the `/etc/ssh/sshd_config` file to enforce the verification process for the designated user(s):

```bash
Match User your_username
    ForceCommand /path/to/verify_auth.sh
```

#### 6. **Restart SSH Service**

After making changes to the SSH configuration, restart the SSH service to apply the changes:

```bash
sudo systemctl restart sshd
```

#### 7. **Security Considerations**

- **Token Expiry**: Consider adding an expiry date for tokens (e.g., tokens could expire after a few minutes or hours). You could implement this by storing the token’s creation time in the token file and checking it during authentication.
- **HTTPS**: Make sure that the link to the authentication page (`authenticate.php`) is served over HTTPS to prevent eavesdropping.
- **Token File Permissions**: The `/var/auth_tokens` directory should be set with strict permissions to prevent unauthorized access to token files. Only the web server and the SSH service should be able to read/write to this directory.

```bash
sudo chmod 700 /var/auth_tokens
sudo chown root:www-data /var/auth_tokens  # Change to appropriate web server user (e.g., www-data)
```

This ensures that only the necessary services (SSH and the web server) can read or write to the token storage directory.

### Conclusion

With these steps:

1. **Token Generation**: When a user attempts to SSH, a unique token is generated and stored in a secure directory (`/var/auth_tokens`).
2. **Token Email**: An email is sent with a link to authenticate the SSH session by clicking it.
3. **Authentication**: The user clicks the link, and the script validates the token.
4. **SSH Access**: If the token is valid, the user is authenticated and granted access to the shell.

By using this setup, you've created a more secure and customizable mechanism for SSH authentication that requires email-based link-click verification.

🥹😄🥹🥱🤭🥹😄🥱🥱😊😃🥱😄😄😊🥱🤗😄😊🤗🤗😍😄🤗😄☺️😶‍🌫️😄☺️🤗😄☺️😶‍🌫️☺️😄🤗☺️😄🤗😊😤😊😄🥱😊😄🥱😊😄🥱😊😄🤗😄😊😠😊😄😠😊😄🤗😊😄🤗😄😠😄😊🤗😊😄😠😄🤗😊😄🤗☺️😄😠😄😠☺️😄🤗☺️😙🤗☺️😙🤗😙😠😜😙🤗😙☺️😙🤗😙😠😜😄🤗😠😜😄🤗

To set up SSH login via an email link verification system and use `ForcedCommand` with `$SHELL` access, you can implement a solution by integrating several components. Here's an outline of how you can achieve this:

### 1. **SSH ForcedCommand Configuration**
   The `ForcedCommand` option in the SSH configuration forces the server to execute a specific command for every login, regardless of what the user attempts to run. In your case, we will configure it to allow the user to log in only if they pass a secondary email verification process.

   **Steps:**
   - Edit the `/etc/ssh/sshd_config` file on the server.
     ```bash
     sudo nano /etc/ssh/sshd_config
     ```
   - Add the following lines:
     ```
     Match User * 
         ForceCommand /usr/local/bin/verify_ssh_login.sh
         AllowTcpForwarding no
         X11Forwarding no
     ```
     This will ensure that every user triggers the `verify_ssh_login.sh` script on login.

### 2. **Email Verification Process**
   You'll need a system to send an email with a unique verification link that the user must click before proceeding with SSH login. This involves setting up a mechanism to generate a one-time link and then checking the link when the user logs in.

   **Steps:**
   - **Install SSMTP (or an SMTP client):**
     ```bash
     sudo apt-get install ssmtp
     ```
   - **Configure your email client:**
     Edit the `/etc/ssmtp/ssmtp.conf` to configure the outgoing mail server (you may use Gmail's SMTP server or any other service).
     Example configuration for Gmail:
     ```ini
     root=postmaster@gmail.com
     mailhub=smtp.gmail.com:587
     AuthUser=your_email@gmail.com
     AuthPass=your_password
     FromLineOverride=YES
     UseSTARTTLS=YES
     ```
   - **Create the verification script:**
     You'll need a custom script to generate a one-time login token and send it to the user via email.

     Here's a sample script (`/usr/local/bin/verify_ssh_login.sh`):

     ```bash
     #!/bin/bash
     # verify_ssh_login.sh
     
     # Assume the username is the email
     USER_EMAIL="$USER@example.com"
     LOGIN_TOKEN=$(uuidgen)
     VERIFICATION_LINK="https://your-domain.com/verify?token=$LOGIN_TOKEN"

     # Store the token in a temporary file or database
     echo "$LOGIN_TOKEN $USER_EMAIL" > /tmp/ssh_login_tokens.txt

     # Send the email
     echo -e "Subject: SSH Login Verification\n\nPlease click the link to verify your login: $VERIFICATION_LINK" | ssmtp $USER_EMAIL
     
     echo "A verification link has been sent to your email. Please click it to complete the login."
     exit 1  # Prevent login until the verification link is clicked
     ```

   - **Set execute permissions for the script:**
     ```bash
     sudo chmod +x /usr/local/bin/verify_ssh_login.sh
     ```

### 3. **Web Interface for Verification Link**
   Create a web page that the user can visit by clicking the link in their email. This web page will verify the token and then allow the user to continue with their SSH session.

   **Steps:**
   - Set up a simple web server (for example, Apache, Nginx, or any server of your choice) to handle the verification page.
   - The script should check the token against the list of generated tokens (`/tmp/ssh_login_tokens.txt` or a database).
   - After the user clicks the verification link, the server should delete the token from the list, allowing the SSH session to continue.

   **Example PHP verification page (verification.php):**
   ```php
   <?php
if (isset($_GET['token'])) {
    $token = $_GET['token'];
    $file = '/tmp/ssh_login_tokens.txt';

    // Check if file exists
    if (file_exists($file)) {
        $tokens = file($file, FILE_IGNORE_NEW_LINES);
        $isVerified = false;

        foreach ($tokens as $index => $line) {
            list($stored_token, $user_email) = explode(' ', $line);

            // Compare the token
            if ($stored_token == $token) {
                // Token is valid, proceed with the process (e.g., mark as verified)
                unset($tokens[$index]); // Remove the verified token

                // Update the file
                file_put_contents($file, implode("\n", $tokens));

                echo "Verification successful. You may now return to your SSH session.";
                $isVerified = true;
                break;
            }
        }

        if (!$isVerified) {
            echo "Invalid token.";
        }
    } else {
        echo "Token file not found.";
    }
} else {
    echo "No token provided.";
}
?>

   ```

### 4. **Finalizing SSH Login Process**
   - After the user clicks the verification link, the `verify_ssh_login.sh` script will allow the SSH session to proceed. You may use a flag or update the status in a file to indicate the user has successfully verified their login.

   **Example:**
   ```bash
   #!/bin/bash
   if [ ! -f /tmp/ssh_login_verified_$USER ]; then
       echo "Please verify your login by clicking the link sent to your email."
       exit 1
   fi

   # Allow login
   exec $SHELL
   ```

### Summary
This solution requires:
1. Configuring SSH `ForcedCommand` to run a verification script.
2. Sending a verification email with a unique token using `ssmtp`.
3. Providing a verification link for the user to click.
4. Setting up a web server to verify the token and update the login status.

This provides an additional layer of security, ensuring that users cannot log in without verifying their email first.

😑😮🤤😮🤭😏😵🥺😮‍💨😵😮‍💨🥺😵🥺😵😮‍💨🥺😵😮‍💨🥺😵😮‍💨🥺😵😵😮‍💨🥺😵😵😮‍💨🥺😵🥺😵😮‍💨🥺😵🥺😮‍💨😵🥺😮‍💨😵🥺🤭😵🤭🥺😵😮‍💨🥺😵🥺😵😮‍💨🥺😵🥺😮‍💨😵🤭🥺😵🥺🤭😵🤭🥺🤭🥺😵😮‍💨🥺😵😵😮‍💨🥺😵🥺😮‍💨😵🥺😵😮‍💨🥺😵🥺😮‍💨😵😮‍💨🥺😵😮‍💨😵😮‍💨🥺😵🤭🥺😵😮‍💨🥺😵😮‍💨🥺🤧😢🤔🤧
To set up SSH login via email link verification, with the forced command and email features like `ssmtp` or `php`, you'd typically follow these general steps:

### Overview of the Process:
1. **User Initiates SSH Login**: The user attempts to log in via SSH.
2. **Email Verification**: An email with a login link is sent to the user's registered email address.
3. **Forced Command Setup**: Once the user clicks the link, SSH access is granted with a restricted environment using the `forcedcommand` directive.
4. **$SHELL and Access Codes**: The server returns the user's shell or a custom shell based on the login script execution.

### Prerequisites:
- **Email System**: You need a mail system (like `ssmtp` or `sendmail`) installed and configured to send emails.
- **PHP**: You’ll need PHP installed to handle dynamic link generation and email sending.
- **SSH Setup**: Ensure SSH is configured to support forced commands and user verification.

### Steps:

#### 1. **Install Required Packages**:
Make sure the required tools are installed.

```bash
sudo apt update
sudo apt install ssmtp sendmail php openssh-server
```

#### 2. **Configure Mail System (ssmtp or sendmail)**:
- For `ssmtp`, you will need to configure it in `/etc/ssmtp/ssmtp.conf` with your SMTP server details.

Example `ssmtp.conf`:
```conf
root=postmaster
mailhub=smtp.your-email-provider.com:587
AuthUser=your-email@example.com
AuthPass=your-email-password
FromLineOverride=YES
UseSTARTTLS=YES
```

- If using `sendmail`, configure it according to your system and email provider.

#### 3. **PHP Script for Email Generation**:
Create a PHP script that generates a unique login link and sends it to the user’s email address.

```php
<?php
$email = $_POST['email'];
$login_token = bin2hex(random_bytes(16));  // Create a unique token

// Store the token with the email for later verification
file_put_contents('/tmp/verification_tokens.txt', "$email:$login_token\n", FILE_APPEND);

// Generate login link
$link = "https://yourserver.com/verify.php?token=$login_token&email=$email";

// Send the email
$subject = "SSH Login Verification";
$message = "Click the following link to log in: $link";
mail($email, $subject, $message, "From: your-email@example.com");

echo "A verification email has been sent.";
?>
```

#### 4. **Verification Script (verify.php)**:
This script handles the verification when the user clicks the link.

```php
<?php
$token = $_GET['token'];
$email = $_GET['email'];

// Check if token exists in the stored list
$tokens = file('/tmp/verification_tokens.txt', FILE_IGNORE_NEW_LINES);
foreach ($tokens as $line) {
    list($stored_email, $stored_token) = explode(':', $line);
    if ($stored_email === $email && $stored_token === $token) {
        // Token is valid, proceed with SSH login setup
        header('Location: ssh://yourserver.com');
        exit;
    }
}

echo "Invalid token.";
?>
```

#### 5. **Configure SSH with ForcedCommand**:
In `/etc/ssh/sshd_config`, you can configure SSH to execute a command upon successful login.

```bash
Match User yourusername
    ForceCommand /usr/local/bin/verify_user_login.sh
    AllowTcpForwarding no
    X11Forwarding no
```

This will execute a custom script (`/usr/local/bin/verify_user_login.sh`) when the user logs in.

#### 6. **Verification Script (`verify_user_login.sh`)**:
The script verifies the login, and can enforce restrictions, including setting the user’s `$SHELL` based on their verification.

```bash
#!/bin/bash

# You can add custom logic to verify the user’s login
# Example: Check for a file or a token passed

echo "Welcome to the server, $USER"

# Optionally set the user's shell based on a condition
export SHELL="/bin/bash"
```

Make sure this script is executable:

```bash
sudo chmod +x /usr/local/bin/verify_user_login.sh
```

#### 7. **Restart SSH Server**:
After configuring the SSH server, restart it to apply changes:

```bash
sudo systemctl restart sshd
```

### How It Works:
1. **User initiates login**: The user attempts to SSH into the server.
2. **Verification via email**: The server sends an email with a link.
3. **Link clicked**: User clicks on the email link to verify the login.
4. **SSH Forced Command**: Once the user is verified, the `ForceCommand` in SSH configuration allows a restricted command to run, allowing login with the desired shell or restricting access based on `$SHELL`.

This setup uses a combination of email verification, PHP scripts, SSH forced commands, and custom scripts to manage user access.

😋🤭😮‍💨😦😮‍💨😋😦😋🤭😦😦🤭🤤😦😦🤭🙂😦😦🤭🤤😦🤭🤤😦🤭🤤😦🤭🤤😦😦😋🤭😦😮😮‍💨🤤😵😮😮‍💨🤤😫😮‍💨🤤😦🤤😫😮‍💨😵😮‍💨🤤😫😋😵🤤😤😫😋😵😋😮‍💨😫😤😋😵😤🤤😫😵😋😤😵😤🤤😫🤤😫🤤😮‍💨😫🤤😮‍💨😫😮‍💨😮‍💨🤤😫😤😋😦😤😋😵😤😬😵😑😤😤😬😵😮😤🤤😵😋😵🤤😵🤤😤🤤😵😤😋😵😵😵😤😋😵😤😋😤😵😋😵😋😵😵😤🤤😵😋😤😵😤😋😵😤🤤😵😋😵😤🤤🤤😵😤🤤😵😋😤😵😤😵😋

To set up SSH login with an email verification link using `ssmtp`, you can follow the steps below with adjustments for using `ssmtp` instead of `msmtp`, and also a dedicated folder with full access for storing tokens securely.

### 1. Install Required Packages
First, install the necessary tools for sending emails and running your PHP server:

```bash
sudo apt-get update
sudo apt-get install ssmtp php libapache2-mod-php apache2
```

### 2. Configure `ssmtp` for Email Sending
Configure `ssmtp` by editing the `/etc/ssmtp/ssmtp.conf` file for your email provider (e.g., Gmail). The configuration may look like this:

```ini
root=your-email@gmail.com
mailhub=smtp.gmail.com:587
AuthUser=your-email@gmail.com
AuthPass=your-email-password
UseSTARTTLS=YES
FromLineOverride=YES
```

Make sure to replace `your-email@gmail.com` and `your-email-password` with your actual credentials. You may need to enable "Less Secure Apps" on your Google account or use an App Password if you have 2FA enabled.

### 3. Create a Directory for Storing Tokens
Create a directory where you will store the email verification tokens securely. This directory should have appropriate permissions so that it is accessible only to authorized users:

```bash
sudo mkdir /var/www/tokens
sudo chmod 700 /var/www/tokens
sudo chown www-data:www-data /var/www/tokens
```

This ensures that only the web server can write to this directory.

### 4. Create the Token Generation Script (send_verification_email.sh)
Write a shell script to handle the token generation and email sending process. This script will generate a random token, store it in a file, and send the verification link to the user.

Create the script `/var/www/html/send_verification_email.sh`:

```bash
#!/bin/bash

# Get the email address from POST request
email="$1"
token=$(openssl rand -hex 16)  # Generate a random 16-byte token

# Store the token with the email in a file
echo "$email:$token" >> /var/www/tokens/tokens.txt

# Construct the verification URL
verification_link="https://yourdomain.com/verify.php?token=$token"

# Send the verification email using ssmtp
echo -e "Subject: SSH Login Verification\n\nClick the following link to verify your SSH login: $verification_link" | ssmtp $email

echo "Verification email sent to $email"
```

Make sure the script is executable:

```bash
chmod +x /var/www/html/send_verification_email.sh
```

### 5. Create the PHP Verification Script (verify.php)
When the user clicks on the verification link, the server needs to check whether the token is valid. Create a PHP file `verify.php` to handle this process.

Create `/var/www/html/verify.php`:

```php
<?php
$token = $_GET['token'];  // Token from the email link
$tokenFile = '/var/www/tokens/tokens.txt';

if (!file_exists($tokenFile)) {
    echo "Invalid or expired token.";
    exit;
}

// Read the token file and check if the token is valid
$tokens = file($tokenFile, FILE_IGNORE_NEW_LINES);

foreach ($tokens as $line) {
    list($email, $saved_token) = explode(':', trim($line));

    if ($saved_token === $token) {
        // Token is valid, grant SSH access
        echo "Token verified. You can now SSH into the server.";
        
        // Optionally, you can log the successful verification or move the token to a 'used' list
        file_put_contents('/var/www/tokens/verified_users.txt', "$email\n", FILE_APPEND);
        
        // Remove the token from the tokens file to prevent reuse
        file_put_contents($tokenFile, implode("\n", array_filter($tokens, fn($line) => !str_contains($line, $token))) . "\n");

        exit;
    }
}

echo "Invalid or expired token.";
?>
```

### 6. Set Up the SSH Forced Command for Verification
To ensure that users can only SSH into the server after verifying their email, you can enforce a forced command in your SSH configuration. This command will check if the user has been verified.

Edit `/etc/ssh/sshd_config`:

```bash
# Enforce forced command
Match User *
    ForceCommand /path/to/verify_user.sh
```

Where `/path/to/verify_user.sh` is a script that checks if the user is in the verified users list and allows or denies SSH access.

Create `/path/to/verify_user.sh`:

```bash
#!/bin/bash

# Check if the user is in the verified_users.txt file
if grep -q "$USER" /var/www/tokens/verified_users.txt; then
    # User is verified, allow SSH session
    exec /bin/bash
else
    # User is not verified, deny access
    echo "Your email verification is pending or expired. Please check your email."
    exit 1
fi
```

Make sure the script is executable:

```bash
chmod +x /path/to/verify_user.sh
```

### 7. Configure Apache to Serve the PHP Scripts
Ensure your Apache server is set up to serve PHP files. If Apache is not installed or configured, you can install it with:

```bash
sudo apt-get install apache2
sudo systemctl start apache2
```

Make sure `mod_php` is enabled:

```bash
sudo a2enmod php
sudo systemctl restart apache2
```

### 8. Testing the System
1. The user accesses your web page, enters their email address, and the `send_verification_email.sh` script generates a token and sends the email.
2. The user clicks the verification link in the email, which triggers the `verify.php` script to validate the token.
3. If the token is valid, the user can now SSH into the server, and the `verify_user.sh` script will check if they are on the list of verified users before allowing shell access.

### Security Considerations:
- **Token Expiry**: Implement token expiration to prevent abuse (e.g., delete tokens after 15 minutes or one use).
- **Secure Token Storage**: Store tokens in a secure location, preferably hashed, and ensure proper file permissions are set.
- **SSH Configuration**: Ensure your SSH configuration does not allow bypassing the forced command or opening insecure access.

### Conclusion:
This method allows you to integrate email-based SSH login verification using `ssmtp` and store tokens in a secure directory. By checking tokens via a PHP verification script and enforcing SSH access through a forced command, you can ensure that only users who have clicked the verification link are granted access.

😗😊😙😙😊😙😊😊😄😄☺️😙☺️😙☺️😚☺️😚☺️😚☺️😚😚☺️😚☺️😚☺️😚☺️😚☺️😚☺️😚😚☺️😚☺️😚☺️😚☺️😚☺️😚☺️😚😚☺️😚☺️😚😌😚😌😚😌😚😚☺️😚☺️😚☺️😚☺️😚☺️😚☺️😚😚☺️😚☺️😚☺️😚☺️😚☺️😚😌😚😌😘☺️😘☺️😆☺️😁☺️😚☺️😚☺️😚☺️😚😚😌😚☺️😚😌
To fully implement the SSH login with email verification using a unique token, along with ForcedCommand for shell access, let's improve upon the existing setup. I'll include steps to create folders for token storage and secure access control for that folder, along with relevant scripts.

### Steps:

1. **Create a Directory for Token Storage**
   - First, we need to create a directory for storing the token file securely.

   ```bash
   sudo mkdir -p /var/www/tokens
   sudo chown root:root /var/www/tokens
   sudo chmod 700 /var/www/tokens
   ```

   - This ensures that only the root user can access this directory. You can change the ownership and permissions as needed depending on your setup.

2. **Generate the Token and Store It**
   - Create a script that generates a unique token and stores it in the `/var/www/tokens/` directory.

   Here’s a script that does it:

   ```bash
   #!/bin/bash
   # generate-token.sh

   # Generate a unique token using openssl or any preferred method
   TOKEN=$(openssl rand -base64 32)
   
   # Store the token in a file named after the username (for example, user1_token)
   echo $TOKEN > "/var/www/tokens/$USER"_token

   # Send the email with the verification link
   EMAIL_BODY="Click the link below to verify your SSH login:\n\nhttps://yourserver.com/verify-token?token=$TOKEN"
   echo -e "$EMAIL_BODY" | ssmtp $USER_EMAIL
   ```

   This script generates a unique token, saves it to a file named after the user, and sends the verification link via email.

3. **Email Verification via a Web Script**

   The user will receive an email with a link. When clicked, the server must validate the token.

   - Assuming you have PHP installed, create a PHP script for token verification.

     1. Install PHP and Nginx (if not already installed):

        ```bash
        sudo apt install nginx php-fpm
        ```

     2. Configure Nginx to route the token verification request to PHP:

        Edit `/etc/nginx/sites-available/default`:

        ```nginx
        server {
            listen 80;

            location /verify-token {
                fastcgi_pass 127.0.0.1:9000;
                fastcgi_param SCRIPT_FILENAME /var/www/html/verify-token.php;
                include fastcgi_params;
            }
        }
        ```

     3. Create the PHP script `verify-token.php` in `/var/www/html/`:

        ```php
        <?php
        if (isset($_GET['token'])) {
            $token = $_GET['token'];

            // Check if token file exists (based on username or other unique identifier)
            $file_path = '/var/www/tokens/' . $_SERVER['REMOTE_ADDR'] . '_token'; // Assuming the user's IP address is unique per session
            if (file_exists($file_path)) {
                $stored_token = file_get_contents($file_path);
                
                if ($stored_token === $token) {
                    // Token is valid - allow SSH access
                    echo "Token verified! You can now log in via SSH.";
                    
                    // Update file or database to mark the user as verified
                    file_put_contents('/var/www/tokens/verified_users', $_SERVER['REMOTE_ADDR']);
                } else {
                    echo "Invalid token.";
                }
            } else {
                echo "No token found.";
            }
        }
        ?>
        ```

4. **Configure ForcedCommand in SSH for Token Validation**
   We now need to set up SSH so that it checks whether the user has verified the token before allowing shell access.

   - Open `/etc/ssh/sshd_config` and modify it as follows:

     ```bash
     Match User your_username
         ForceCommand /path/to/validate-ssh-login.sh
         PasswordAuthentication no
     ```

   - Create a custom script (`validate-ssh-login.sh`) to check if the user has been verified:

     ```bash
     #!/bin/bash
     # validate-ssh-login.sh

     # Check if user has been verified
     if [ ! -f /var/www/tokens/verified_users ]; then
         echo "Email verification required."
         exit 1
     fi

     # If the user is verified, allow access to shell
     exec $SHELL
     ```

   - The script checks for a file (`verified_users`) which will be updated once the user clicks the verification link. If the user isn't verified, they will not be allowed to proceed.

5. **Set Permissions for Token Files and Verify Access**

   - Ensure that only the appropriate users can write to the token directory and the verification files:

     ```bash
     sudo chown -R root:root /var/www/tokens
     sudo chmod 700 /var/www/tokens
     ```

6. **Restart Services**
   After making these changes, restart the necessary services:

   ```bash
   sudo systemctl restart ssh
   sudo systemctl restart nginx
   ```

### Testing the Flow:

1. **Test Token Generation and Email**:
   - Run the `generate-token.sh` script to generate a token and send an email to the user.
   
   - The user should receive an email with the verification link. When they click on the link, the server will verify the token.

2. **SSH Login**:
   - Once the user clicks the verification link, the token is stored, and they can proceed with the SSH login.
   
   - Upon trying to log in via SSH, the `ForceCommand` will trigger and run the `validate-ssh-login.sh` script.
   
   - If the user is verified (i.e., the token has been validated), the login will proceed to the shell; otherwise, they will receive a message that email verification is required.

### Additional Considerations:
- **Security**: Make sure that the directory where tokens are stored (`/var/www/tokens/`) is properly secured and not accessible to unauthorized users.
- **Token Expiry**: You might want to add token expiration functionality for better security. For example, deleting tokens after they’ve been used or expired can enhance safety.
- **Logging**: You should log user login attempts, token verifications, and any potential issues for troubleshooting and auditing purposes.

This solution provides a secure, email-based SSH login with token validation and forced command execution to ensure the user is authenticated before gaining access to the system.

🙂🤭😮🙂🥲🥱😮🥱🙂😮🙂🤭🤭🙂😮😢🥱🙂😢🙂🥱😮🥱🙂😮🥱🙃😢🥱🙃😮😢🙂🥱😢🙂🥱😢🥱🙂😮😮🤭🙂😮🙂😮🤭🙃😮🤭🙂🙂🥱😮🥱🙂😮🙂😮🙂😮🙂🥱😮🤭🙂😮🙃🙂‍↔️😮😮😮🙂😮🥱🙂😮🥱🙂😮🙂😮🙂🥱😯🙂🥱😮🙂😯🥱🙂😮🥱🙂😮😯🥱🤤😯🤤😯🙂🥱😯🥱🙂😯🙂🥱😯🙂🥱😯😮🙂😯🥱🙂😯😶🙂😮😶🙃😙😮🙂😮😶🙂😮🙂😮😶🙂😯🥱🙂😯🙂😯🥱🙂😯🙂😮🙂🥱😮😶🙂😮😶🙂😯

To implement a full setup that uses SSH login with email verification, including file access permissions for PHP scripts to read and write user tokens, you’ll need to follow the steps below. These steps include the required codes, configuration, and security measures.

### Full Setup for SSH Login with Email Verification

This guide assumes you're using a Linux server with Apache or Nginx, PHP, `ssmtp` for email sending, and basic knowledge of SSH configuration.

#### 1. **Set Up SSH Server**
Ensure the SSH server is running and configured to use a custom script for user verification.

- **Install SSH Server (if not already installed)**:
  ```bash
  sudo apt-get install openssh-server
  ```

- **Configure SSH**:
  Open the SSH configuration file and add the `ForceCommand` directive to call the verification script.
  ```bash
  sudo nano /etc/ssh/sshd_config
  ```

  Add or modify these lines:
  ```bash
  PasswordAuthentication yes
  PermitRootLogin no
  ForceCommand /usr/local/bin/ssh_verification_script.sh
  ```

- **Restart SSH to apply the configuration**:
  ```bash
  sudo systemctl restart sshd
  ```

#### 2. **Create SSH Verification Script**

This script checks whether a user has already verified their login by email. If not, it sends a verification email with a unique token.

- **Create the script**:
  ```bash
  sudo nano /usr/local/bin/ssh_verification_script.sh
  ```

- **Example Script**:
  ```bash
  #!/bin/bash
  
  # Check if the user is already verified by looking for a verification flag
  if [ -f "/tmp/${USER}_verified" ]; then
    exec $SHELL  # If verified, launch the shell
  else
    # Generate a unique verification token and save it to a file
    verification_token=$(openssl rand -hex 16)
    echo $verification_token > /tmp/${USER}_token

    # Send the email with verification link
    verification_link="http://yourserver.com/verify.php?user=${USER}&token=${verification_token}"
    echo "Please verify your login by clicking this link: ${verification_link}" | ssmtp ${USER}@example.com

    # Display message prompting the user to verify
    echo "Verification email sent. Please check your inbox to complete login."
    exit 1
  fi
  ```

  - The script first checks if the user has a `$_verified` file in `/tmp/`.
  - If not, it generates a unique token, saves it to a file `/tmp/${USER}_token`, and sends a verification email via `ssmtp`.
  - If verified, the user is allowed to access the shell.

#### 3. **Set Up a Web Server to Handle the Verification Link**

You need to set up a web server to handle the verification request when the user clicks the verification link in the email.

- **Install Apache Web Server**:
  ```bash
  sudo apt-get install apache2
  ```

- **Create the PHP Script for Verification**:
  ```bash
  sudo nano /var/www/html/verify.php
  ```

- **PHP Verification Script**:
  ```php
  <?php
  // Ensure user and token are passed in the URL
  if (isset($_GET['user']) && isset($_GET['token'])) {
      $user = $_GET['user'];
      $token = $_GET['token'];
      
      // Set file paths for user token and verification status
      $token_file = "/tmp/{$user}_token";
      $verified_file = "/tmp/{$user}_verified";

      // Check if the token file exists and compare tokens
      if (file_exists($token_file)) {
          $stored_token = file_get_contents($token_file);
          
          if ($stored_token === $token) {
              // Mark the user as verified by creating the verification file
              touch($verified_file);
              echo "You have been successfully verified! You can now log in.";
              
              // Clean up token file after verification
              unlink($token_file);
          } else {
              echo "Invalid or expired verification token.";
          }
      } else {
          echo "Verification token not found.";
      }
  } else {
      echo "Invalid request.";
  }
  ?>
  ```

  - This script checks the URL for the `user` and `token` parameters.
  - It then compares the token from the URL to the one saved on the server.
  - If the token matches, it creates a `verified` file (`/tmp/${USER}_verified`), allowing the user to log in via SSH.
  - After verification, the token file is deleted for security.

#### 4. **Ensure PHP Can Read and Write to `/tmp/` Directory**

To ensure PHP can read and write to the `/tmp/` directory (where token files are stored), you need to set the proper permissions.

- **Set permissions** for the `/tmp/` directory (if needed):
  ```bash
  sudo chmod 1777 /tmp
  ```

- **Set directory permissions for PHP scripts**:
  You might need to grant Apache or your web server user (e.g., `www-data`) read/write access to the temporary files:
  ```bash
  sudo chown www-data:www-data /tmp
  ```

#### 5. **Set Up Email Sending (ssmtp or msmtp)**

For email functionality, you’ll need an SMTP service.

- **Install `ssmtp`** (or `msmtp` as an alternative):
  ```bash
  sudo apt-get install ssmtp
  ```

- **Configure `ssmtp`** by editing the configuration file:
  ```bash
  sudo nano /etc/ssmtp/ssmtp.conf
  ```

  Example configuration:
  ```ini
  root=your-email@example.com
  mailhub=smtp.example.com:587
  AuthUser=your-email@example.com
  AuthPass=your-email-password
  UseTLS=YES
  ```

- **Test sending an email**:
  ```bash
  echo "Test email body" | ssmtp recipient@example.com
  ```

#### 6. **Security Considerations**

- **Use HTTPS for the verification URL**: Ensure your Apache server is configured with SSL (e.g., using Let's Encrypt) to protect tokens in transit.
- **Permissions for `/tmp/`**: Ensure that the `/tmp/` directory is protected against unauthorized access, but that PHP and SSH have proper access for reading/writing tokens and verification files.
  ```bash
  sudo chmod 700 /tmp/
  sudo chown root:root /tmp/
  ```

- **Clean Up Temporary Files**: Consider adding a cron job to periodically clean up expired token files and verification files.

#### 7. **Test the Setup**

- **Test SSH login**: When a user attempts to SSH into the system, the script will check if they’ve verified their email.
- **Email Verification**: The user should receive a verification email. Clicking the link will verify the user and allow access on subsequent logins.

### Conclusion

This guide sets up an SSH-based login system where the user must verify their identity via an email link before accessing the system. The verification process uses a PHP web server, an SSH login script, and an SMTP service like `ssmtp` to send the verification email.

Make sure to test all components and secure the system with appropriate file permissions and SSL encryption for email and web interactions.

😙😙☺️😙☺️😄☺️😙😚☺️😁😌😄😌😄😌😚😌😚😌😌😁😁☺️😄😍😍😄😍😚😚😍😍😊😬😬🥲😬🥲😬😗🥲😬🥲😬🥲😬🥲😤🥲😤🥲😤🥲😤🥲😤🥲🥲😤🥲😤🥲😤🥲😤🥲😤🥲😤🥲😤😶😤😶😤😶😌😤🥲😤🥲😤🥲😬😢😏😢😏😢😏😊😏😗😢🤗🤗😢🤗🤭🤗😢😢🤗😢🤗😢😢🤗😢🤗😢🤗😢🤗

Setting up SSH login with email link-click authentication on an Ubuntu server using `ssmtp`, PHP, and `ForceCommand` is a complex setup. Below are simplified instructions on how you could implement such a solution, focusing on the necessary steps and configurations. However, please note that email link-click authentication for SSH access is unconventional and may require custom development for secure implementation.

### Steps to Achieve SSH Login with Email Link Click Authentication

#### 1. Install Required Software

Make sure you have the following installed:

- **OpenSSH server** for SSH access
- **ssmtp** for sending emails
- **PHP** for handling server-side logic

```bash
sudo apt update
sudo apt install openssh-server ssmtp php
```

#### 2. Configure ssmtp

Edit the `ssmtp` configuration to set up your email server.

```bash
sudo nano /etc/ssmtp/ssmtp.conf
```

Here's a sample configuration for Gmail. Replace with your email provider's SMTP settings:

```bash
root=postmaster@example.com
mailhub=smtp.gmail.com:587
AuthUser=your-email@gmail.com
AuthPass=your-email-password
UseTLS=YES
UseSTARTTLS=YES
FromLineOverride=YES
```

**Note**: For Gmail, you may need to enable less secure apps or set up App Passwords for enhanced security.

#### 3. Configure PHP for Sending Email

Create a PHP script to handle the email link generation and verification. Here is an example of how you can send an email with a verification link.

```php
<?php
$to = "user@example.com";
$subject = "SSH Login Link";
$token = bin2hex(random_bytes(16)); // Generate a random token for authentication
$verification_link = "http://your-server-ip/verify.php?token=$token";

// Store the token temporarily in a database or file for validation later
file_put_contents("tokens/$token.txt", "user@example.com");

$message = "Click the link below to authenticate your SSH login:\n$verification_link";

$headers = 'From: no-reply@your-server.com' . "\r\n" .
           'Reply-To: no-reply@your-server.com' . "\r\n" .
           'X-Mailer: PHP/' . phpversion();

mail($to, $subject, $message, $headers);
echo "Verification link sent to $to!";
?>
```

#### 4. Create Verification Script (`verify.php`)

This script verifies the token and allows the user to initiate the SSH login.

```php
<?php
if(isset($_GET['token'])) {
    $token = $_GET['token'];

    if(file_exists("tokens/$token.txt")) {
        $email = file_get_contents("tokens/$token.txt");
        // Start a session and store the email in it for future use
        session_start();
        $_SESSION['user_email'] = $email;

        // Redirect to a page indicating successful login or directly handle SSH session
        echo "You have been authenticated! Please proceed to SSH.";
    } else {
        echo "Invalid or expired token!";
    }
} else {
    echo "No token provided!";
}
?>
```

#### 5. Modify `sshd_config` to Use ForceCommand

You need to set up `ForceCommand` in `sshd_config` to run a custom script once the user logs in through SSH. This can ensure the user is only allowed to log in if they are authenticated via the email link.

Edit the SSH configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```

Add the following line to the `sshd_config`:

```bash
ForceCommand /usr/local/bin/ssh-login-script.sh
```

This ensures that once the SSH session is initiated, the custom script `ssh-login-script.sh` will be executed. 

#### 6. Create the SSH Login Script (`ssh-login-script.sh`)

Now, create a simple script that will check if the user has been authenticated through the email link:

```bash
sudo nano /usr/local/bin/ssh-login-script.sh
```

```bash
#!/bin/bash

# Check if the user has been authenticated via email
if [ -f "/tmp/authorized_user" ]; then
    # If authenticated, allow them to access the shell
    $SHELL
else
    # If not authenticated, close the connection
    echo "You need to authenticate via email to log in."
    exit 1
fi
```

#### 7. Implement the Email Login Validation

When the user clicks on the link in the email, you should save a session or token indicating that they are authenticated. You can create a file in `/tmp/authorized_user` to signify that the user is authorized to log in.

Modify `verify.php` to create this file:

```php
<?php
if(isset($_GET['token'])) {
    $token = $_GET['token'];

    if(file_exists("tokens/$token.txt")) {
        $email = file_get_contents("tokens/$token.txt");

        // Store the authentication status in a temporary file
        file_put_contents("/tmp/authorized_user", $email);

        session_start();
        $_SESSION['user_email'] = $email;
        echo "You have been authenticated! Please proceed to SSH.";
    } else {
        echo "Invalid or expired token!";
    }
} else {
    echo "No token provided!";
}
?>
```

#### 8. Secure the Setup

Ensure that the authentication file (`/tmp/authorized_user`) is deleted after the session ends or after a timeout.

You might want to implement additional security features, such as limiting the validity of the token, logging out after a certain period, and securing access to the `/tmp/authorized_user` file.

---

### Final Thoughts

This is a basic guide for setting up SSH login via email verification on Ubuntu using PHP and `ssmtp`. However, implementing email-based authentication for SSH is non-standard, and you should consider using proven alternatives like multi-factor authentication (MFA) or SSH keys for better security.

🙄😛😙🙄🙄😋😗🙄🙄🥹😃🙄🥹😃🫢😮‍💨🥹😃😮‍💨😄🥹🤭🥹😄😮‍💨😛🤭🥹😄🙄😮‍💨🥹😃🙄😄😛🙄😛😄😮‍💨😛😄😮‍💨😛😙😮‍💨😙😛😮‍💨😛😃😮‍💨😮‍💨😛😄😮‍💨😛😃🙄😄😛😮‍💨😛😄😮‍💨🙄😛😄🙄😄😛🙄🙄😄😛😮‍💨😛😄😮‍💨😄😛😮‍💨😛😄😮‍💨😛😄🙄😄😛🥹😄😛😃😢🙂😐😢😙😐😙😛😢😛😙😢😢😛😙😛😙😮‍💨😮‍💨😛😗😮‍💨😙😛😢😙😛😛😢😗😮‍💨😛😗😢😛😗😢😮‍💨😛😢😛😢😙😝😢😙😢😝😙😙😮‍💨😛😙😮‍💨😙😝😢😝😙

To set up SSH login using email link click authentication in an Ubuntu server with shell access, you need to achieve two things:

1. **Configure SSMTP for sending emails**: SSMTP allows you to send emails without needing a full mail server, which is perfect for this setup.
2. **Use SSH's `ForceCommand` to restrict access to a shell script that can verify the email link click**.

This example will guide you through the steps using simple `.sh` scripts and `ssmtp` for email configuration. The idea is to generate a unique token for the user, send it via email, and have the user click on a link to authenticate, which triggers the shell script to authenticate their session.

### 1. **Install and Configure `ssmtp`**

Install `ssmtp` on your Ubuntu server to send emails from the server to the user's email.

```bash
sudo apt update
sudo apt install ssmtp
```

Configure `ssmtp` by editing its configuration file:

```bash
sudo nano /etc/ssmtp/ssmtp.conf
```

Here's a basic configuration for using Gmail (you can modify this for other email services):

```
root=postmaster
mailhub=smtp.gmail.com:587
hostname=yourdomain.com
AuthUser=your-email@gmail.com
AuthPass=your-email-password
FromLineOverride=YES
UseSTARTTLS=YES
```

You will need to replace the `AuthUser` and `AuthPass` with your Gmail account and password. For added security, consider using an App Password if you're using Gmail.

### 2. **Create the Shell Script to Send the Authentication Link**

Create a script (`send_email.sh`) that generates a unique token and sends it via email.

```bash
#!/bin/bash

# Create a unique token (for simplicity, using the current timestamp)
token=$(date +%s | sha256sum | base64 | head -c 32)

# Store the token in a file
echo $token > /tmp/token.txt

# Prepare the authentication link
auth_link="http://your-server.com/verify_token?token=$token"

# Send the email with the link using ssmtp
echo -e "Subject: SSH Login Authentication\n\nClick the link to authenticate your SSH session:\n$auth_link" | ssmtp user@example.com

# Output the token (optional, for debugging purposes)
echo "Token sent: $auth_link"
```

### 3. **Set Up SSH Configuration with `ForceCommand`**

You can configure `ForceCommand` to restrict SSH login to a specific shell script that verifies the token when the user logs in.

Edit the SSH configuration file `/etc/ssh/sshd_config`:

```bash
sudo nano /etc/ssh/sshd_config
```

Add or modify these lines:

```
ForceCommand /path/to/your/verify_token.sh
AllowUsers yourusername
```

This configuration ensures that any login attempt will force the user to run the `verify_token.sh` script, instead of providing normal shell access.

### 4. **Create the Token Verification Script**

Create a script `verify_token.sh` that checks the token and allows the user to proceed if the token matches.

```bash
#!/bin/bash

# Get the token from the SSH command line arguments
provided_token=$1

# Check if the token matches the one stored
stored_token=$(cat /tmp/token.txt)

if [[ "$provided_token" == "$stored_token" ]]; then
    echo "Token validated. Access granted!"
    # Grant access to the shell (optional, you can customize further)
    /bin/bash
else
    echo "Invalid token. Access denied."
    exit 1
fi
```

### 5. **Triggering Token Verification from the Email Link**

When the user clicks the authentication link in their email, the server should verify the token. You can configure this to work with a web server (like Apache or Nginx) or even a local script that responds to HTTP requests.

Since you mentioned not using web access directly, a simple way to test is to store the tokens in a file and compare them in your shell script.

### Final Notes:

- **Security**: This method is for demonstration purposes. In a real production environment, you should implement secure methods to manage tokens (e.g., via HTTPS and cryptographic tokens).
- **Email Delivery**: Ensure that the email server can reliably deliver messages, especially when using a service like Gmail, which may flag your server as a spammer if it's sending too many emails.

Once everything is set up, you should be able to initiate the process by running the `send_email.sh` script. The user will receive an email with a link, and upon clicking, their SSH session will be validated via the `verify_token.sh` script.


🥲😍🥲😊😤☺️🙄☺️🙄☺️🙄☺️🥹😌🙄😌🙄😌🙄☺️🙄☺️🙄🙄☺️🤭😌😌☺️😌😌🙄😌🙄😌🙄😌😌😌😌😌😌😌😌😌😌😌🙄😌🙄😌😝😌😝☺️🤭☺️😋☺️🫢☺️🤭☺️🥹☺️🙄😌😶☺️😍😌🫢😌😮‍💨😌🤭😌🤩🫢😍🫢😍🤭😍🤭😍😍😍😍😝😝😍🤭😍🤭😍🫢😍😮‍💨😍😢😍😝😍🙄😍🤭🤭😍🤭😍🙄🤩
To set up SSH login on an Ubuntu server via email link click authentication with `$SHELL` access using only `ssmtp`, `.sh` scripts, and `ForceCommand` in the `sshd_config` without web access, you can follow these steps.

### Requirements:
- Ubuntu server with `ssmtp` installed.
- SSH configured to allow specific commands via the `ForceCommand` option.

### Steps:

#### 1. Install `ssmtp` on the Ubuntu server

First, install `ssmtp` (a simple mail relay system):

```bash
sudo apt update
sudo apt install ssmtp
```

#### 2. Configure `ssmtp`

Configure `ssmtp` for sending emails. The configuration file is `/etc/ssmtp/ssmtp.conf`.

Edit the configuration file:

```bash
sudo nano /etc/ssmtp/ssmtp.conf
```

Here’s an example configuration for Gmail (replace with your email provider’s details):

```bash
root=postmaster@example.com
mailhub=smtp.gmail.com:587
AuthUser=your-email@example.com
AuthPass=your-email-password
FromLineOverride=YES
UseSTARTTLS=YES
```

Ensure that the email credentials and SMTP server details are correct. **Note:** It's highly recommended to use App Passwords instead of your regular email password for security reasons if you're using Gmail.

#### 3. Create a script to send a login link

Create a shell script that sends an email with a login link. You can generate a token or use a simple authentication method, for instance, with a random string:

```bash
#!/bin/bash

# Generate a random authentication token (or use a simpler method)
AUTH_TOKEN=$(openssl rand -hex 12)

# Send an email to the user with the authentication token
echo "Click the link below to authenticate your SSH session: \
https://yourserver.com/ssh-login/$AUTH_TOKEN" | ssmtp user@example.com

# Print token for reference (in production, you may store it securely)
echo "Generated authentication token: $AUTH_TOKEN"
```

Save this script as `send_login_link.sh` and make it executable:

```bash
chmod +x send_login_link.sh
```

#### 4. Create a `ForceCommand` script

In `sshd_config`, set up the `ForceCommand` option to execute a script upon SSH login. Create a shell script (e.g., `verify_login.sh`) to verify the link or token.

```bash
#!/bin/bash

# Example: Check if the login token matches one sent in the email
if [[ "$SSH_AUTH_TOKEN" == "$EXPECTED_TOKEN" ]]; then
    # If it matches, allow the user to proceed to their shell
    exec $SHELL
else
    # If not, deny access
    echo "Invalid authentication token."
    exit 1
fi
```

Save this script as `verify_login.sh` and make it executable:

```bash
chmod +x verify_login.sh
```

#### 5. Configure SSH to use the script

In the SSH configuration file `/etc/ssh/sshd_config`, set the `ForceCommand` option to execute the `verify_login.sh` script.

```bash
sudo nano /etc/ssh/sshd_config
```

Add the following line:

```bash
ForceCommand /path/to/verify_login.sh
```

#### 6. Restart the SSH service

After making changes to the SSH configuration, restart the SSH service:

```bash
sudo systemctl restart sshd
```

#### 7. Authentication process overview:

1. The user runs `send_login_link.sh` to send an email with the authentication token.
2. The user receives the email, clicks the link (which would be a token-based authentication URL).
3. Upon SSH login, the server checks the token against a stored or generated token (from email).
4. If the token is correct, the user is granted access to `$SHELL`.

### Security Considerations:
- This approach doesn't include web-based token validation, so it relies on a pre-shared method for token verification.
- You can extend the script to store the token securely in a file or database for better validation.
- Avoid hardcoding passwords or sensitive information in scripts, especially in a production environment.

This is a simple implementation, and in a real-world environment, you should consider additional security mechanisms, like 2FA or more robust token management.
🥰😗☺️😙☺️😙😁😍😁😍😍😍😋🤤🙃🥹😊🙃🥹😊🤤🥹😊😊🤤🥹😊😊🤤🥹🥰☺️🤤🤭☺️😍🙂‍↔️😋😍🤭☺️🙂‍↔️🤭😍😋🙂‍↔️😂😃😶😂🥱😜😶‍🌫️😂😂😃😶‍🌫️🥱😂😶‍🌫️🥱😂😶‍🌫️😂😶‍🌫️😃😂😃😶‍🌫️😍😂😂🙃😶‍🌫️😍🙃🙃😶‍🌫️😍🙃😶‍🌫️😶‍🌫️🙃😍😍🙃😶‍🌫️😍🙃🙃😶‍🌫️😍🥱😶‍🌫️😍😍🙃😶‍🌫️🙃😍😶‍🌫️😍🙃😶‍🌫️😍🙃😶‍🌫️🙃😶‍🌫️😍🙃😶‍🌫️🤩🙃😶‍🌫️🙃🤩😶‍🌫️😍😶‍🌫️🙃🤩😶‍🌫️😍🥱😶‍🌫️🙃😶‍🌫️😍🙃😶‍🌫️😍🥱😶‍🌫️😶‍🌫️🥱😍😶‍🌫️😍🥱😶‍🌫️😶‍🌫️😍🥱😶‍🌫️😶‍🌫️😍🥱😶‍🌫️😍🥱😶
To achieve SSH login authentication via email link click with `$SHELL` access using `ssmtp` and simple shell scripts on Ubuntu, you would need to implement a custom solution. Here's how you can approach this:

### 1. Prerequisites
- Ubuntu server running SSH.
- `ssmtp` installed for sending email notifications.
- Email service to send the link for SSH login.
- A basic shell script for generating a login link.
- Use of `ForceCommand` in the `sshd_config` for security.

### Step-by-Step Solution:

#### Step 1: Install `ssmtp` and Configure it
Install `ssmtp` for email sending.

```bash
sudo apt update
sudo apt install ssmtp
```

#### Step 2: Configure `ssmtp` (`/etc/ssmtp/ssmtp.conf`)
Edit `ssmtp.conf` to configure your mail settings. You need to use a working SMTP server (e.g., Gmail, SendGrid, etc.).

```bash
sudo nano /etc/ssmtp/ssmtp.conf
```

Here’s an example for Gmail:

```bash
root=postmaster
mailhub=smtp.gmail.com:587
AuthUser=your_email@gmail.com
AuthPass=your_email_password
UseTLS=YES
UseSTARTTLS=YES
FromLineOverride=YES
```

#### Step 3: Create a Script to Send Email with a Login Link (`/usr/local/bin/send_login_link.sh`)
You need a script that generates a unique login link, sends it via email, and sets up the temporary SSH access.

```bash
#!/bin/bash

# Generate a unique link/token (could use UUID or any other method)
TOKEN=$(uuidgen)

# Set your email and recipient
FROM_EMAIL="your_email@gmail.com"
TO_EMAIL="recipient_email@example.com"

# Prepare the login link (this could point to a script or page that validates the token)
LOGIN_LINK="ssh://$USER@$HOSTNAME?token=$TOKEN"

# Send the email with the login link
echo -e "Subject: SSH Login Link\n\nClick on the link to login to your server:\n$LOGIN_LINK" | ssmtp $TO_EMAIL
```

Make the script executable:

```bash
sudo chmod +x /usr/local/bin/send_login_link.sh
```

#### Step 4: SSH Configuration (`/etc/ssh/sshd_config`)
Modify SSH settings to force the login to run a specific command for validation when the user clicks the link.

Edit `sshd_config`:

```bash
sudo nano /etc/ssh/sshd_config
```

Ensure you have these settings:

```bash
ForceCommand /usr/local/bin/validate_login.sh
AllowUsers your_username
PasswordAuthentication no
```

The `ForceCommand` ensures that every SSH login will run the script `/usr/local/bin/validate_login.sh`, which will verify the login link.

#### Step 5: Create the Validation Script (`/usr/local/bin/validate_login.sh`)
This script will be responsible for validating the token received from the link.

```bash
#!/bin/bash

# Extract the token passed via SSH
TOKEN=$1

# Validate the token (you could store tokens in a file or database)
VALID_TOKEN="your_valid_token_here"

if [[ "$TOKEN" == "$VALID_TOKEN" ]]; then
  # Allow the user to continue with the shell
  exec $SHELL
else
  echo "Invalid token!"
  exit 1
fi
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/validate_login.sh
```

#### Step 6: Send the Login Link
Finally, you can run the `send_login_link.sh` script to send the login link to the recipient:

```bash
/usr/local/bin/send_login_link.sh
```

The recipient will get an email with the SSH login link. When they click the link, it will pass the token to the SSH login process, where the token will be validated. If valid, the user will be granted access to the shell.

### Additional Considerations:
- **Security**: This approach is highly simplified. In a production environment, you would need a more robust method for validating tokens (e.g., with an expiry time, or by storing tokens securely).
- **Error Handling**: Add proper error handling and logging in the scripts to ensure smooth operations.
- **Link Validation**: This example uses a static token. You may want to implement a more secure, time-sensitive token system (e.g., using time-based one-time passwords or a hashed method).

This solution provides basic functionality to authenticate SSH via email link and `$SHELL` access with `ssmtp`.

😌🥱🥺😐😏😯😐😏😯😏😯😐😏😲😶😏😲😶😏😲😲😐😏😯😲😔😐😯😲😯😯🥺😯😶😏😏😲😲😶😏😲😏😲😯😏😲😏😲🥱🥺😲😲😶🥺😲😶😏😲😶😏😲😶😏😯😶😏😲😏🥱😲🥱😏☹️😏☹️🥱😏😲😶😏😲🥱😏☹️😶😏☹️😶😏☹️😶😏☹️😶😏☹️😶😏😯😶😏😶😏😲😶😏😲😶😏😲😶😏😲😏😲😐🫠😯😐🙃😲😶🫠😲🥱😏😲😶😏😯😏😶😏😲😏😶😲😶😏😲😶😏😲😲😶😏😶😏😶😲😏😶😏😳

Certainly! Here's a detailed guide to set up SSH login with email-based verification using a verification link, incorporating all necessary files and configurations. This setup uses **SSH**, **ssmtp**, a **custom verification script**, and a **web server** to validate the token sent to the user's email. The steps are comprehensive and include everything you need for a secure setup.

### 1. Prerequisites

Ensure that the following packages are installed on your Ubuntu server:

- **OpenSSH** for SSH access.
- **ssmtp** for sending emails.
- **Apache2** for hosting the verification page.
- **PHP** for processing the verification page.
- **OpenSSL** for generating random tokens.

Run the following commands to install the required packages:

```bash
sudo apt update
sudo apt install openssh-server ssmtp apache2 openssl php
```

### 2. Configure SSH (`/etc/ssh/sshd_config`)

To make SSH use the custom verification script (`verify_login.sh`), edit the SSH server configuration.

```bash
sudo nano /etc/ssh/sshd_config
```

Make the following changes:

```bash
# /etc/ssh/sshd_config

# ForceCommand to run the verification script during SSH login
ForceCommand /path/to/verify_login.sh

# Allow empty passwords for this specific method (safer methods can be used in real scenarios)
PermitEmptyPasswords yes

# Ensure other methods like password and public key authentication are disabled
PasswordAuthentication no
PubkeyAuthentication yes
```

Replace `/path/to/verify_login.sh` with the actual path to your custom verification script.

### 3. Create the Verification Script (`verify_login.sh`)

The script `verify_login.sh` will generate a unique verification token, send an email with a verification link, and wait for the user to click the link before granting SSH access.

Create the script:

```bash
sudo nano /path/to/verify_login.sh
```

Add the following content to the script:

```bash
#!/bin/bash

# /path/to/verify_login.sh

# Step 1: Generate a one-time verification token (base64 encoded random string)
VERIFICATION_TOKEN=$(openssl rand -base64 32)

# Step 2: Send the email with the verification link using ssmtp
EMAIL="user@example.com"  # Replace with the user's email address
VERIFICATION_URL="https://yourserver.com/verify.php?token=$VERIFICATION_TOKEN"

# Email body content
EMAIL_BODY="To: $EMAIL
Subject: SSH Login Verification

Please click on the following link to verify your SSH login:
$VERIFICATION_URL"

# Send the email
echo "$EMAIL_BODY" | ssmtp $EMAIL

# Step 3: Inform the user that an email has been sent
echo "A verification link has been sent to your email. Please click the link to continue."

# Step 4: Wait for the user to verify the token by checking for a specific file
while [ ! -f "/tmp/$VERIFICATION_TOKEN" ]; do
  sleep 1
done

# Step 5: Once the token is verified, allow the user to access the shell
exec $SHELL
```

Make the script executable:

```bash
sudo chmod +x /path/to/verify_login.sh
```

### 4. Create the Web Verification Handler (`verify.php`)

You need a PHP script that will handle the verification when the user clicks the verification link in the email. This script will check if the token exists and is valid.

Create the PHP verification page:

```bash
sudo nano /var/www/html/verify.php
```

Add the following content to the file:

```php
<?php
// /var/www/html/verify.php

// Check if the token is provided in the URL
if (isset($_GET['token'])) {
    $token = $_GET['token'];

    // Check if the token file exists (it means the user has clicked the verification link)
    $tokenFile = "/tmp/$token";
    $expiryTime = 600;  // Token expiration time in seconds (10 minutes)

    if (file_exists($tokenFile)) {
        // Check if the token has expired
        $fileModifiedTime = filemtime($tokenFile);
        if (time() - $fileModifiedTime > $expiryTime) {
            echo "This verification link has expired.";
            unlink($tokenFile);  // Clean up the expired token
        } else {
            // Verification successful
            echo "Verification successful! You can now log in.";
            touch($tokenFile);  // Create the file to signal SSH to allow login
        }
    } else {
        echo "Invalid or expired token.";
    }
} else {
    echo "No token provided.";
}
?>
```

### 5. Configure `ssmtp` for Email Sending (`/etc/ssmtp/ssmtp.conf`)

Configure **ssmtp** to send emails using your email provider's SMTP settings.

Edit the ssmtp configuration file:

```bash
sudo nano /etc/ssmtp/ssmtp.conf
```

Add the following configuration (modify with your SMTP details):

```bash
# /etc/ssmtp/ssmtp.conf

root=postmaster
mailhub=smtp.yourmailserver.com:587
UseSTARTTLS=YES
AuthUser=your-email@example.com
AuthPass=your-email-password
FromLineOverride=YES
```

Replace `smtp.yourmailserver.com`, `your-email@example.com`, and `your-email-password` with your SMTP provider’s details.

### 6. Ensure Apache2 is Running

Make sure the Apache2 service is running so that the `verify.php` page can be accessed:

```bash
sudo systemctl enable apache2
sudo systemctl start apache2
```

Test the web server by navigating to `http://yourserver.com/verify.php` in a browser. It should show "No token provided."

### 7. Restart SSH Service

After making the necessary changes, restart the SSH service to apply the new configuration:

```bash
sudo systemctl restart sshd
```

### 8. Security Considerations

1. **HTTPS**: Make sure the verification link is served over HTTPS to secure the token during transmission.
2. **Token Expiry**: In the PHP script, I included a mechanism for token expiration (10 minutes).
3. **Token Cleanup**: The token file is cleaned up automatically in the PHP script once it's expired or used. You can extend the cleanup functionality as needed.

### 9. Testing

1. Try logging in via SSH. The server should send an email with the verification link.
2. The user should click the link, and upon verification, SSH access will be granted.

### Final Thoughts

- **Email Security**: Ensure the email provider used in `ssmtp` supports TLS to encrypt email transmissions.
- **Token Expiry**: You can adjust the expiration time and add more sophisticated cleanup logic as needed.
- **Improving Security**: For a production environment, consider integrating more secure and robust solutions like multi-factor authentication (MFA) or hardware tokens.

By following these steps, you’ll have an SSH login system that requires email-based verification for access, making your server more secure against unauthorized access.


🥹🥹😐🥹😃🤭🥹😃😐😃😐🥹😃🥱😗🥹🥱😗😗🥱😗🤭🥱🥹😗🤭🤭🥹😗🤭🤭🥹😃🤭🥹😀🤭😘😃🤭🥹😃🤭🥱🥹😃🥱🤭🥹😃🤭😃🥱🥹😃🤭🥹😃🥱🥹😃🤭🥹😃🤭🥱🥹😃🤭🥱🥹😃🤭🥹😃🤭🥱🥹😃🥱🥱🥹😃🤭🤭🥹😃🤭🥹😃🤭😘😃🤭😃😘🥱😘😃🥱😃🥱🥹😃🥱🥱🥹😃🥱😃🥱😃🥹🥱🤭🥹😃🥱🥱🥹😃🥱😃😊🥱😃😤😊🥱🥹😃🥱🥹😃🥱😃🥹🥱🥹😃🥹😃🥹🤭🥹😃🥱😘😃🤭😘😃🤭😃😘🤭🥹😃😘🤭😃🥹😃🤭😘😀🤭😐😘😀🤭🤭😘😃🤭🤭😃😐🤭😘😃😐😘😀😐😃😐😘😃🤭🤭😃🥹😀🤭🤭😘😀🤭🤭

To set up SSH login with email link verification on an Ubuntu server, here's a detailed, complete, and correct guide incorporating **SSH**, **ssmtp**, a **custom verification script**, and a **web server** for token-based verification. This solution ensures that SSH access is only granted after the user clicks on a verification link sent to their email. This guide assumes you have basic Linux and server management knowledge.

### 1. Prerequisites

Ensure the following packages are installed:

- **OpenSSH** for SSH access
- **ssmtp** for sending emails
- **Apache2** for hosting the verification page
- **OpenSSL** for token generation

Run the following command to install them:

```bash
sudo apt update
sudo apt install openssh-server ssmtp apache2 openssl php
```

### 2. Configure SSH (`/etc/ssh/sshd_config`)

To configure SSH to run a custom script that will handle the email-based verification, edit the SSH configuration file.

```bash
sudo nano /etc/ssh/sshd_config
```

Make the following changes (add the `ForceCommand` to run the verification script and ensure `PermitEmptyPasswords` is enabled for this method):

```bash
# /etc/ssh/sshd_config

# ForceCommand to execute a custom script for login verification
ForceCommand /path/to/verify_login.sh

# Permit empty passwords if using email link (you can modify this for better security)
PermitEmptyPasswords yes

# Allow authentication using keys or other methods
PasswordAuthentication no
PubkeyAuthentication yes
```

Replace `/path/to/verify_login.sh` with the actual path to your verification script.

### 3. Create the Verification Script (`verify_login.sh`)

Create the custom script `/path/to/verify_login.sh`. This script will:

1. Generate a one-time token.
2. Send an email to the user with a verification link.
3. Wait for the user to verify the token by clicking the link in the email.
4. Once the token is verified, grant access to the shell.

```bash
sudo nano /path/to/verify_login.sh
```

Add the following content:

```bash
#!/bin/bash

# /path/to/verify_login.sh

# Step 1: Generate a one-time verification token (base64 encoded random string)
VERIFICATION_TOKEN=$(openssl rand -base64 32)

# Step 2: Send email with verification link using ssmtp
# Modify the recipient email and the verification link URL accordingly
EMAIL="user@example.com"  # Replace with the actual email of the user
VERIFICATION_URL="https://yourserver.com/verify.php?token=$VERIFICATION_TOKEN"

# Email body
EMAIL_BODY="To: $EMAIL
Subject: SSH Login Verification

Please click on the following link to verify your SSH login:
$VERIFICATION_URL"

# Send the email using ssmtp
echo "$EMAIL_BODY" | ssmtp $EMAIL

# Step 3: Inform the user that an email has been sent
echo "A verification link has been sent to your email. Please click the link to continue."

# Step 4: Wait for user to verify the token (create the file to signal completion)
while [ ! -f "/tmp/$VERIFICATION_TOKEN" ]; do
  sleep 1
done

# Step 5: Once the token is verified, allow the user to access the shell
exec $SHELL
```

Make the script executable:

```bash
sudo chmod +x /path/to/verify_login.sh
```

### 4. Create the Web Verification Handler (`verify.php`)

On your web server, create a PHP script to handle the verification link. This script checks if the token is valid and creates the file signaling that the user has verified their email.

**Location**: `/var/www/html/verify.php`

```bash
sudo nano /var/www/html/verify.php
```

Add the following content:

```php
<?php
// /var/www/html/verify.php

// Check if the token is provided
if (isset($_GET['token'])) {
    $token = $_GET['token'];

    // Check if the token exists (it means the user has clicked the verification link)
    if (file_exists("/tmp/$token")) {
        echo "Verification successful! You can now log in.";
        
        // Create the file to signal SSH that verification is complete
        touch("/tmp/$token");
    } else {
        echo "Invalid or expired token.";
    }
} else {
    echo "No token provided.";
}
?>
```

### 5. Configure `ssmtp` for Email Sending (`/etc/ssmtp/ssmtp.conf`)

Next, configure **ssmtp** to send emails. You need to set your email provider's SMTP settings in `/etc/ssmtp/ssmtp.conf`. This is necessary for sending the verification email.

Edit the file:

```bash
sudo nano /etc/ssmtp/ssmtp.conf
```

Add the following configuration (modify the values accordingly):

```bash
# /etc/ssmtp/ssmtp.conf

root=postmaster
mailhub=smtp.yourmailserver.com:587
UseSTARTTLS=YES
AuthUser=your-email@example.com
AuthPass=your-email-password
FromLineOverride=YES
```

Replace `smtp.yourmailserver.com`, `your-email@example.com`, and `your-email-password` with the credentials for your SMTP provider.

### 6. Ensure Apache2 is Running

Make sure Apache2 is running and serving your PHP verification script:

```bash
sudo systemctl enable apache2
sudo systemctl start apache2
```

Test the web server by navigating to `http://yourserver.com/verify.php` in a browser. If it shows "No token provided," your server is working correctly.

### 7. Final Steps: Restart SSH

After making all the necessary changes, restart the SSH service to apply the new configuration:

```bash
sudo systemctl restart sshd
```

### 8. Security Considerations

1. **Token Expiry**: Add an expiry mechanism for tokens. You can set a timeout (e.g., 10 minutes), and if the token file isn't created within that time, the user should be asked to try again.

2. **HTTPS**: Ensure that the verification page (`verify.php`) is served over HTTPS to protect the token from being intercepted.

3. **Cleanup**: Periodically clean up expired tokens or handle cleanup in your PHP script by deleting tokens after use.

### 9. Token Expiry (Optional)

To implement a basic token expiration mechanism, modify your PHP verification script to check for token age.

Edit `/var/www/html/verify.php`:

```php
<?php
// /var/www/html/verify.php

if (isset($_GET['token'])) {
    $token = $_GET['token'];
    $tokenFile = "/tmp/$token";
    $expiryTime = 600;  // Token expiration time in seconds (e.g., 10 minutes)

    // Check if the token exists
    if (file_exists($tokenFile)) {
        // Check if the token has expired
        $fileModifiedTime = filemtime($tokenFile);
        if (time() - $fileModifiedTime > $expiryTime) {
            echo "This verification link has expired.";
            unlink($tokenFile);  // Optionally clean up the expired token
        } else {
            echo "Verification successful! You can now log in.";
            touch($tokenFile);  // Allow SSH login by updating the file
        }
    } else {
        echo "Invalid or expired token.";
    }
} else {
    echo "No token provided.";
}
?>
```

This will make the token expire 10 minutes after creation.

### Summary

1. **SSH Configuration**: Forces SSH login to execute a custom verification script (`verify_login.sh`).
2. **Verification Script**: Generates a token, sends an email with the verification link, and waits for the user to verify the token.
3. **Web Server Script**: A PHP script handles the link click and verifies the token.
4. **Email Sending**: The `ssmtp` tool sends the verification email.
5. **Security Considerations**: Ensure secure email transport, HTTPS for the verification link, and a token expiration mechanism.

By following these steps, SSH access will be granted only after the user clicks the verification link sent to their email, effectively providing email-based login for SSH.




😃😃😄😙🙃🙃🙃😙😁😁😄😄😙😚😗😗😁😄😙😄😁😙😙😙😙😄😁😆😆😄😙😚😚😁😁😁😁😙😙😙😙😙😙😙😙😁😁😙😙😙😙😙😚😁😁😚😚😚😘😘😆😅😅😘😆😘😆😅😅😘😚😙😆😁😁😚😚😆😆😁😄😄😙😚😘😆😚😚😚😚😚😁😆😚😚😚😚😁😚😚😚😁😆😆😁😁😁😔🥺😤😤😳😏😊😊😚😊😊😊😚😚😚😚😚😚😚😚😚😳😳😚😚😚😚😳😳😳😳😳😳😳😳😳😳😳😳😳😳😳
To implement an SSH login system that uses email-based verification without a database, where the process is simple and relies on a unique token sent to the user’s email, here is a more complete and self-contained approach. This method will not require any database but instead use a local file to store tokens temporarily for the verification process. After the user clicks the verification link, the script will validate the token and proceed with SSH access.

### Overview:
1. **Configure SSH Server for email-based verification**.
2. **Set up email configuration (`ssmtp.conf`)**.
3. **Write the verification shell script (`verify.sh`)**.
4. **Set up the email sending and verification process**.
5. **Implement verification token validation with a local file**.
6. **Configure SSH to force running the verification script**.

### Step 1: Configure SSH Server (`/etc/ssh/sshd_config`)

1. Open your SSH configuration file:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Force the execution of the `verify.sh` script upon SSH login and disable password authentication:
   ```bash
   # Force the execution of a verification script
   ForceCommand /path/to/verify.sh

   # Disable password authentication (you can use key-based authentication if desired)
   PasswordAuthentication no

   # Allow only public key authentication
   PubkeyAuthentication yes

   # Restart the SSH service
   sudo systemctl restart sshd
   ```

### Step 2: Configure Email Sending with `ssmtp.conf`

Set up `ssmtp` (or any other simple mail transfer agent) to send verification emails.

1. Open the `ssmtp` configuration file:
   ```bash
   sudo nano /etc/ssmtp/ssmtp.conf
   ```

2. Configure the email details (example using Gmail):
   ```bash
   # SSMTP Configuration for Gmail
   root=your_email@gmail.com
   mailhub=smtp.gmail.com:587
   hostname=yourdomain.com
   UseSTARTTLS=YES
   AuthUser=your_email@gmail.com
   AuthPass=your_app_specific_password
   FromLineOverride=YES
   ```

   **Important**: Use an [app-specific password](https://support.google.com/accounts/answer/185833?hl=en) if you have 2FA enabled on Gmail.

### Step 3: Write the Verification Shell Script (`verify.sh`)

This script will send a verification email with a unique token. After clicking the link, the token will be validated, and the user can gain access to the shell.

#### Example `verify.sh`:

```bash
#!/bin/bash

# Extract the email address (from the first argument passed to SSH)
USER_EMAIL="$1"

# Create a temporary token (UUID or random string)
TOKEN=$(uuidgen)  # Or use another method to generate a unique token

# Store the token temporarily in a file for validation later
echo "$TOKEN,$USER_EMAIL" >> /tmp/ssh_tokens.txt

# Generate the verification URL (this assumes you have a web service handling the token validation)
VERIFICATION_LINK="http://yourdomain.com/verify?token=$TOKEN"

# Send the verification email
echo -e "Subject: SSH Login Verification\n\nPlease click the following link to verify your SSH login:\n\n$VERIFICATION_LINK" | ssmtp "$USER_EMAIL"

# Inform the user that a verification email has been sent
echo "A verification email has been sent to $USER_EMAIL. Please click the link in your inbox."

# Wait for the user to verify the token
while true; do
    # Check if the token is valid (this is simulated by checking the file)
    if grep -q "$TOKEN" /tmp/ssh_tokens.txt; then
        # If the token is valid, delete it from the temporary file
        sed -i "/$TOKEN/d" /tmp/ssh_tokens.txt
        break  # Exit the loop and proceed to the shell
    fi
    sleep 5
done

# If the token is valid, allow the user to access the shell
exec $SHELL
```

### Step 4: Web Server for Token Validation

You need a simple web service to handle the verification when the user clicks the link. This can be done with a basic PHP, Python, or Node.js script.

#### Example PHP Script (`verify.php`):

Create a simple PHP script to handle the token verification when the user clicks the link in their email.

```php
<?php
// Simple PHP verification script

if (isset($_GET['token'])) {
    $token = $_GET['token'];

    // Check if the token exists in the temporary file (for simplicity, we check a flat file here)
    $token_file = '/tmp/ssh_tokens.txt';

    // Read the file and check if the token exists
    $tokens = file($token_file, FILE_IGNORE_NEW_LINES);
    foreach ($tokens as $line) {
        list($stored_token, $stored_email) = explode(',', $line);
        if ($stored_token == $token) {
            // If the token is valid, mark it as verified
            echo "Verification successful! You may now proceed with your SSH login.";
            // In real scenarios, you can also remove the token here or mark it as used
            exit();
        }
    }

    echo "Invalid or expired verification token.";
} else {
    echo "No token provided.";
}
?>
```

Ensure this PHP script is accessible at `http://yourdomain.com/verify`.

### Step 5: Allow SSH Access After Token Validation

After the user clicks the verification link and the token is validated, the `verify.sh` script will allow SSH access. The `ForceCommand` option in the SSH configuration ensures that users can only run the `verify.sh` script and not bypass it.

### Step 6: Clean-up and Security Measures

1. **Token Expiry**: You can add a time expiry check to the `verify.sh` script by modifying the file structure. For example, each entry in the `/tmp/ssh_tokens.txt` file could include a timestamp. If the token is older than a certain duration, it could be invalidated.

2. **HTTPS**: Ensure the verification link is served over HTTPS to prevent token interception.

3. **Log out tokens after successful verification**: To further secure the system, after the user verifies their token, you should remove or mark the token as used in the `/tmp/ssh_tokens.txt` file.

### Conclusion

This solution provides SSH login with email-based verification using a unique token. When a user logs into the system, they are sent a verification link via email. Upon clicking the link, the user is verified, and the SSH session is allowed to continue. This method is simple and does not require a database, using a temporary file for token storage and validation. The use of `ForceCommand` ensures that no user can bypass the verification process.

😗😗😊😙😊😙😊😚😊😙😍😙😙🥰😙🥰😙🥹😙🥹😃😙😘😄🥰😃🥰😃🥰😙🥰😚😍😍😃😍😃😍😄😍😄😍😄😍😚😍😚😍😄😍😄😍😄😍😁😁😍😁😍😚😍😁🥰😁🥰😁🥰😄😍😚😍😚😍😙😍😄😍😄😍😄😍😁😍😄😍😄😄😍😄😍😚😍😄😍😄😍😁😍😄😍😚😍😚😚😍😚😍😁😍😁😍😆😍😅🤩😅🤩😄🤩😄😍😄😍😚😍😚😍😁😍😁☺️🥲☺️😙☺️😄😍😁😍😁😄😍😁😍😁😍😄😍😄😍😄😍

To create an SSH login process that uses email-based verification via a browser link, we can expand the initial setup to include a PHP-based web interface for users to verify the SSH login request. This involves setting up both server-side email verification (via `ssmtp`) and browser-based verification (using a PHP script to handle the verification URL and trigger the shell access).

Here’s how we can break this down:

### Step-by-Step Guide

---

### Step 1: Configure `ssmtp` for Sending Email Verification Link

Ensure `ssmtp` is properly configured to send the verification link to the user.

1. **Install `ssmtp` (if not installed)**:
   ```bash
   sudo apt-get install ssmtp
   ```

2. **Configure `ssmtp` in `/etc/ssmtp/ssmtp.conf`**:

   Here’s an example configuration for Gmail:

   ```bash
   root=postmaster
   mailhub=smtp.gmail.com:587
   AuthUser=your-email@gmail.com
   AuthPass=your-email-password
   UseTLS=YES
   UseSTARTTLS=YES
   FromLineOverride=YES
   ```

   Replace `your-email@gmail.com` and `your-email-password` with your actual credentials. For better security, use an app password if you're using Gmail.

---

### Step 2: Configure SSH to Use `ForceCommand`

To trigger a custom verification script on SSH login, configure the `sshd_config` file to use `ForceCommand`.

1. **Edit `/etc/ssh/sshd_config`**:
   
   Add the following lines to the config file:

   ```bash
   ForceCommand /path/to/verification.sh
   AllowUsers your-username
   PermitEmptyPasswords no
   ```

   Replace `/path/to/verification.sh` with the path to your verification script, and `your-username` with your actual username.

2. **Restart the SSH service**:
   
   After editing the config, restart the SSH service to apply changes:

   ```bash
   sudo systemctl restart sshd
   ```

---

### Step 3: Create the Verification Script (`verification.sh`)

This script will send a verification link via email and wait for the user to click the link on a web page to confirm their identity. 

#### `/path/to/verification.sh`:

```bash
#!/bin/bash

# Define a random verification code
verification_code=$(openssl rand -base64 12)
verification_url="https://yourdomain.com/verify.php?code=$verification_code"

# Send the email with the verification code and URL
email_subject="SSH Login Verification"
email_body="Click the link to verify your login: $verification_url"
echo -e "Subject:$email_subject\n\n$email_body" | ssmtp recipient-email@example.com

# Output the message to the user
echo "A verification email has been sent. Please check your inbox and click the verification link to continue."

# Wait for the user to verify (this could involve a database/API check in a real system)
while true; do
    # Here we simulate a check for the verification code (in a real system, this could be checking a database or a log file)
    if [ -f "/tmp/verified_$verification_code" ]; then
        break
    fi
    sleep 10
done

# After successful verification, grant shell access
exec /bin/bash
```

Explanation:
- **Verification Code Generation**: The script generates a random code that is embedded into a URL (`verification_url`).
- **Email Sending**: It sends an email with the link that the user will click to verify their login.
- **Waiting for Verification**: The script waits for a signal (e.g., a file creation) indicating that the user has verified their login by clicking the link.

---

### Step 4: PHP Web Interface for Verification

Next, you need a PHP script that handles the verification process when the user clicks on the link in the email.

#### Set up a PHP script (`verify.php`) to handle verification requests.

1. **Install Apache, PHP, and MySQL** (if not installed already):

   ```bash
   sudo apt-get install apache2 php mysql-server libapache2-mod-php
   sudo systemctl start apache2
   sudo systemctl enable apache2
   ```

2. **Create the `verify.php` script**:

   Place the following code in `/var/www/html/verify.php`:

   ```php
   <?php
   // This should be stored securely, using sessions or a database
   $verification_code = $_GET['code'];

   if (!empty($verification_code)) {
       // Here we simulate the verification process by creating a file to signal success
       $verification_file = "/tmp/verified_$verification_code";

       // You could add additional logic to check the code against a database or a file-based store
       file_put_contents($verification_file, "verified");

       echo "Verification successful! You can now return to your SSH session.";
   } else {
       echo "Invalid verification code.";
   }
   ?>
   ```

   or if you want to delete token after successful verification then  use below command

   ```php

      <?php
     // This should be stored securely, using sessions or a database
     $verification_code = $_GET['code'];

    if (!empty($verification_code)) {
    // Here we simulate the verification process by creating a file to signal success
    $verification_file = "/tmp/verified_$verification_code";

    // You could add additional logic to check the code against a database or a file-based store
    file_put_contents($verification_file, "verified");

    // Verification successful message
    echo "Verification successful! You can now return to your SSH session.";

    // Automatically delete the verification file after a short delay
    // Optionally you can adjust the delay time, for instance sleep(10) will wait for 10 seconds
    sleep(5); // Wait for 5 seconds before deleting the file
    unlink($verification_file); // Delete the file

   } else {
    echo "Invalid verification code.";
   }
   ?>

    
   
   ```


   another format almost same

   ```php

       <?php
        // Check if a code is provided in the URL
        if (isset($_GET['code']) && !empty($_GET['code'])) {
        // Sanitize the input to avoid potential security risks (e.g., directory traversal)
        $verification_code = basename($_GET['code']); // Get the base name to avoid directory traversal attacks

        // Define the path to save the file
        $destination_path = "/tmp/verified_$verification_code";

        // Simulate the verification process: create the file at the destination
        if (file_put_contents($destination_path, "verified")) {
        // Success message
        echo "Verification successful! The verification file has been created.";

        // Optionally, set a delay before deleting the verification file
        sleep(5); // Wait for 5 seconds before deleting the file
        
        // Delete the verification file after the delay
        unlink($destination_path); // Remove the file

        // Inform the user that the file has been deleted
        echo " The verification file has been deleted.";
        } else {
        // Failure to create the file
        echo "Failed to create the verification file. Please try again.";
        }
  
        } else {
       // Error if no code is provided in the URL
       echo "Invalid or missing verification code.";
       }
       ?>


   ```

   use above file with sudo access in php

```php

    <?php
// Check if a code is provided in the URL
if (isset($_GET['code']) && !empty($_GET['code'])) {
    // Sanitize the input to avoid potential security risks (e.g., directory traversal)
    $verification_code = basename($_GET['code']); // Get the base name to avoid directory traversal attacks

    // Define the path to save the file
    $destination_path = "/tmp/verified_$verification_code";

    // Command to create the file with sudo
    $create_command = "echo 'verified' | sudo tee $destination_path > /dev/null";

    // Execute the command with sudo
    $create_result = shell_exec($create_command);

    // Check if the file was successfully created
    if ($create_result !== null) {
        // Success message
        echo "Verification successful! The verification file has been created.";

        // Optionally, set a delay before deleting the verification file
        sleep(5); // Wait for 5 seconds before deleting the file

        // Command to delete the file with sudo
        $delete_command = "sudo rm -f $destination_path";

        // Execute the delete command
        shell_exec($delete_command);

        // Inform the user that the file has been deleted
        echo " The verification file has been deleted.";
    } else {
        // Failure to create the file
        echo "Failed to create the verification file. Please try again.";
    }
} else {
    // Error if no code is provided in the URL
    echo "Invalid or missing verification code.";
}
?>



```


sudo permissions: The web server user (e.g., www-data on Apache) must be granted permission to execute the commands with sudo without requiring a password. You can set this up by modifying the sudoers file (via visudo), but this should be done with extreme caution. For example, adding a line like this:

   Explanation:
   - This script checks for a `code` parameter in the URL.
   - If the code is present and valid, it creates a temporary file (`/tmp/verified_<code>`) that the SSH login script (`verification.sh`) will check to proceed with shell access.

4. **Configure Web Server**:
   Make sure the `Apache` web server is running and accessible via `https://yourdomain.com/verify.php`.

```php
www-data ALL=(ALL) NOPASSWD: /bin/tee, /bin/rm
```
---

### Step 5: Ensure the Full Flow Works

1. **Triggering the Email**:
   When a user attempts to SSH into the server, they will be prompted with a message indicating that a verification email has been sent to them.

2. **Verification via Web Link**:
   The user will check their email and click the verification link (e.g., `https://yourdomain.com/verify.php?code=<verification_code>`).

3. **Verification Success**:
   The PHP script will create a file (`/tmp/verified_<code>`) when the user clicks the link, signaling to the `verification.sh` script that the user has verified their identity.

4. **Allow Shell Access**:
   Once the `verification.sh` script detects that the user has verified their identity, it will allow the user to access the shell (`exec /bin/bash`).

---

### Additional Notes:

- **Security Considerations**:
  - Use HTTPS for the verification page (`verify.php`) to ensure secure communication.
  - Store and handle sensitive data like verification codes securely, e.g., in a database with proper access controls.
  - Use session management or database to track which codes have been used for verification.

- **Scalability**:
  - If you need to handle multiple users or concurrent logins, store verification codes in a database and track their statuses (e.g., verified/unverified).
  - Ensure the verification URL is unique to each session or login attempt.

This setup provides an SSH login flow where the user must verify their identity via an email link and a browser interface, combining SSH, email, PHP, and system scripting in a complete solution.
😗😊😗😊😙😊😄😊😄😊😃😊😃😊😄😄😊😄😊😁😊😁😊😁😄😊😄😊😄😊😄😊😚😊😚😊😄😊😄😄😊😄😊😚😊😚😊😚☺️😆☺️😆😚☺️😚☺️😙☺️😙😊😙☺️😗☺️😃😊😃😊😄😊😃😃😊😃😊😙😊😙☺️😗☺️😙☺️😗☺️😃😃☺️☺️😄☺️😃☺️😃☺️😄☺️😄☺️😄☺️😙☺️☺️😙😙☺️😙☺️😄☺️😄☺️😄☺️😃☺️😃☺️😗☺️😙☺️😗☺️😃☺️😃☺️😃☺️😃☺️😃☺️😃😃☺️😃☺️😗☺️😗☺️😗☺️😗☺️😗☺️😗☺️😗☺️😗☺️😗☺️😃☺️😃☺️😃😃☺️😃☺️😃☺️😃☺️😃☺️😃☺️😃☺️😃☺️😃☺️😃☺️😃☺️😃☺️😃☺️😗☺️😃☺️😃☺️😃😗

To implement a more secure and complete SSH login verification process via email link with token expiration and read/write permissions, you’ll need to enhance the script logic and the system components. Here’s an improved, secure version of the setup.

### Key Features:
1. **Token Expiration**: The token will expire after a set period (e.g., 15 minutes).
2. **Token File Read/Write Permissions**: Tokens and other session data will be stored securely with proper file permissions.
3. **Email with Verification Link**: The user receives a link to verify their login.
4. **SSH Access Control**: Only after successful email verification will the user be allowed to access the shell.

### Updated Setup Steps

---

### 1. **Configure SSH Server (`sshd_config`)**

First, ensure SSH is set to use the verification script by disabling password authentication and enforcing the use of the custom script.

```bash
# /etc/ssh/sshd_config

PasswordAuthentication no
ChallengeResponseAuthentication yes
ForceCommand /path/to/your/verify_login.sh  # Specify the script to handle verification
```

Restart the SSH service:

```bash
sudo systemctl restart sshd
```

---

### 2. **Create the Verification Script (`verify_login.sh`)**

The updated script will handle the creation of the token, sending the email, and waiting for verification. It also includes logic for token expiration and ensuring proper permissions for the token files.

```bash
#!/bin/bash

# Set the user's email address
USER_EMAIL="user@example.com"  # Replace with the actual user's email address
EXPIRATION_TIME=900  # Token expiration time in seconds (15 minutes)
TOKEN_FILE="/tmp/verification_token.txt"
VERIFIED_FILE="/tmp/verification_complete.txt"
CURRENT_TIME=$(date +%s)

# Generate a unique token
VERIFICATION_TOKEN=$(openssl rand -base64 32)

# Generate a verification URL with the token
VERIFICATION_URL="https://yourdomain.com/verify.php?token=$VERIFICATION_TOKEN"

# Store the token with expiration timestamp
echo "$VERIFICATION_TOKEN $CURRENT_TIME" > $TOKEN_FILE

# Ensure the token file has proper read/write permissions
chmod 600 $TOKEN_FILE

# Send the email with the verification link
echo -e "Subject: SSH Login Verification\n\nClick the following link to complete your SSH login: $VERIFICATION_URL" | ssmtp $USER_EMAIL

# Notify user to check email
echo "A verification link has been sent to $USER_EMAIL. Please check your inbox and click the link to complete the login process."

# Wait for the user to click the link and verify
while [ ! -f $VERIFIED_FILE ]; do
    # Check for expiration
    TOKEN_TIMESTAMP=$(awk '{print $2}' $TOKEN_FILE)
    if [ $((CURRENT_TIME - TOKEN_TIMESTAMP)) -ge $EXPIRATION_TIME ]; then
        echo "Verification token expired. Please request a new login link."
        exit 1
    fi
    sleep 5
done

# If verification is successful, grant shell access
echo "Verification successful. Granting access to $USER_EMAIL."
exec /bin/bash
```

Ensure the script is executable:

```bash
chmod +x /path/to/your/verify_login.sh
```

---

### 3. **Configure `ssmtp.conf` for Email Sending**

Ensure `ssmtp` is installed and configured to send emails.

Example `/etc/ssmtp/ssmtp.conf`:

```bash
# /etc/ssmtp/ssmtp.conf
root=postmaster
mailhub=smtp.yourmailserver.com:587  # Replace with your SMTP server
AuthUser=your_email@example.com      # Your email address
AuthPass=your_email_password         # Your email password
FromLineOverride=YES
UseSTARTTLS=YES
```

Install `ssmtp` if not already installed:

```bash
sudo apt-get install ssmtp
```

---

### 4. **Create the Web Verification Endpoint**

You need a web server to handle the token verification. This can be done with a simple PHP script that verifies the token and marks it as verified.

**PHP Script (`verify.php`)**:

```php
<?php
$token = $_GET['token'];  // Get token from URL

$token_file = '/tmp/verification_token.txt';
$verified_file = '/tmp/verification_complete.txt';

// Read the stored token and timestamp
$stored_token = file_get_contents($token_file);
list($stored_token_value, $timestamp) = explode(" ", $stored_token);

$current_time = time();
$token_expiry = 900;  // Token expiry time in seconds (15 minutes)

if ($token == $stored_token_value) {
    // Check if the token is expired
    if (($current_time - $timestamp) > $token_expiry) {
        echo "The verification link has expired. Please request a new link.";
    } else {
        // Mark the verification as complete
        file_put_contents($verified_file, "verified");

        echo "Verification successful! You can now return to your SSH session.";
    }
} else {
    echo "Invalid verification link.";
}
?>
```

Ensure that the PHP script is served via HTTPS and that the web server has access to the `/tmp/verification_token.txt` file.

---

### 5. **Permissions and Expiry Logic**

- **Permissions**: The `/tmp/verification_token.txt` file should have proper read/write permissions for the verification script to write the token and read it during the verification process.
  
- **Expiration Logic**: The script stores both the token and the timestamp when it was generated. If more than 15 minutes pass before the user verifies, the token expires, and they are prompted to request a new one.

- **Token Verification**: Once the user clicks the link, the PHP script checks if the token matches the one stored on the server and whether it’s expired. If the token is valid, it creates a `verification_complete.txt` file that lets the SSH script know the user is authorized.

---

### 6. **Web Server Setup for HTTPS (Optional)**

Ensure your verification endpoint is available over HTTPS. Use Let's Encrypt or another SSL provider to secure your server. Example with `nginx`:

```nginx
server {
    listen 443 ssl;
    server_name yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    root /var/www/html;
    index index.php;

    location /verify.php {
        fastcgi_pass 127.0.0.1:9000;  # PHP-FPM setup
        fastcgi_param SCRIPT_FILENAME /var/www/html/verify.php;
        include fastcgi_params;
    }
}
```

Ensure the `nginx` or Apache web server is properly configured to serve your `verify.php` file securely.

---

### 7. **Testing**

1. After setting up everything, test the flow:
    - Attempt an SSH login.
    - Receive the verification email with the link.
    - Click the link to verify and return to the SSH session.
    - After successful verification, the user is granted access to the shell.

2. If the verification link expires (after 15 minutes), the user will need to request a new verification link.

---

### 8. **Final Notes**
- **Security**: Always secure the web server with HTTPS, especially when handling sensitive login tokens.
- **Session Management**: The verification token expiration should be enforced on both the web server and SSH verification scripts.
- **Email Provider**: Ensure your email provider allows sending emails via `ssmtp`. You may want to use services like SendGrid, Amazon SES, or Gmail with OAuth for better security.

This setup allows for a secure, token-based SSH login verification process that ensures the user’s identity via an email link with token expiration and proper file permissions.

😃🥲😗😙🥲😙🥹😄🥹😄🥹😁🥹😁😄🥹😄😄🥹😁🥹😁🥹😚🥹😚🥹😄🥹😄🥹😄🥹😄🥹😄🥹😁🥹😁🥹😁🥹😁🥹😄🥹😄🥹😄🥹😁🥹😁🥹😁🥹😁🥹😁🥹🥹😁😁🥹😁🥹😁🥹😁🥹😁🥹😄🥹😄😄🥹😁🥹😁🥹😁🥹😁🥹😁🥹😁🥹😁🥹😁🥹😄🥹😄🥹😄🥹😄🥹😄😄🥹😄🥹😄🥹😄🥹😄🥹😄🥹😄🥹😄🥹😄🥹😄🥹😄🥹😄🥹😄🥹😄🥹🥹😄😄🥹🥹😄🥹😄😄🥹🥹😄😁🥹😁🥹😄🥹😁🥹😁
To configure SSH login via email link click verification (two-factor authentication via a link sent to an email), incorporating `$SHELL` access, `ssmtp.conf`, and `.sh` files (for scripts), and forcing the command with `ForceCommand` and browser, we need to address multiple aspects.

The setup will involve:

1. **SSH Configuration** to force specific commands.
2. **Email Sending Mechanism** via `ssmtp`.
3. **Shell Script** that handles the email verification and forces specific commands.

I'll break it down step-by-step.

### 1. **SSH Server Configuration (`/etc/ssh/sshd_config`)**

We need to configure SSH to force the execution of specific commands and ensure the correct authentication. Add the following to your `sshd_config` file to enforce the command (e.g., the script to check the verification):

```bash
# Enforce the use of a specific script with ForceCommand
ForceCommand /path/to/verify_email.sh
AllowTcpForwarding no
PermitTTY yes
```

This ensures that every SSH login will trigger the execution of the script `verify_email.sh`.

### 2. **The Email Verification Script (`verify_email.sh`)**

This script will handle the email verification part. It will send an email with a unique verification link to the user, which they must click to authenticate. The script will check if the user has clicked the link and proceed accordingly.

#### `verify_email.sh`:

```bash
    #!/bin/bash

# Define a unique token for verification
TOKEN=$(openssl rand -hex 16)
USER_EMAIL="$1"  # Assumes email is passed to the script via SSH command line

# Set token expiration time (in seconds, e.g., 10 minutes = 600 seconds)
EXPIRATION_TIME=600
START_TIME=$(date +%s)

# Create a temporary file to track the token
TEMP_DIR="/tmp/verification"
mkdir -p $TEMP_DIR
echo $TOKEN > "$TEMP_DIR/$USER_EMAIL.token"

# Send email with the verification link
VERIFICATION_URL="https://yourserver.com/verify?token=$TOKEN"
echo -e "Subject: SSH Login Verification\n\nClick the link to verify your SSH login: $VERIFICATION_URL" | ssmtp $USER_EMAIL

# Wait for user to verify (by clicking the link, ideally with a server-side handler)
echo "Waiting for verification..."
while true; do
    # Check if token has expired
    CURRENT_TIME=$(date +%s)
    ELAPSED_TIME=$((CURRENT_TIME - START_TIME))
    
    if [ -f "$TEMP_DIR/$USER_EMAIL.verified" ]; then
        echo "Email verified, continuing SSH session."
        break
    elif [ $ELAPSED_TIME -gt $EXPIRATION_TIME ]; then
        echo "Token expired. Verification failed."
        rm "$TEMP_DIR/$USER_EMAIL.token"  # Clean up token
        exit 1
    fi

    # Sleep for a short time before checking again
    sleep 5
done

# Provide the user with shell access
exec $SHELL


```

This script generates a verification token and sends it via email using `ssmtp`. The user will have to click the verification link. The script checks if a file named `.verified` exists to confirm the user has verified via the link.

### 3. **`ssmtp.conf` Configuration**

The `ssmtp` utility will send the email. You'll need to configure it with your mail provider.

#### `/etc/ssmtp/ssmtp.conf`:

```bash
root=postmaster
mailhub=smtp.yourmailprovider.com:587
AuthUser=your-email@example.com
AuthPass=your-email-password
UseTLS=YES
UseSTARTTLS=YES
FromLineOverride=YES
```

This configuration ensures that `ssmtp` uses your email credentials to send the verification email.

### 4. **Web Verification Handler**

You also need a web server to handle the verification link (e.g., a simple web server or PHP script) that will create the `.verified` file.

#### Example: PHP Verification Handler (`verify.php`)

This is a very simple PHP script that listens for the token and creates the verification file:

```php
<?php
if (isset($_GET['token']) && !empty($_GET['token'])) {
    $token = $_GET['token'];
    
    // Ensure you validate the token properly (to avoid attacks)
    $email = "user@example.com";  // Retrieve the correct email based on token (you'll need a mapping)
    $file_path = "/tmp/verification/$email.token";
    
    if (file_exists($file_path) && file_get_contents($file_path) === $token) {
        touch("/tmp/verification/$email.verified");
        echo "Email verified successfully!";
    } else {
        echo "Invalid token.";
    }
} else {
    echo "No token provided.";
}
?>
```

This script checks the token from the URL, verifies it, and creates the `.verified` file for the user. Ensure that the email-to-token mapping is correctly implemented for security.

### 5. **Testing and Adjustments**

- **Test the complete flow**: Ensure the user can log in, receive an email, click the verification link, and then get access to the shell.
- **Security considerations**: You should secure the token validation process, ensuring it's unique per session and properly validated.
- **Permissions**: Ensure the SSH user has the proper permissions to execute scripts and access `$SHELL`.

### 6. **Browser Handling**

The browser will be used to click the link, so there’s no need for any specific browser-side handling beyond the simple PHP script.

---

### Notes:
- This setup assumes that you have a basic web server with PHP installed to handle the token verification process.
- You may want to integrate more robust email validation or token expiration checks to ensure the security of the verification process.

😗😙😙😚😆😆😘😘😘😘😘😄☺️😃😊😗😙☺️☺️😙☺️😗😊😃😃😊😙😊😃😊😙😊😙😄😊😄😊😙😊😙😊😙😊😙😄😊😙😊😙😊😙😊😚😊😙😙😊😙😊😙😊😙😊😚😊😁😊😄☺️😃☺️😚☺️😚☺️😚😄☺️😄😊😚😊😙😊😙😊😙😊😚😙😊😙😊😚😊😚😊😚☺️😚☺️😚☺️😚☺️😚😚☺️😚☺️😘☺️😘☺️😆☺️😁☺️😄😄☺️😁☺️😚☺️😄☺️😄☺️😄😄☺️😄☺️😙☺️😚☺️😚☺️😚😚☺️😚☺️😚☺️☺️☺️
To set up SSH login on an Ubuntu server with email verification using a clickable link (facilitated with `ssmtp.conf`), and to include `$SHELL` and `ForceCommand` with a `.sh` script, you will need to combine several configurations and steps. Here's an overview of how to approach this task:

### **1. Install Required Packages**

You'll need to install the `ssmtp` package for email handling and any dependencies that might be required for your script:

```bash
sudo apt update
sudo apt install ssmtp mailutils
```

### **2. Configure `ssmtp`**

To send email via a third-party service like Gmail, you need to configure `ssmtp.conf` file:

```bash
sudo nano /etc/ssmtp/ssmtp.conf
```

Here's an example configuration for Gmail:

```ini
root=your-email@gmail.com
mailhub=smtp.gmail.com:587
hostname=your-server-name
FromLineOverride=YES
AuthUser=your-email@gmail.com
AuthPass=your-app-password
UseSTARTTLS=YES
```

Replace `your-email@gmail.com` and `your-app-password` with your Gmail address and generated app password. You can create an app password from your Google account security settings if 2FA is enabled.

### **3. Setup Email Link Verification**

To facilitate email verification with a clickable link, you can write a shell script that generates a unique token and sends the verification email to the user.

1. **Create a script** that will generate the verification link and email it.

```bash
nano /usr/local/bin/send_verification_email.sh
```

Script content (this example assumes an existing email service):

```bash
#!/bin/bash

# Generate a unique token
TOKEN=$(openssl rand -base64 32)

# Store the token (could be in a database or file system)
echo "$TOKEN" > /tmp/verification_token.txt

# Set the verification URL (You can change this to match your web app)
VERIFICATION_URL="https://yourserver.com/verify?token=$TOKEN"

# Send the email using ssmtp
TO="$1"
SUBJECT="SSH Login Verification"
BODY="Click the following link to verify your SSH login: $VERIFICATION_URL"

echo -e "Subject:$SUBJECT\n$BODY" | ssmtp "$TO"

echo "Verification email sent to $TO"
```

This script generates a unique token and sends it to the user via email with a clickable verification link. Replace `yourserver.com` with your actual domain.

2. **Make the script executable:**

```bash
chmod +x /usr/local/bin/send_verification_email.sh
```

### **4. SSH Login Flow**

You will need to create an SSH login process where the user enters an email address, receives a verification link, and then is granted access after clicking the link.

1. **Modify SSH Configuration (`/etc/ssh/sshd_config`)**

Use `ForceCommand` to restrict the user to running a specific command (e.g., a script that checks for the verification token).

```bash
sudo nano /etc/ssh/sshd_config
```

Add the following line at the end of the configuration:

```ini
ForceCommand /usr/local/bin/verify_login.sh
```

2. **Create the Verification Script**

Create the script `verify_login.sh` that checks if the user clicked the verification link:

```bash
sudo nano /usr/local/bin/verify_login.sh
```

Example content:

```bash
#!/bin/bash

# Path to the file storing the token
TOKEN_FILE="/tmp/verification_token.txt"

# Check if the token file exists and if it matches the expected value
if [[ -f "$TOKEN_FILE" ]]; then
  TOKEN=$(cat "$TOKEN_FILE")

  # You can implement a more complex token validation here
  if [[ "$TOKEN" == "expected-token" ]]; then
    # Allow the user to login
    echo "Login verified"
    exit 0
  else
    echo "Invalid token"
    exit 1
  fi
else
  echo "No token found"
  exit 1
fi
```

Make the script executable:

```bash
chmod +x /usr/local/bin/verify_login.sh
```

This script will check for a verification token stored in `/tmp/verification_token.txt` and allow the login if the correct token is present.

### **5. Update `.bashrc` or `.profile`**

To ensure the correct shell is used and the user experience is smooth after login, you can add this to the user's `.bashrc` or `.profile` file:

```bash
# Add this to the user's .bashrc or .profile
export SHELL=/bin/bash
```

This ensures that `$SHELL` is correctly set.

### **6. Test the Flow**

- When a user tries to SSH into the server, they will be forced to run the verification script (`verify_login.sh`).
- The verification script checks if the user has clicked the verification link.
- If the link is clicked and the token is verified, the user is granted access.

### **Notes and Improvements:**

1. **Security**: The token generation and verification process should be secure. For example, use HTTPS for the verification link, and ensure the token is stored securely.
2. **SMTP Configuration**: You may need to fine-tune `ssmtp.conf` depending on your email provider.
3. **Token Expiry**: You might want to implement a mechanism to expire tokens after a certain period or after they are used.

This approach is a basic setup, and additional layers of security (like multi-factor authentication) should be considered for production environments.
😃😊😙☺️😚☺️😚☺️😁☺️😚☺️😁☺️😄☺️😙😁☺️😁☺️😚☺️😚☺️😚☺️😁☺️😁☺️😄😌😄😌😄😌😚😌😚😌😘😌😚😌😚😌😚😌😚😌😚😌😚😚😌😚😌😚😌😚😌🥲😌😚😚😌😚😌😚😌😚😌😚😌😁😌😁😌😚😁😌😁😌😚😌😁😌😁😌😁😚😌😚😌😚😌😁😌😚😌😚🤩😚😌😚🤩😚🤩😘😚🤩😚🤩😚🤩😚🤩😚🤩😚🤩😚😘🤩😘🤩😘🤩😘🤩😆🤩😚🤩😚🤩😚🤩😁🤩😚😚🤩🤩😚🤩😚🤩😚😁🤩😁

To set up SSH login for an Ubuntu server via an email link verification process, using `ssmtp.conf` and `$SHELL`, and employing `ForceCommand` with a `.sh` script, you would need a few pieces of the puzzle in place.

### Steps Overview

1. **Configure Email for Sending Verification Link**
   - Set up `ssmtp` for sending email notifications with a verification link.
   
2. **Create the SSH Public Key Authentication and ForceCommand**
   - Use `ForceCommand` in the SSH configuration to run a script that handles verification.

3. **Set Up the Script to Handle Link Verification**
   - The script should authenticate the user based on the link received.

4. **Configure SSH for Enforced Command**
   - SSH should be configured to always invoke the script when the user logs in, forcing them to verify via the email link.

### Step 1: Install and Configure `ssmtp`
`ssmtp` is a simple SMTP client used to send emails from your server. You can install and configure it to send verification emails.

1. Install `ssmtp`:
   ```bash
   sudo apt update
   sudo apt install ssmtp
   ```

2. Configure `ssmtp` in `/etc/ssmtp/ssmtp.conf` to use your email provider (for example, Gmail):

   ```ini
   root=your-email@gmail.com
   mailhub=smtp.gmail.com:587
   UseSTARTTLS=YES
   AuthUser=your-email@gmail.com
   AuthPass=your-email-password
   FromLineOverride=YES
   ```

3. Test your configuration by sending a test email:

   ```bash
   echo "Subject: Test Email" | ssmtp recipient@example.com
   ```

### Step 2: Configure SSH to Use ForceCommand
In `/etc/ssh/sshd_config`, add a `ForceCommand` directive to force the execution of a verification script upon SSH login.

1. Open the SSH configuration file:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Add or modify the following lines to invoke the verification script:

   ```bash
   Match User your-ssh-username
   ForceCommand /path/to/verify_login.sh
   AllowTcpForwarding no
   ```

3. Restart the SSH service to apply changes:

   ```bash
   sudo systemctl restart ssh
   ```

### Step 3: Write the Verification Script
Create a `.sh` script that will handle the email link verification. This script will check if the user has clicked the verification link (e.g., stored in a file or database).

1. Create the verification script `/path/to/verify_login.sh`:

   ```bash
   sudo nano /path/to/verify_login.sh
   ```

2. Script example:

   ```bash
   #!/bin/bash

   # Check if verification file exists (this file is set after email verification)
   if [ -f /home/your-ssh-username/.verified ]; then
       # Proceed to the user's shell if verified
       exec $SHELL
   else
       # Send verification email with the link
       VERIFICATION_LINK="http://yourserver.com/verify?user=$USER&key=$RANDOM"
       echo -e "Subject: SSH Login Verification\n\nClick the following link to verify your login: $VERIFICATION_LINK" | ssmtp your-email@example.com

       echo "Verification email sent. Please verify via the email link."
       exit 1
   fi
   ```

3. Ensure that the script is executable:

   ```bash
   sudo chmod +x /path/to/verify_login.sh
   ```

### Step 4: Email Link Handling (Web Server Component)
To complete the verification process, you need a web server to handle the click on the verification link. The server should mark the user as verified when the link is clicked.

1. Set up a web server (e.g., Apache or Nginx) and write a script to handle the verification request.

2. Example PHP script (`/var/www/html/verify.php`):

   ```php
   <?php
   $user = $_GET['user'];
   $key = $_GET['key'];

   // Check the key and user against your database or logic
   if ($user && $key) {
       file_put_contents("/home/$user/.verified", "verified");
       echo "Verification successful. You may now log in via SSH.";
   } else {
       echo "Invalid verification link.";
   }
   ?>
   ```

3. Ensure the web server is running and accessible.

### Step 5: Final Considerations

- **Security**: Make sure the verification link is secure and not easily guessable. The above example uses `$RANDOM`, but you might want a more robust mechanism (e.g., JWT, tokens, or a better random generator).
- **$SHELL**: The `$SHELL` variable in the script will execute the user's default shell once they are verified.
- **Persistent Verification**: The script uses a file (`~/.verified`) to track whether the user has clicked the verification link. You could also use a database for more persistent tracking.

### Step 6: Test
1. SSH into the server as the user.
2. The script will send a verification email.
3. Click the link in the email.
4. If the link is valid, you’ll be granted access to the shell.

This setup involves combining SSH configuration, email handling with `ssmtp`, a web server for verification, and a shell script for login enforcement.


😃🥰😙😊😚☺️😁☺️😁☺️😆☺️😆☺️😆☺️😘☺️😘😌😘☺️😁😘☺️😘☺️😚☺️😚☺️😚☺️😚☺️😚☺️😁☺️😚☺️😁😁☺️😁😁☺️😁☺️😁☺️😁☺️😘☺️😘☺️😘☺️😘☺️😘☺️😁😁😍😁😍😁😍😁😍😆😍😆😍😆😍😘😍😘😍😘😚😚☺️😁☺️😁☺️😁☺️😁☺️😁☺️😁😁☺️😁☺️😁☺️😁☺️😁☺️😁☺️😆☺️😆😍😘😍😘😍😘😍😘😍😘😍😘😍😘😍😘😍😆😍😆😍😆😍😆😆😍😆😍😘😍😘😍😘😍😘
### Complete Step-by-Step Guide for SSH Login via Email Link Verification on Ubuntu Server

This guide provides a step-by-step approach to setting up SSH login on an Ubuntu server that uses an email-based verification link. We'll walk through the configuration of `ssmtp`, SSH, a verification script, and optional web interface for the verification process. We will also ensure that the security aspects of the system are handled appropriately.

---

### **1. Install and Configure `ssmtp` for Sending Emails**

We will use `ssmtp` (Simple SMTP) for sending email notifications and verification links to users when they attempt to log in via SSH.

#### 1.1 Install `ssmtp`:
Run the following command to install `ssmtp`:

```bash
sudo apt-get update
sudo apt-get install ssmtp
```

#### 1.2 Configure `ssmtp` to Use Your Email Provider's SMTP Server
You need to configure `ssmtp` to send emails via an SMTP server. Here’s how to do it:

- Open the `ssmtp.conf` file:

```bash
sudo nano /etc/ssmtp/ssmtp.conf
```

- Configure it for your email provider. For Gmail, the configuration would look like:

```bash
root=your-email@gmail.com
mailhub=smtp.gmail.com:587
AuthUser=your-email@gmail.com
AuthPass=your-app-password
UseTLS=YES
UseSTARTTLS=YES
FromLineOverride=YES
```

**Important:** If you use Gmail, you should set up an [App Password](https://support.google.com/accounts/answer/185833) instead of using your main Gmail password.

#### 1.3 Test Email Sending

You can test whether `ssmtp` is working by sending a test email. For example:

```bash
echo "Test Email Body" | ssmtp recipient@example.com
```

Make sure the email is received correctly.

---

### **2. Configure SSH to Run a Custom Script for Verification**

In this step, we'll set up the SSH configuration to trigger a script every time a user logs in. The script will send an email containing a verification link, and the user will need to click the link to complete the login process.

#### 2.1 Edit SSH Configuration (`sshd_config`)

We will use the `ForceCommand` directive to run a custom verification script each time a user attempts to SSH into the server. Open the SSH server configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```

Look for the `ForceCommand` directive. If it doesn’t exist, add it at the bottom of the file:

```bash
ForceCommand /path/to/verify-login.sh
```

Make sure the script is accessible by the SSH daemon, so you’ll want to set appropriate permissions for `/path/to/verify-login.sh`.

#### 2.2 Restart SSH Service

After modifying the `sshd_config` file, restart the SSH service for the changes to take effect:

```bash
sudo systemctl restart sshd
```

---

### **3. Create the Verification Script**

This script will be responsible for sending the verification email to the user and blocking the login until the user clicks the verification link.

#### 3.1 Create the Script (`verify-login.sh`)

Create the verification script:

```bash
sudo nano /path/to/verify-login.sh
```

Here’s an example script that will generate a verification link and send it via email:

```bash
   
    #!/bin/bash

# Get the username of the person attempting to log in
USERNAME=$(whoami)

# Get the user's email address (from the system password entry)
EMAIL=$(getent passwd $USERNAME | cut -d: -f5)

# Generate a random verification code (UUID)
VERIFICATION_CODE=$(uuidgen)

# Define a secure temporary file for the verification code
VERIFICATION_FILE="/tmp/verification_code_${USERNAME}_$VERIFICATION_CODE"

# Save the verification code in the temporary file
echo $VERIFICATION_CODE > $VERIFICATION_FILE

# Create a URL that the user will click to verify their login
VERIFICATION_LINK="http://your-server.com/verify.php?code=$VERIFICATION_CODE"

# Send the verification email with the link
echo -e "Subject: SSH Login Verification\n\nHello $USERNAME,\n\nPlease click the following link to verify your SSH login: $VERIFICATION_LINK" | ssmtp $EMAIL

# Notify the user that the verification email has been sent
echo "A verification link has been sent to your email address. Please click the link to complete the login."

# End the session until the user clicks the link
exit 1


```


- Replace `/path/to/verify-login.sh` with the actual path to the script.
- Ensure that the script is executable:

```bash
sudo chmod +x /path/to/verify-login.sh
```

This script will send the user a verification link with a randomly generated code. The login will be blocked (via `exit 1`) until the user clicks the link.

#### 3.2 Testing the Script

You can manually test this script to ensure it works by running it as a regular user:

```bash
/path/to/verify-login.sh
```

Make sure an email is sent and the verification link works.

---

### **4. Web Interface for Link Verification (Optional)**

We need a web server to handle the verification process. When the user clicks the link, the server will check if the code is valid.

#### 4.1 Set Up a Simple PHP or Python Server

You can set up a web server to handle the verification. Here’s an example PHP script that verifies the code.

##### Example PHP Verification Script (`verify.php`):

```php

     <?php
// Retrieve the verification code from the URL (GET parameter)
$code = isset($_GET['code']) ? basename($_GET['code']) : '';  // Ensure the code is a safe filename

// Ensure that the code parameter is provided
if (empty($code)) {
    echo "No verification code provided.";
    exit;
}

// Path to the verification code file (matching the naming convention used in the Bash script)
$verification_file = "/tmp/verification_code_" . $code;

// Check if the file exists and validate the code
if (file_exists($verification_file)) {
    // Validation success
    echo "SSH login verified successfully.";

    // Optionally, you could log the user in here or set a session
    // For now, we simply remove the verification file
    unlink($verification_file);
} else {
    // Invalid or expired verification code
    echo "Invalid or expired verification code.";
}
?>

    

```

#### 4.2 Web Server Configuration

You need a web server (e.g., Apache, Nginx) to serve this PHP file. For example, install Apache and PHP:

```bash
sudo apt-get install apache2 php libapache2-mod-php
```

- Place the `verify.php` file in the web root (e.g., `/var/www/html`).
- Make sure the web server is running and accessible at `http://your-server.com/verify`.

---

### **5. Completing the Login Process**

Once the user clicks the verification link:

1. The web server checks if the code is valid.
2. If the code is valid, the user is allowed to complete the login process.
3. You can either:
   - Allow the user to continue the session by removing the verification code file, or
   - Automatically trigger some action (e.g., creating a file that indicates successful verification).

The `verify-login.sh` script can then check for this verification status and proceed to grant the user access.

#### Example Check for Verification Status:

```bash
#!/bin/bash

# Check if the user has verified their login
if [ -f /tmp/verification_code_$USERNAME ]; then
    echo "Please verify your login by clicking the link sent to your email."
    exit 1  # Block the login
else
    # Verification successful, proceed with regular shell login
    exec $SHELL
fi
```

---

### **6. Testing the Entire Flow**

- Test the SSH login. When a user logs in, the verification email should be sent.
- Verify that the user cannot proceed until they click the link.
- Ensure that the web verification process works correctly.
- After the user clicks the link, they should be able to access the shell.

---

### **Important Considerations**

1. **Security:** 
   - Secure the verification link by using HTTPS.
   - Consider encrypting or hashing the verification code to prevent brute force or code interception.

2. **User Experience:** 
   - This system adds extra steps for users, so assess whether this is acceptable for your use case.

3. **Server Load:**
   - Ensure you rate-limit email requests to avoid abuse.
   - Consider adding expiry dates for verification links to enhance security.

4. **Testing and Logging:** 
   - Test the entire process thoroughly, including failure scenarios.
   - Add logging to capture failed verification attempts.

By following these steps, you will have a working SSH login system with email verification for Ubuntu.



☺️😗😙☺️😙😊😚😊😄😊😃😊😚😊😚😊😄😊😄😊😄😊😙😊😚😊😁😜😄😜😃😜😃😜😙☺️😙😙☺️😙☺️😙😜😙😜😙☺️😙☺️😚☺️😄☺️😄☺️😄☺️😁☺️😁☺️😁☺️😁☺️😁☺️😁☺️😆😆☺️😆☺️😁☺️😁☺️😁☺️😁☺️😚😜😚😜😚☺️😘☺️😘😜😚😜🥲😜🥲😜🥲😜😚😜😚😜😚😜😚😜😚😜😚😜😚😚😜😁☺️😁☺️😁☺️😄☺️😄☺️😄☺️😁☺️😁☺️😁☺️😁☺️😁☺️😁☺️😁☺️😁☺️😄☺️😁☺️😚☺️😚☺️😚☺️😚☺️😚☺️😚☺️😚☺️😚☺️😁☺️😚☺️😚☺️😁☺️😁☺️😁😁☺️😁☺️😁☺️😁☺️😁☺️😁☺️😁☺️😁☺️😁☺️😁☺️😁☺️😁☺️🤪😁

Configuring SSH login via an email link with `ssmtp` and `$SHELL` access, while using `ForceCommand` without the traditional token or key-based authentication, is a non-standard approach that requires several creative workarounds. This setup will likely require combining multiple methods, including email-based verification, server-side scripting, and SSH configuration adjustments. Here's an outline of the steps to configure this:

### **Overview:**
1. **Email Link Authentication:** Create a system that sends an email with a unique verification link.
2. **Scripting and `$SHELL` Access:** Use the link to trigger a server-side script that generates a temporary session and allows SSH access.
3. **ForceCommand:** Use `ForceCommand` to enforce a specific script on login that verifies the link/token.

### **Prerequisites:**
1. **ssmtp** installed and configured to send emails.
2. An SMTP server (e.g., Gmail, SendGrid, etc.) to send emails.
3. The server must have a web server (e.g., Apache or Nginx) to handle the link-click verification.

### **Step-by-Step Configuration**

#### 1. **Install and Configure ssmtp**
First, set up `ssmtp` to send emails. You can use a service like Gmail or any other SMTP server.

```bash
sudo apt-get update
sudo apt-get install ssmtp
```

Configure `ssmtp` by editing `/etc/ssmtp/ssmtp.conf`:

```ini
root=postmaster
mailhub=smtp.gmail.com:587
AuthUser=your-email@gmail.com
AuthPass=your-email-password
UseTLS=YES
UseSTARTTLS=YES
FromLineOverride=YES
```

Replace `your-email@gmail.com` and `your-email-password` with your actual credentials.

#### 2. **Create the Email Verification System**
You'll need a web server to handle the unique link. Let’s create a simple PHP script to handle the link-click verification.

- Install Apache and PHP:

```bash
sudo apt-get install apache2 php libapache2-mod-php
```

- Create a simple script to generate and verify tokens.

**Step 1:** Create a `tokens.txt` file to store the tokens:

```bash
touch /var/www/html/tokens.txt
chmod 600 /var/www/html/tokens.txt
```

**Step 2:** Create a PHP script to handle the link click and token validation. Create a file `/var/www/html/verify.php`:

```php
<?php
// Get the token from the URL
$token = $_GET['token'];
$tokensFile = '/var/www/html/tokens.txt';

if(file_exists($tokensFile)) {
    $tokens = file($tokensFile, FILE_IGNORE_NEW_LINES);

    if(in_array($token, $tokens)) {
        echo "Token verified! You can now login to the server via SSH.";
        
        // Remove the token after successful use
        $tokens = array_diff($tokens, array($token));
        file_put_contents($tokensFile, implode(PHP_EOL, $tokens));

        // Send SSH access email notification
        mail("your-email@gmail.com", "SSH Access Granted", "The token $token was successfully verified.");
    } else {
        echo "Invalid token.";
    }
} else {
    echo "No tokens available.";
}
?>
```

**Step 3:** Create a script to generate tokens and send emails.

Create a script `/usr/local/bin/send_verification_email.sh`:

```bash
#!/bin/bash

# Generate a unique token
TOKEN=$(openssl rand -hex 16)
TOKEN_URL="http://your-server-ip/verify.php?token=$TOKEN"

# Store the token in a file for later verification
echo "$TOKEN" >> /var/www/html/tokens.txt

# Send the email with the token link
echo "Click the following link to verify your SSH login: $TOKEN_URL" | ssmtp recipient-email@example.com

echo "Verification email sent with token: $TOKEN"
```

Make sure the script is executable:

```bash
chmod +x /usr/local/bin/send_verification_email.sh
```

#### 3. **Modify SSH Configuration:**
To ensure that a user cannot log in without a valid token, you can set `ForceCommand` in the SSH configuration file. The `ForceCommand` directive can run a custom script instead of allowing the usual shell or login process.

Edit `/etc/ssh/sshd_config` and add the following:

```bash
ForceCommand /usr/local/bin/verify_login.sh
```

This will ensure that when a user connects via SSH, they will be forced to run `/usr/local/bin/verify_login.sh` instead of getting a regular shell.

#### 4. **Create the SSH Verification Script:**

Now, create the script `/usr/local/bin/verify_login.sh`, which will check whether a user has a valid token before allowing them to use the shell.

```bash
#!/bin/bash

# Prompt user for token input
echo "Please enter the token you received via email:"
read USER_TOKEN

# Check if the token is valid
TOKEN_FILE="/var/www/html/tokens.txt"

if grep -q "$USER_TOKEN" "$TOKEN_FILE"; then
    # Valid token, proceed with shell access
    echo "Token verified! Access granted."
    # Set the user's shell to bash
    exec /bin/bash
else
    # Invalid token
    echo "Invalid token. Access denied."
    exit 1
fi
```

Make the script executable:

```bash
chmod +x /usr/local/bin/verify_login.sh
```

#### 5. **Restart SSH and Apache Services:**

After configuring everything, restart the SSH and Apache services:

```bash
sudo systemctl restart sshd
sudo systemctl restart apache2
```

#### 6. **Using the System:**
- When a user wants to log in via SSH, they will run `ssh user@your-server-ip`.
- The server will then ask them for the verification token.
- The user will need to click the email verification link sent to their email, which will provide them with a token.
- Once the token is entered, the SSH access will be granted if it matches a valid token.

### **Security Considerations:**
1. **Token Expiry:** Tokens should have an expiration time. You can implement this in your PHP verification script.
2. **Token Storage:** Tokens are stored in plain text (`tokens.txt`). Consider using encryption for higher security.
3. **Email Access:** Email accounts should be secured with two-factor authentication to prevent unauthorized access.

### Conclusion:
This method is highly customized and involves combining several components, including PHP for token management, `ssmtp` for email, and SSH configurations. However, keep in mind that this isn't the most secure or efficient way to handle SSH access compared to traditional key-based or password authentication.


😗😊😃😊😄😊😄😄😊😄☺️☺️😙☺️☺️😄☺️😄😄☺️😄☺️😄😍😄😍😁😍😄😍😄😄😂😁😂😚😍😚☺️🥲☺️🥲😌🥹😌😘😘😌🥰😌🥰😌🥰😌😘😌😘😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😘😌😘😌😘😌😘😌😘😌😘😌😘😌🥰😌🥰😌🥰😌🥰😌🥰😌😘😌😘😌😘😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😌😘😌🥰😌🥰🙂‍↕️🥰🙂‍↕️🥰🙂‍↕️🥰🙂‍↕️🥰🥰🙂‍↕️🥰🙂‍↕️🥰😌🥰😌🥰😌😘😌😘😌😘😌😘😌😘😘😌😘😌😘

To configure SSH login on an Ubuntu server using an email link click authentication method combined with a fixed token authentication file, and enforce the use of `$SHELL` with `ForceCommand`, you'll need to integrate several components, including email handling, token-based authentication, and SSH server configuration.

However, it's important to note that SSH generally doesn't natively support "email link click" authentication, but we can simulate this flow using a two-step process involving token-based access. Here’s how you can configure it:

### Steps:

#### 1. **Install and Configure `ssmtp` (or `msmtp`) for Email Sending**
   First, install `ssmtp` or `msmtp` to send an email with a token link.

   ```bash
   sudo apt update
   sudo apt install ssmtp
   ```

   **Configure `ssmtp`:**  
   Edit `/etc/ssmtp/ssmtp.conf` to set up an SMTP server for sending emails.

   Example `ssmtp.conf` (adjust according to your mail provider):

   ```ini
   root=postmaster
   mailhub=smtp.gmail.com:587
   AuthUser=your-email@gmail.com
   AuthPass=your-email-password
   UseTLS=YES
   UseSTARTTLS=YES
   FromLineOverride=YES
   ```

   If using Gmail or a similar service, you might need to allow "less secure apps" or use an App Password for authentication.

#### 2. **Create a Token-based Authentication System**
   To simulate "email link click" authentication, you’ll need to generate a temporary token and email it to the user. When the user clicks the link, the token should be verified against a server-side file (e.g., `/etc/ssh-token`).

   Example:

   1. **Generate a unique token (e.g., UUID)** for the user.

      ```bash
      TOKEN=$(uuidgen)
      echo $TOKEN > /etc/ssh-token
      ```

   2. **Send the token via email:**

      Create a script to send an email to the user with a link that includes the token.

      Example script (`send-token.sh`):

      ```bash
      #!/bin/bash

      TO=$1
      TOKEN=$(cat /etc/ssh-token)
      URL="https://your-server.com/validate_token?token=$TOKEN"

      SUBJECT="SSH Login Token"
      BODY="Click the link to authenticate: $URL"

      echo -e "Subject:$SUBJECT\n\n$BODY" | ssmtp $TO
      ```

   To send the token, run:

   ```bash
   ./send-token.sh user@example.com
   ```

#### 3. **Create Token Validation Mechanism**
   The user clicks the link, which should hit a server-side script that verifies the token and updates an `authorized_tokens` file.

   Example in `validate_token.sh`:

   ```bash
   #!/bin/bash

   RECEIVED_TOKEN=$1
   VALID_TOKEN=$(cat /etc/ssh-token)

   if [ "$RECEIVED_TOKEN" == "$VALID_TOKEN" ]; then
       echo "Token is valid. Access granted."
       echo "user" >> /etc/ssh-authorized-users
   else
       echo "Invalid token!"
   fi
   ```

   **Note:** This is a simplified example; in practice, you should implement proper web server logic, SSL encryption, and secure token storage.

#### 4. **Configure SSH Server to Force Command and Token Authentication**
   Now, configure the SSH server to enforce the token-based authentication and use a forced command for shell access.

   Edit `/etc/ssh/sshd_config`:

   ```bash
   # Disable normal password authentication
   PasswordAuthentication no

   # Only allow users who have an "authorized token"
   Match User *
       ForceCommand /usr/local/bin/validate-ssh-token.sh
   ```

   Create a script `/usr/local/bin/validate-ssh-token.sh` that checks if the user is authorized based on the token stored in `/etc/ssh-authorized-users`:

   ```bash
   #!/bin/bash
   if grep -q "$USER" /etc/ssh-authorized-users; then
       exec $SHELL
   else
       echo "Access denied"
       exit 1
   fi
   ```

#### 5. **Test the Authentication**
   1. Generate a token and send the link.
   2. The user clicks the link, which verifies the token.
   3. If valid, the user's SSH access is allowed.

#### Additional Considerations:
- **Security**: Email-based authentication has inherent security risks (e.g., interception of email). Consider using more secure methods like multi-factor authentication (MFA) or hardware keys.
- **Expiry Time**: Tokens should have an expiration time and be deleted once used, to prevent reuse.
- **Web Server**: You would need a web server (like Apache or Nginx) to handle the token validation process properly.

By following these steps, you'll implement a form of "email link click" authentication for SSH access on Ubuntu, using token-based authentication and a forced command.


😗😊😙😊😚☺️😁☺️😁☺️😘😘😳😂🥹🥱🤩🥹😶😄🥱😃🥱😗🥹😤😊😤🥹😗😤😙😤🥹😗😤😊😙😤😊😙😤😊😙😠😙🙁😊😙😜😚😯😟💩😵‍💫😟😵‍💫😟😵‍💫😧🙂🙁😶🙂🙁😶🙂☹️🙁😶🙃🫨😤🥺🤕🥱😬😷🙁🫢🤡😷😠🫢🤕🤕😠😒🤕🤕😠🤫😷🙁🤫💤💤❣️❣️🙀🌞🙀🌞❣️🙀🌞👣❣️💦🌝👣💦🌝❣️😿🌞❣️🌞💤❣️💤🌞👣
To implement this setup using `.sh` files, a clickable link for verification, and a post-login token validation, here’s an approach that integrates everything into a streamlined process:

### Step-by-Step Setup

### 1. **Install and Configure `ssmtp`**

Install and configure `ssmtp` to enable email sending on the server, following the steps provided previously. This setup will allow us to send an email with a clickable verification link containing the token.

### 2. **Generate Token and Send Email with Verification Link**

The `send_token.sh` script will generate a token, store it, and send an email with a clickable verification link. Place this file in `/var/www/html/` for easy access.

#### Example Script (`/var/www/html/send_token.sh`):

```bash
#!/bin/bash

# Generate a unique token and store it in /tmp
TOKEN=$(uuidgen)
echo "$TOKEN" > /tmp/token.txt

# Define the verification link
VERIFICATION_URL="http://your-server-ip/verify_token.sh?token=$TOKEN"

# Email subject and body with the clickable verification link
SUBJECT="Your Login Token"
BODY="Click the following link to verify your login: $VERIFICATION_URL"

# Send the email
echo -e "Subject: $SUBJECT\n\n$BODY" | ssmtp recipient@example.com
```

Make the script executable:

```bash
chmod +x /var/www/html/send_token.sh
```

This script will generate a token, store it in `/tmp/token.txt`, and send an email with a clickable link to the verification endpoint.

### 3. **SSH Configuration (Allow Login, Require Token Post-Login)**

Modify the SSH configuration to allow login but force users to run a verification script upon login.

#### SSH Configuration (`/etc/ssh/sshd_config`):

Edit the SSH configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```

Add or modify these lines:

```plaintext
# ForceCommand to run a token validation script after login
ForceCommand /usr/local/bin/validate_token.sh

# Allow only specific users to log in
AllowUsers youruser

# Optionally, restrict SSH access to certain IP addresses
# ListenAddress 0.0.0.0
```

This will make SSH run `validate_token.sh` for verification as soon as the user logs in.

### 4. **Create Token Validation Script (`validate_token.sh`)**

Now, create a script that will be triggered upon SSH login to check if the user has verified their token by clicking the link.

#### Example Script (`/usr/local/bin/validate_token.sh`):

```bash
#!/bin/bash

# Check if the token has been verified
if [[ -f /tmp/token_verified ]]; then
    # Token is verified, grant shell access
    echo "Token verified. Access granted."
    exec $SHELL
else
    # Token is not verified, deny access
    echo "Token not verified. Please click the verification link sent to your email."
    exit 1
fi
```

Make the script executable:

```bash
sudo chmod +x /usr/local/bin/validate_token.sh
```

This script checks for the presence of `/tmp/token_verified`, which will only exist if the user has clicked the verification link.

### 5. **Create Verification Script (`verify_token.sh`) for the Link**

To handle the verification link click, create a script accessible via the web server. This script will validate the token passed as a query parameter and create a marker file (`/tmp/token_verified`) upon successful verification.

#### Example Script (`/var/www/html/verify_token.sh`):

```bash
#!/bin/bash

# Extract the token from the query string
QUERY_STRING=$(echo "$1" | sed 's/token=//')
USER_TOKEN=$QUERY_STRING

# Check if the token matches
if [[ "$USER_TOKEN" == "$(cat /tmp/token.txt)" ]]; then
    # Create a marker file to indicate successful verification
    touch /tmp/token_verified
    echo "Token verified successfully. You can now log in via SSH."
else
    echo "Invalid token. Access denied."
fi
```

Ensure it’s executable:

```bash
chmod +x /var/www/html/verify_token.sh
```

This script checks the token passed in the URL. If it matches the stored token, it creates `/tmp/token_verified`.

### 6. **Testing the Process**

1. **Run `send_token.sh`**:
   Execute the token generation and email script:

   ```bash
   /var/www/html/send_token.sh
   ```

2. **Click the Verification Link**:
   Open your email and click the link. This should run `verify_token.sh`, setting the marker file if the token matches.

3. **SSH into the Server**:
   Log in via SSH:

   ```bash
   ssh youruser@your-server-ip
   ```

   If the token has been verified, the `validate_token.sh` script will detect `/tmp/token_verified` and grant access.

### 7. **Optional Enhancements**

- **Token Expiry**: You can add a check in `verify_token.sh` or `validate_token.sh` to enforce token expiration by comparing timestamps.
- **Secure Token Storage**: `/tmp` works for short-term storage, but consider a secure, persistent storage location if higher security is needed.
- **Remove Marker After Login**: Add logic to delete `/tmp/token_verified` after a successful login to avoid repeated access without re-verification.

### Final Notes

This process allows login without blocking SSH access but requires the user to click an emailed verification link to gain full access. This setup is flexible and integrates clickable link verification into the token-based system. Be sure to test thoroughly before using this in a production environment.

😊😗😗☺️😄☺️😚☺️😚☺️☺️😘😘😌🥰😌🥰😌🥰😌🥰😌🥰🥰😌😘😌😁😌😁😌😆😌😆😌😆😌😅🙂‍↕️😆🙂‍↕️🙂‍↕️😁🙂‍↕️😚🙂‍↕️😘🙂‍↕️😘🙂‍↕️😘🙂‍↕️😘🙂‍↕️🥰🙂‍↕️🥰🙂‍↕️🥰🙂‍↕️😊🙂‍↕️😘😅🙂‍↕️😅🙂‍↕️😁🙂‍↕️😁🙂‍↕️😚🙂‍↕️😚😌🥲😌😙😌😃😌😄😌😙😌🙂😙😌😌😙😙😌😄😌😄😌😄😌😄😌😄😌😙😌😙😙😌😙😌😚😌😚😌😚😌😚😌😁😁😌😁😌😚😌😘😌😘😌😘😌😘😌😚😌😚😚😌😚😌😌🙂‍↕️🙂‍↕️🙂‍↕️🙂‍↕️🙂‍↕️😌😌


Here is the revised and complete set of commands and configurations to set up token-based authentication for SSH access, including database storage, email sending, and token validation.

---

### **1. Set Up the Database (MySQL)**

To store tokens and manage their expiration and verification status, we'll use MySQL.

#### Install MySQL:

```bash
sudo apt-get install mysql-server
```

#### Create a Database and Table for Storing Tokens:

1. **Log into MySQL**:

   ```bash
   mysql -u root -p
   ```

2. **Create a Database and Table**:

   ```sql
   CREATE DATABASE ssh_tokens;
   USE ssh_tokens;

   CREATE TABLE tokens (
       id INT AUTO_INCREMENT PRIMARY KEY,
       token VARCHAR(64) NOT NULL,
       user_email VARCHAR(255) NOT NULL,
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
       expires_at TIMESTAMP NOT NULL,
       verified BOOLEAN DEFAULT FALSE
   );
   ```

This table will store each token, its associated email, expiration time, creation time, and whether it has been verified.

---

### **2. Configure `ssmtp` for Email Sending**

`ssmtp` will be used to send the token to the user.

#### Install `ssmtp`:

```bash
sudo apt-get install ssmtp
```

#### Edit `ssmtp.conf`:

Open `/etc/ssmtp/ssmtp.conf` and configure it to use your SMTP server details (replace with your actual email server configuration):

```ini
root=your-email@example.com
mailhub=smtp.your-email-provider.com:587
AuthUser=your-email@example.com
AuthPass=your-email-password
FromLineOverride=YES
UseSTARTTLS=YES
```

Replace `your-email@example.com` with your actual email address, and provide your SMTP server credentials.

---

### **3. Generate Token and Send Email**

Now, we need to modify the token generation script to store the token in the MySQL database and send the token via email.

#### Token Generation Script (`generate_send_token.sh`):

```bash
#!/bin/bash

USER_EMAIL="user@example.com"
TOKEN=$(openssl rand -hex 32)
EXPIRY_TIME=$(date -d "+10 minutes" +%Y-%m-%d\ %H:%M:%S)  # Token expires in 10 minutes

# Insert token into the MySQL database
mysql -u root -pYourPassword -D ssh_tokens -e "INSERT INTO tokens (token, user_email, expires_at) VALUES ('$TOKEN', '$USER_EMAIL', '$EXPIRY_TIME');"

# Send email with the token link
EMAIL_SUBJECT="SSH Authentication Token"
EMAIL_BODY="Click the link to authenticate: http://yourserver.com/verify?token=${TOKEN}"

echo -e "Subject:${EMAIL_SUBJECT}\n\n${EMAIL_BODY}" | ssmtp $USER_EMAIL
```

- The script generates a random token, inserts it into the MySQL database with the expiration time, and sends an email to the user with a link to verify the token.

Make sure to replace `YourPassword` in the `mysql` command with your actual MySQL root password.

---

### **4. Verify Token via Web Interface**

The user will click the link in the email, and the web server will handle token verification.

#### Web Token Verification Script (`verify_token.php`):

```php
<?php
if (isset($_GET['token'])) {
    $token = $_GET['token'];

    // Connect to the MySQL database
    $db = new mysqli('localhost', 'root', 'YourPassword', 'ssh_tokens');

    if ($db->connect_error) {
        die("Connection failed: " . $db->connect_error);
    }

    // Query to check if the token exists and is valid
    $result = $db->query("SELECT * FROM tokens WHERE token = '$token' AND expires_at > NOW() AND verified = 0");

    if ($result->num_rows > 0) {
        $row = $result->fetch_assoc();
        // Token is valid and not yet verified
        echo "Token valid! You can now authenticate with SSH.";

        // Mark token as verified
        $db->query("UPDATE tokens SET verified = 1 WHERE token = '$token'");

        // Optionally, you can redirect to an SSH login or perform other actions
    } else {
        echo "Invalid or expired token.";
    }

    $db->close();
} else {
    echo "No token provided.";
}
?>
```

This script verifies that the token is valid and not expired. If the token is valid, it marks the token as `verified` in the database.

---

### **5. Modify SSH Configuration and Forced Command**

To restrict SSH login and force users to authenticate with a token, we use SSH's `ForcedCommand` feature.

#### Modify SSH Configuration (`/etc/ssh/sshd_config`):

```bash
Match User yourusername
    ForcedCommand /path/to/token_validation_script.sh
```

Replace `yourusername` with the actual username you want to enforce token-based authentication for.

#### Reload SSH service:

```bash
sudo systemctl restart sshd
```

---

### **6. Token Validation Script**

Modify the token validation script to check if the token is valid and verified in the database before proceeding with the SSH session.

#### Token Validation Script (`token_validation_script.sh`):

```bash
#!/bin/bash

# Get the token from the user's environment or file (this depends on how you handle the token)
TOKEN=$(cat /home/yourusername/.tokens/token_file)

# Connect to the database and check token validity
RESULT=$(mysql -u root -pYourPassword -D ssh_tokens -e "SELECT * FROM tokens WHERE token = '$TOKEN' AND expires_at > NOW() AND verified = 1 LIMIT 1;")

if [ -z "$RESULT" ]; then
    echo "Invalid or expired token."
    exit 1
else
    # Allow the user to continue
    exec /bin/bash
fi
```

This script checks if the token exists, has not expired, and has been marked as `verified` in the database. If the token is valid, the script proceeds with the SSH session (`exec /bin/bash`). If invalid or expired, it terminates the session.

---

### **7. Testing the Setup**

#### Steps to Test the Configuration:

1. **Generate a Token**:
   Run the `generate_send_token.sh` script. The user will receive an email with a token link.
   
2. **Verify the Token**:
   When the user clicks the token link, the `verify_token.php` script is executed, and it marks the token as `verified` if it's valid.

3. **SSH Login**:
   After verifying the token, the user can attempt to SSH into the server. The `token_validation_script.sh` will check the token from the database and allow the user to log in if it's valid.

---

### **Conclusion**

This setup implements token-based authentication for SSH access. It uses MySQL to store token information and email-based validation. The SSH server is configured to restrict login to a validation script, and the script verifies the token before granting access. The use of `ssmtp` ensures the email is sent with the token, and `ForcedCommand` ensures users cannot perform other actions on login without successful token validation.

Make sure to test all parts of the setup thoroughly before deploying it to production.


😛😮‍💨😮‍💨😛🙃😢☹️🙂😶☹️😶😝🙂☹️😝😙☹️😝😙☹️😝🙂🙁😙😝🙁😙😝☹️😙😝☹️😝😗😤😝😗😝☹️😗😤😝😗😤😝😗☹️😝😙😤😝😗☹️😙😝☹️😝😙☹️😝😗☹️😝😙☹️😝😗☹️😝😗😗😝☹️☹️😛😗☹️😝😗😛😗☹️😗😝☹️😗😝☹️😗😛☹️😛😗☹️😗😝😗😝☹️☹️😛😗😙😝☹️😝😗☹️😝😗😝😗☹️😝😗☹️😗😝☹️😗☹️😝😗☹️😗😝😝😗☹️☹️😶😗😗😝☹️🙁😝😗☹️😶😗☹️😙😶😶😗🙁😶😗😯
To configure the above setup with a database for token storage instead of using a simple file in `/tmp`, we'll update the process to use a database (e.g., MySQL or PostgreSQL) for storing the token. This ensures that the token is persisted in a more robust and secure manner.

We'll use MySQL as an example, but this can easily be adapted for other databases like PostgreSQL.

### Step-by-Step Setup with Database Integration

### 1. **Install MySQL (if not already installed)**

Install MySQL or MariaDB on your server. Here's how to install MySQL on a Debian-based system:

```bash
sudo apt update
sudo apt install mysql-server
sudo mysql_secure_installation
```

After installation, secure the MySQL installation and make sure the service is running:

```bash
sudo systemctl start mysql
sudo systemctl enable mysql
```

### 2. **Create Database and Table for Token Storage**

Log into MySQL and create a database and table to store the tokens.

```bash
sudo mysql -u root -p
```

Once in the MySQL shell, run the following commands to create the database and table:

```sql
CREATE DATABASE token_db;

USE token_db;

CREATE TABLE tokens (
    id INT AUTO_INCREMENT PRIMARY KEY,
    token VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 3. **Install MySQL Client**

Install MySQL client tools if not already installed:

```bash
sudo apt install mysql-client
```

### 4. **Update `send_token.sh` to Store Token in Database**

Modify the `send_token.sh` script to insert the generated token into the MySQL database instead of storing it in a file. The script will also include the email address for the user to whom the token is being sent.

#### Updated `send_token.sh`:

```bash
#!/bin/bash

# Email recipient
EMAIL="recipient@example.com"

# Generate a unique token
TOKEN=$(uuidgen)

# Store the token in the MySQL database
mysql -u root -pYOUR_PASSWORD -e "USE token_db; INSERT INTO tokens (token, email) VALUES ('$TOKEN', '$EMAIL');"

# Define the verification URL
VERIFICATION_URL="http://your-server-ip/verify_token.sh?token=$TOKEN"

# Email subject and body with clickable link
SUBJECT="Your Login Token"
BODY="Click the following link to verify your login: $VERIFICATION_URL"

# Send the email using ssmtp
echo -e "Subject: $SUBJECT\n\n$BODY" | ssmtp $EMAIL
```

Make sure to replace `YOUR_PASSWORD` with the actual password for your MySQL root user. 

**Make the script executable:**

```bash
chmod +x /var/www/html/send_token.sh
```

This script will now generate a unique token, store it in the `tokens` table in the MySQL database, and send an email with the verification link.

### 5. **Update `verify_token.sh` to Verify Token Against Database**

Modify the `verify_token.sh` script to check the token in the database instead of checking a file in `/tmp`. When the token is valid, create a marker in the database indicating the token has been verified.

#### Updated `verify_token.sh`:

```bash
#!/bin/bash

# Extract the token from the query string
QUERY_STRING=$1
USER_TOKEN=$(echo "$QUERY_STRING" | sed 's/token=//')

# Check if the token exists in the database and if it has not expired
RESULT=$(mysql -u root -pYOUR_PASSWORD -s -N -e "USE token_db; SELECT token FROM tokens WHERE token = '$USER_TOKEN' LIMIT 1;")

if [[ "$RESULT" == "$USER_TOKEN" ]]; then
    # Token is valid, mark it as verified
    mysql -u root -pYOUR_PASSWORD -e "USE token_db; UPDATE tokens SET token_verified = 1 WHERE token = '$USER_TOKEN';"
    echo "Token verified successfully. You can now log in via SSH."
else
    echo "Invalid token. Access denied."
fi
```

Make sure to replace `YOUR_PASSWORD` with the MySQL root password.

**Make the script executable:**

```bash
chmod +x /var/www/html/verify_token.sh
```

This script now checks the database for the token, and if it is valid, updates the database to mark it as verified.

### 6. **Update `validate_token.sh` to Check Database for Verification**

Modify the `validate_token.sh` script to check the database for a verified token instead of checking for a marker file in `/tmp`.

#### Updated `validate_token.sh`:

```bash
#!/bin/bash

# Retrieve the user's email or username (this may vary depending on your system)
USER_EMAIL=$(whoami)

# Check if the token is verified by querying the database
RESULT=$(mysql -u root -pYOUR_PASSWORD -s -N -e "USE token_db; SELECT token_verified FROM tokens WHERE email = '$USER_EMAIL' AND token_verified = 1 LIMIT 1;")

if [[ "$RESULT" == "1" ]]; then
    # Token is verified, grant shell access
    echo "Token verified. Access granted."
    exec $SHELL
else
    # Token is not verified
    echo "Token not verified. Please click the verification link sent to your email."
    exit 1
fi
```

This script now queries the database to check if the token has been verified, using the email address of the user who is logging in.

### 7. **Testing the Process**

1. **Run `send_token.sh`**:
   Execute the token generation and email script:

   ```bash
   /var/www/html/send_token.sh
   ```

2. **Click the Verification Link**:
   Open your email and click the link. This should run `verify_token.sh`, which updates the database to mark the token as verified.

3. **SSH into the Server**:
   Log in via SSH:

   ```bash
   ssh youruser@your-server-ip
   ```

   The `validate_token.sh` script will check the database to see if the token has been verified. If verified, access will be granted.

### 8. **Optional Enhancements**

- **Token Expiry**: You can add a `expires_at` field in the `tokens` table and modify the `verify_token.sh` and `validate_token.sh` scripts to enforce token expiration.
- **Secure MySQL Connection**: Consider using a more secure method to authenticate MySQL, such as using a user with limited permissions and securing connections with SSL.
- **Audit Logging**: Log successful and failed token verifications to track login attempts.

### Final Notes

This setup integrates token generation, email-based verification, and SSH login validation with a MySQL database. The token is securely stored in the database, and its status is checked during the login process.

