.
Certainly! Below is the corrected and detailed guide to configuring Duo Authentication Proxy with FreeRADIUS and PAM for SSH, ensuring accuracy based on available documentation:

---

## Guide to Configure Duo Authentication Proxy with FreeRADIUS and PAM for SSH (Including API Authentication)

This guide outlines how to integrate Duo for two-factor authentication using **FreeRADIUS**, **PAM**, and **API authentication**.

---

### Step 1: Install Necessary Dependencies

Install the required dependencies for FreeRADIUS and Duo Authentication Proxy.

#### On RPM-based distributions (RHEL, CentOS, Fedora):
```bash
sudo yum install gcc make libffi-devel zlib-devel freeradius freeradius-utils freeradius-ldap freeradius-python3
```

#### On Debian-based distributions (Ubuntu, Debian):
```bash
sudo apt-get install build-essential libffi-dev zlib1g-dev freeradius freeradius-utils freeradius-ldap freeradius-python3
```

---

### Step 2: Download and Install Duo Authentication Proxy

1. **Download the Duo Authentication Proxy**:
   ```bash
   wget --content-disposition https://dl.duosecurity.com/duoauthproxy-latest-src.tgz
   ```

2. **Extract the Package**:
   ```bash
   tar xzf duoauthproxy-latest-src.tgz
   cd duoauthproxy-<version>
   ```

3. **Build and Install**:
   ```bash
   make
   sudo ./install
   ```

   The proxy installs to `/opt/duoauthproxy`.

---

### Step 3: Configure Duo Authentication Proxy

1. **Edit the Duo Authentication Proxy Configuration File**:
   ```bash
   sudo nano /opt/duoauthproxy/conf/authproxy.cfg
   ```

2. **Add the RADIUS Server Configuration**:
   ```ini
   [radius_server_auto]
   ikey=<your-integration-key>
   skey=<your-secret-key>
   api_host=<your-api-host>
   radius_ip_1=127.0.0.1
   secret=testing123
   failmode=safe
   ```
   Replace `<your-integration-key>`, `<your-secret-key>`, and `<your-api-host>` with your Duo Admin Panel values.

3. **Optional: Add HTTP Proxy Configuration**:
   ```ini
   [http_proxy]
   host=<proxy-host>
   port=<proxy-port>
   username=<proxy-username>
   password=<proxy-password>
   ```

4. **Restart the Duo Authentication Proxy**:
   ```bash
   sudo systemctl restart duoauthproxy
   ```

---

### Step 4: Configure FreeRADIUS to Use Duo Authentication Proxy

1. **Edit the RADIUS Default Site Configuration**:
   ```bash
   sudo nano /etc/freeradius/3.0/sites-enabled/default
   ```

   Add the following in the `authorize` section:
   ```bash
   authorize {
       duo
   }
   ```

   Add the following in the `authenticate` section:
   ```bash
   authenticate {
       Auth-Type DUO {
           duo
       }
   }
   ```

2. **Create Duo Module Configuration**:
   ```bash
   sudo nano /etc/freeradius/3.0/mods-available/duo
   ```

   Add:
   ```ini
   duo {
       ikey = <your-integration-key>
       skey = <your-secret-key>
       api_host = <your-api-host>
       failmode = safe
   }
   ```

   Enable the module:
   ```bash
   sudo ln -s /etc/freeradius/3.0/mods-available/duo /etc/freeradius/3.0/mods-enabled/
   ```

3. **Configure RADIUS Clients**:
   ```bash
   sudo nano /etc/freeradius/3.0/clients.conf
   ```

   Add:
   ```ini
   client localhost {
       ipaddr = 127.0.0.1
       secret = testing123
       require_message_authenticator = no
   }
   ```

4. **Restart FreeRADIUS**:
   ```bash
   sudo systemctl restart freeradius
   ```

---

### Step 5: Configure PAM for Duo with SSH

1. **Install the Duo PAM Module**:
   ```bash
   sudo apt-get install libpam-duo
   ```

2. **Edit PAM Configuration for SSH**:
   ```bash
   sudo nano /etc/pam.d/sshd
   ```

   Add:
   ```bash
   auth required pam_duo.so
   ```

3. **Edit the Duo PAM Configuration File**:
   ```bash
   sudo nano /etc/duo/pam_duo.conf
   ```

   Add:
   ```ini
   [duo]
   ikey=<your-integration-key>
   skey=<your-secret-key>
   host=<your-api-host>
   pushinfo=yes
   autopush=yes
   failmode=safe
   prompt=1
   groups=<group-name>
   ```
   Replace `<group-name>` with the user group requiring Duo.

4. **Edit the SSH Configuration File**:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

   Ensure:
   ```bash
   ChallengeResponseAuthentication yes
   UsePAM yes
   ```

5. **Restart SSH Service**:
   ```bash
   sudo systemctl restart sshd
   ```

---

### Step 6: Test the Configuration

1. **Test SSH Login**:
   ```bash
   ssh username@your-server
   ```
   After entering your password, Duo prompts for second-factor authentication.

2. **Test RADIUS Authentication**:
   ```bash
   radtest username password 127.0.0.1 0 testing123
   ```

3. **Test API Authentication**:
   ```bash
   curl -X POST https://<your-api-host>/auth/v2/auth \
     -H "Content-Type: application/json" \
     -d '{"ikey": "<your-integration-key>", "skey": "<your-secret-key>", "username": "test-user"}'
   ```

---

### Notes

1. **Shared Secret**:
   Ensure the shared secret (`testing123`) matches across Duo Proxy, FreeRADIUS, and `radtest`.

2. **Log Locations**:
   - Duo Authentication Proxy: `/opt/duoauthproxy/log/`
   - FreeRADIUS: `/var/log/freeradius/radius.log`

3. **Firewall and SELinux**:
   Allow RADIUS ports through the firewall and update SELinux rules:
   ```bash
   sudo firewall-cmd --add-port=1812/udp --permanent
   sudo firewall-cmd --reload
   sudo semanage port -a -t radiusd_port_t -p udp 1812
   ```

---

This comprehensive guide ensures proper integration of Duo with FreeRADIUS and PAM for SSH with accurate configurations.
