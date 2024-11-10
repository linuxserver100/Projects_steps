Setting up MySQL, configuring it for remote connections, and connecting it with a PHP script involves several steps. Here is a complete guide to achieve this:

### 1. **Install MySQL**

#### On Ubuntu/Debian:

```bash
sudo apt update
sudo apt install mysql-server
```

#### On CentOS/RHEL:

```bash
sudo yum install mysql-server
```

#### On Windows:
1. Download MySQL Installer from the official MySQL website: https://dev.mysql.com/downloads/installer/
2. Run the installer and choose "Server only" or "Developer Default" for a simple installation.
3. Follow the instructions, configure root password, and note down your configuration details.

### 2. **Start MySQL Service**

#### On Ubuntu/Debian:

```bash
sudo systemctl start mysql
```

#### On CentOS/RHEL:

```bash
sudo systemctl start mysqld
```

#### On Windows:
- The MySQL service should be started automatically after installation. If not, you can start it manually from the Services application.

### 3. **Secure MySQL Installation**

After installation, run the MySQL secure installation script to set the root password, remove insecure default settings, and improve security.

```bash
sudo mysql_secure_installation
```

- Enter the root password.
- Follow the prompts to set a new root password and configure other security options (remove test databases, disable remote root login, etc.).

### 4. **Allow Remote Connections**

#### 4.1. **Configure MySQL to listen for remote connections:**

1. Open the MySQL configuration file (`/etc/mysql/mysql.conf.d/mysqld.cnf` on Ubuntu/Debian, `/etc/my.cnf` on CentOS/RHEL):

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

2. Find the line that says `bind-address` and change it to `0.0.0.0` to allow connections from any IP:

```ini
bind-address = 0.0.0.0
```

This allows MySQL to accept connections from any remote IP address. If you want to restrict access to a specific IP or range of IPs, replace `0.0.0.0` with the specific IP.

3. Save and close the file.

#### 4.2. **Grant privileges to a MySQL user for remote access:**

1. Log into MySQL as root:

```bash
sudo mysql -u root -p
```

2. Create a new MySQL user (or use an existing user) and grant remote access privileges. For example, to create a user `remote_user` with remote access from any IP:

```sql
CREATE USER 'remote_user'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON database_name.* TO 'remote_user'@'%';
FLUSH PRIVILEGES;
```

- Replace `'remote_user'`, `'password'`, and `'database_name'` with your actual username, password, and database name.
- `'%'` means any host (remote IP). You can specify a specific IP like `'192.168.1.100'` for more restricted access.

3. Exit MySQL:

```sql
EXIT;
```

### 5. **Open MySQL Port on Firewall**

You need to open port 3306 (MySQL default port) on your server's firewall.

#### On Ubuntu/Debian (using `ufw`):

```bash
sudo ufw allow from any to any port 3306 proto tcp
sudo ufw reload
```

#### On CentOS/RHEL (using `firewalld`):

```bash
sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent
sudo firewall-cmd --reload
```

### 6. **Test Remote Connection**

Now, from a remote machine, try connecting to the MySQL server using the MySQL client:

```bash
mysql -u remote_user -p -h <server_ip> -P 3306
```

- Replace `<server_ip>` with the IP address of your server.
- Enter the password for `remote_user` when prompted.

If the connection is successful, you have set up remote access properly.

### 7. **Configure PHP to Connect to MySQL Remotely**

To connect your PHP script to the remote MySQL server, you can use `mysqli` or `PDO`. Here's an example using `mysqli`:

1. **Install PHP MySQL extension** (if not already installed):

#### On Ubuntu/Debian:

```bash
sudo apt install php-mysqli
sudo systemctl restart apache2
```

#### On CentOS/RHEL:

```bash
sudo yum install php-mysqli
sudo systemctl restart httpd
```

2. **Create a PHP script to connect to MySQL:**

```php
<?php
$servername = "your_server_ip"; // MySQL server IP address
$username = "remote_user";      // MySQL username
$password = "password";         // MySQL password
$dbname = "database_name";      // Your database name

// Create connection
$conn = new mysqli($servername, $username, $password, $dbname);

// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

echo "Connected successfully";
?>
```

- Replace `your_server_ip`, `remote_user`, `password`, and `database_name` with your actual details.

3. **Test the PHP script**: Place the PHP script on your web server and access it through your browser. If everything is set up correctly, you should see "Connected successfully" or an error message if there is an issue.

### 8. **Troubleshooting**

- **Firewall issues**: If you cannot connect remotely, double-check your firewall settings.
- **MySQL user privileges**: Ensure the MySQL user has the correct privileges and is allowed to connect from your IP.
- **MySQL binding**: If `bind-address` is set to `127.0.0.1`, MySQL will only accept connections from the local machine. Make sure it's set to `0.0.0.0` or your specific IP.
- **PHP errors**: Check Apache or Nginx logs for any PHP errors if the connection doesn't work from your script.

By following these steps, you should have MySQL installed, configured for remote access, and connected to a PHP script running on your web server.
