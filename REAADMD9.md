
Setting up a mail server on an Ubuntu server that can send and receive emails (even if port 25 is not open) with a web UI involves several steps. In this guide, we'll use **Postfix** for sending emails, **Dovecot** for receiving emails, and **Roundcube** as a webmail client. We’ll also set up a method to send emails through an alternate port.

### Prerequisites

1. **Ubuntu Server**: Ensure you have Ubuntu 20.04 or later installed.
2. **Domain Name**: You need a registered domain name.
3. **Basic command-line knowledge**: Familiarity with terminal commands.

### Step 1: Update the System

Start by updating your system packages:

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 2: Install Necessary Packages

Install Postfix, Dovecot, and required PHP packages for Roundcube:

```bash
sudo apt install postfix dovecot-core dovecot-imapd dovecot-pop3d -y
sudo apt install apache2 php libapache2-mod-php php-mysql php-pear php-mbstring php-xml php-json php-curl php-gd -y
```

During the Postfix installation, you will be prompted to choose the general type of mail configuration. Choose "Internet Site" and enter your domain name.

### Step 3: Configure Postfix

Edit the Postfix configuration file:

```bash
sudo nano /etc/postfix/main.cf
```

Configure the following parameters:

```plaintext
myhostname = mail.yourdomain.com
mydomain = yourdomain.com
myorigin = /etc/mailname
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
relayhost = 
mynetworks = 127.0.0.0/8
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
inet_protocols = ipv4
```

### Step 4: Configure Dovecot

Edit the Dovecot configuration:

```bash
sudo nano /etc/dovecot/dovecot.conf
```

Ensure the following lines are included:

```plaintext
mail_location = maildir:~/Maildir
```

Enable the IMAP and POP3 protocols by editing the `10-master.conf` file:

```bash
sudo nano /etc/dovecot/conf.d/10-master.conf
```

Modify the following sections:

```plaintext
service imap-login {
  inet_listener imap {
    port = 0
  }
  inet_listener imaps {
    port = 993
    ssl = yes
  }
}

service pop3-login {
  inet_listener pop3 {
    port = 0
  }
  inet_listener pop3s {
    port = 995
    ssl = yes
  }
}
```

### Step 5: Configure SSL with Let's Encrypt

To secure your mail server with SSL, install Certbot:

```bash
sudo apt install certbot
```

Obtain an SSL certificate:

```bash
sudo certbot certonly --standalone -d mail.yourdomain.com
```

Configure Postfix and Dovecot to use SSL:

**For Postfix:**

Edit the main configuration:

```bash
sudo nano /etc/postfix/main.cf
```

Add these lines:

```plaintext
smtpd_tls_cert_file=/etc/letsencrypt/live/mail.yourdomain.com/fullchain.pem
smtpd_tls_key_file=/etc/letsencrypt/live/mail.yourdomain.com/privkey.pem
smtpd_use_tls=yes
```

**For Dovecot:**

Edit the Dovecot SSL configuration:

```bash
sudo nano /etc/dovecot/conf.d/10-ssl.conf
```

Set:

```plaintext
ssl = required
ssl_cert = </etc/letsencrypt/live/mail.yourdomain.com/fullchain.pem
ssl_key = </etc/letsencrypt/live/mail.yourdomain.com/privkey.pem
```

### Step 6: Configure DNS Settings

In your Cloudflare or DNS provider's dashboard:

1. **Create an MX Record**:
   - Type: MX
   - Name: `@`
   - Mail server: `mail.yourdomain.com`
   - Priority: 10

2. **Create an A Record**:
   - Type: A
   - Name: `mail`
   - Value: Your server's public IP address
   - TTL: Auto

3. **Set up SPF Record** (to allow your server to send emails):
   - Type: TXT
   - Name: `@`
   - Value: `v=spf1 mx a ip4:your_server_ip ~all`

4. **Create DKIM and DMARC Records** for improved email deliverability (optional).

### Step 7: Install Roundcube

**1. Download Roundcube**

Go to the Roundcube download page and get the latest version:

```bash
cd /var/www/html
sudo wget https://github.com/roundcube/roundcubemail/releases/download/latest/roundcubemail-latest.tar.gz
sudo tar -xzf roundcubemail-latest.tar.gz
sudo mv roundcubemail-* roundcube
```

**2. Configure Roundcube**

Rename the configuration sample file:

```bash
cd roundcube
sudo cp config/config.inc.php.sample config/config.inc.php
```

Edit the configuration file:

```bash
sudo nano config/config.inc.php
```

Update the following lines:

```php
$config['db_dsnw'] = 'mysql://roundcube:password@localhost/roundcubemail';
$config['default_host'] = 'ssl://localhost';
$config['default_port'] = 993;
$config['smtp_server'] = 'ssl://localhost';
$config['smtp_port'] = 465;
```

**3. Create Roundcube Database**

Log in to MySQL or MariaDB:

```bash
sudo mysql -u root -p
```

Run the following commands to create the database and user:

```sql
CREATE DATABASE roundcubemail;
CREATE USER 'roundcube'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON roundcubemail.* TO 'roundcube'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

**4. Run Roundcube Installer**

Open your web browser and navigate to `http://yourdomain.com/roundcube/installer`.

Follow the on-screen instructions to complete the installation, and then delete the installer directory for security:

```bash
sudo rm -rf /var/www/html/roundcube/installer
```

### Step 8: Restart Services

Restart Postfix and Dovecot:

```bash
sudo systemctl restart postfix
sudo systemctl restart dovecot
```

### Step 9: Handle Port 25 Restrictions

If port 25 is blocked, configure Postfix to use an alternate SMTP port like 587.

**1. Edit Postfix Configuration:**

Add the following lines to `/etc/postfix/master.cf`:

```plaintext
submission inet n       -       y       -       -       smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_type=dovecot
  -o smtpd_sasl_path=private/auth
  -o smtpd_sasl_local_domain=
  -o smtpd_recipient_restrictions=permit_sasl_authenticated,reject
```

**2. Restart Postfix:**

```bash
sudo systemctl restart postfix
```

### Step 10: Testing the Setup

1. **Send a Test Email**: Use Roundcube or another mail client to send a test email.
2. **Receive a Test Email**: Send an email from an external address to your new email account and check if it arrives.

### Conclusion

You have now successfully set up a mail server on Ubuntu to send and receive emails with a web UI using Roundcube, even if port 25 is blocked. Ensure you regularly monitor and maintain your server for optimal performance and security.
