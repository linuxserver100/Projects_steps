
Setting up email OTP (One-Time Password) login on an Ubuntu server requires a combination of tools to generate and send OTPs via email. Here’s a step-by-step guide using Python and Postfix for sending emails.

### Step 1: Install Required Packages

1. **Update your package list**:
   ```bash
   sudo apt update
   ```

2. **Install required packages**:
   ```bash
   sudo apt install python3 python3-pip postfix
   ```

3. **Install the necessary Python libraries**:
   ```bash
   pip3 install Flask Flask-Mail pyotp
   ```

### Step 2: Configure Postfix for Email Sending

1. **Configure Postfix**:
   - During installation, you may be prompted for the type of mail configuration. Choose "Internet Site".
   - Set the system mail name to your server's domain name or hostname.

2. **Edit the Postfix configuration**:
   ```bash
   sudo nano /etc/postfix/main.cf
   ```

   - Ensure the following lines are present and correctly configured:
   ```plaintext
   myhostname = yourdomain.com
   mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
   ```

3. **Restart Postfix**:
   ```bash
   sudo systemctl restart postfix
   ```

### Step 3: Create a Python Script for OTP Generation and Email Sending

1. **Create a directory for your script**:
   ```bash
   mkdir ~/otp_login
   cd ~/otp_login
   ```

2. **Create a Python script (otp_login.py)**:
   ```bash
   nano otp_login.py
   ```

3. **Add the following code**:
   ```python
   from flask import Flask, request, jsonify
   from flask_mail import Mail, Message
   import pyotp
   import os

   app = Flask(__name__)

   # Configure your email settings
   app.config['MAIL_SERVER'] = 'smtp.example.com'  # Replace with your SMTP server
   app.config['MAIL_PORT'] = 587  # or 465 for SSL
   app.config['MAIL_USERNAME'] = 'your_email@example.com'  # Your email
   app.config['MAIL_PASSWORD'] = 'your_email_password'  # Your email password
   app.config['MAIL_USE_TLS'] = True
   app.config['MAIL_USE_SSL'] = False

   mail = Mail(app)

   @app.route('/login', methods=['POST'])
   def login():
       email = request.json.get('email')
       if not email:
           return jsonify({'error': 'Email is required!'}), 400

       # Generate OTP
       totp = pyotp.TOTP('base32secret3232')  # You should store this secret securely
       otp = totp.now()

       # Send OTP via email
       msg = Message('Your OTP Code', sender='your_email@example.com', recipients=[email])
       msg.body = f'Your OTP code is {otp}.'
       mail.send(msg)

       return jsonify({'message': 'OTP sent to your email!'}), 200

   @app.route('/verify', methods=['POST'])
   def verify():
       email = request.json.get('email')
       otp = request.json.get('otp')
       if not email or not otp:
           return jsonify({'error': 'Email and OTP are required!'}), 400

       totp = pyotp.TOTP('base32secret3232')
       if totp.verify(otp):
           return jsonify({'message': 'OTP verified successfully!'}), 200
       else:
           return jsonify({'error': 'Invalid OTP!'}), 400

   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000)
   ```

### Step 4: Run the Python Script

1. **Run the script**:
   ```bash
   python3 otp_login.py
   ```

### Step 5: Test the OTP Login

1. **Send a login request**:
   - Use `curl` or Postman to send a POST request to `http://your_server_ip:5000/login` with a JSON body:
   ```json
   {
       "email": "user@example.com"
   }
   ```

2. **Check the email** for the OTP and verify it by sending another POST request to `http://your_server_ip:5000/verify`:
   ```json
   {
       "email": "user@example.com",
       "otp": "your_otp_here"
   }
   ```

### Step 6: Secure Your Application

- Consider implementing HTTPS to secure your API.
- Store sensitive data, such as your email password and TOTP secret, securely (e.g., using environment variables).

### Additional Considerations

- Ensure your server can send emails and that your SMTP settings are correct.
- You may need to adjust firewall settings to allow traffic on port 5000.

This guide provides a simple way to implement email OTP login on an Ubuntu server using Flask and Postfix. Adjust the code as necessary to fit your specific requirements.
