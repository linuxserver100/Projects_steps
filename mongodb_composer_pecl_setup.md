To install the MongoDB PHP driver on Ubuntu (or any Linux-based system), you need to follow these steps:

### Step 1: Install Dependencies
Make sure your system is up-to-date and has the necessary dependencies installed:

1. **Update package list and upgrade existing packages**:
   ```bash
   sudo apt update
   sudo apt upgrade
   ```

2. **Install required dependencies**:
   ```bash
   sudo apt install php-pear php-dev libmongoc-1.0-0 libsasl2-dev libssl-dev
   ```

   - `php-pear` is used for installing PECL extensions.
   - `php-dev` includes necessary development tools to build PHP extensions.
   - `libmongoc-1.0-0` is the MongoDB C driver.
   - `libsasl2-dev` and `libssl-dev` are development libraries required for the MongoDB PHP driver.

### Step 2: Install the MongoDB PHP Driver via PECL
Once dependencies are installed, you can install the MongoDB PHP driver using `pecl`.

1. **Install the MongoDB PHP driver**:
   ```bash
   sudo pecl install mongodb
   ```

   This will download, compile, and install the MongoDB driver.

### Step 3: Enable the MongoDB Extension in PHP
After installing the MongoDB driver, you need to enable it in PHP.

1. **Create a configuration file** for the MongoDB extension:
   ```bash
   echo "extension=mongodb.so" | sudo tee /etc/php/$(php -r 'echo PHP_VERSION;')/cli/conf.d/20-mongodb.ini
   ```

   This command dynamically detects your PHP version and adds the MongoDB extension to the appropriate configuration directory.

2. If you're using PHP-FPM (for web servers like Nginx or Apache), enable the extension for that as well:
   ```bash
   echo "extension=mongodb.so" | sudo tee /etc/php/$(php -r 'echo PHP_VERSION;')/fpm/conf.d/20-mongodb.ini
   ```

3. **Restart PHP-FPM** (if applicable) and Apache or Nginx to apply changes:
   ```bash
   sudo systemctl restart php$(php -r 'echo PHP_VERSION;')-fpm
   sudo systemctl restart apache2  # If you're using Apache
   sudo systemctl restart nginx    # If you're using Nginx
   ```

### Step 4: Verify the Installation
To verify that the MongoDB PHP driver has been successfully installed:

1. Run the following command to check if the MongoDB extension is enabled:
   ```bash
   php -m | grep mongodb
   ```

   If everything is set up correctly, this command should output `mongodb`.

2. You can also verify the version of the MongoDB extension by running:
   ```bash
   php -r "echo phpinfo();" | grep -A 10 "mongodb"
   ```

   This should show details about the MongoDB extension, including its version and configuration.

### Step 5: Install the MongoDB PHP Library (Optional)
To interact with MongoDB from PHP, you’ll likely also want the official MongoDB PHP library.

1. Install it via Composer (PHP’s package manager):
   ```bash
   composer require mongodb/mongodb
   ```

   This will install the MongoDB client library for PHP.

### Conclusion
That’s it! You’ve now installed the MongoDB PHP driver and can start working with MongoDB in your PHP applications.
