Certainly! Here's the corrected and refined version of your guide, making sure everything integrates properly with FreeRADIUS, and includes all necessary package installations for FreeRADIUS:

---

## Guide to Configure Duo Authentication Proxy with FreeRADIUS and PAM for SSH

This guide will walk you through the process of setting up Duo for two-factor authentication with **FreeRADIUS** and **PAM** on your SSH server.

### Step 1: Install the Necessary Dependencies

Ensure you have all the necessary dependencies installed for both **FreeRADIUS** and **Duo Authentication Proxy**.

#### On RPM-based distributions (Fedora, RHEL, CentOS):
```bash
sudo yum install gcc make libffi-devel zlib-devel diffutils freeradius freeradius-utils freeradius-ldap freeradius-python
```

#### On Debian-based distributions (Ubuntu, Debian):
```bash
sudo apt-get install build-essential libffi-dev zlib1g-dev freeradius freeradius-utils freeradius-ldap freeradius-python
```

For SELinux configurations (if applicable):

#### On RPM-based distributions:
```bash
sudo yum install selinux-policy-devel chkconfig
```

#### On Debian-based distributions:
```bash
sudo apt-get install selinux-policy-devel chkconfig
```

### Step 2: Download and Install the Duo Authentication Proxy

#### 1. Download the Duo Authentication Proxy:
Download the latest version of the Duo Authentication Proxy from Duo's official website:

```bash
wget --content-disposition https://dl.duosecurity.com/duoauthproxy-latest-src.tgz
```

#### 2. Extract and Build the Duo Authentication Proxy:
Extract the tarball and build the Duo Authentication Proxy:

```bash
tar xzf duoauthproxy-latest-src.tgz
cd duoauthproxy-<version>-src
make
```

#### 3. Install the Duo Authentication Proxy:
Install Duo Authentication Proxy after building it:

```bash
cd duoauthproxy-build
sudo ./install
```

During installation, you will be asked to configure the service user and group.

### Step 3: Configure the Duo Authentication Proxy

Once the Duo Authentication Proxy is installed, configure it by editing the `authproxy.cfg` file.

#### 1. Open the `authproxy.cfg` file for editing:
```bash
sudo nano /opt/duoauthproxy/etc/authproxy.cfg
```

#### 2. Configure the RADIUS Server Settings:

```ini
[radius_server_auto]
ikey = <your-ikey>
skey = <your-skey>
api_host = <your-api-host>
port = 1812
failmode = safe
```

Replace `<your-ikey>`, `<your-skey>`, and `<your-api-host>` with your Duo API keys and host.

#### 3. Configure the RADIUS Client Settings:
In the same file, add the configuration for FreeRADIUS:

```ini
[radius_client]
host = 127.0.0.1
secret = testing123
```

Replace `testing123` with the shared secret you want to use between Duo Authentication Proxy and FreeRADIUS.

### Step 4: Configure FreeRADIUS to Use Duo Authentication Proxy

Now, configure FreeRADIUS to use Duo for two-factor authentication.

#### 1. Edit the FreeRADIUS Configuration:

Open the FreeRADIUS `sites-enabled/default` configuration file:

```bash
sudo nano /etc/freeradius/3.0/sites-enabled/default
```

#### 2. Configure the Authentication Section:
In the `authenticate` section, add the Duo module configuration. Ensure that it is added under the correct section:

```bash
authenticate {
    ...
    duo {
        # Duo Authentication Proxy settings
        server = 127.0.0.1
        secret = testing123
        port = 1812
        failmode = safe
    }
}
```

#### 3. Enable the Duo Module:
Ensure that the Duo module is enabled. This should reside in `/etc/freeradius/3.0/mods-enabled/duo`:

```bash
sudo nano /etc/freeradius/3.0/mods-enabled/duo
```

Ensure that this configuration is correct:

```bash
duo {
    ikey = <your-ikey>
    skey = <your-skey>
    api_host = <your-api-host>
    failmode = safe
    radius_server = localhost
    port = 1812
}
```

Replace `<your-ikey>`, `<your-skey>`, and `<your-api-host>` with your Duo API keys and host.

### Step 5: Configure PAM for Duo Authentication in SSH

Next, configure **PAM** to work with Duo Authentication Proxy.

#### 1. Edit the `sshd` PAM Configuration:
Open the `/etc/pam.d/sshd` file for editing:

```bash
sudo nano /etc/pam.d/sshd
```

#### 2. Add Duo Authentication to PAM:
Add the following line to the file to enable Duo two-factor authentication after the standard SSH login:

```bash
auth required pam_duo.so
```

This line tells PAM to use the Duo module for authentication.

#### 3. Configure the Duo PAM Module:
Ensure that the Duo PAM configuration is correct. Open `/etc/duo/pam_duo.conf` and ensure the settings are correct for your environment:

```bash
# /etc/duo/pam_duo.conf
ikey = <your-ikey>
skey = <your-skey>
host = <your-api-host>
```

Replace `<your-ikey>`, `<your-skey>`, and `<your-api-host>` with your Duo API integration details.

### Step 6: Enable and Start the Duo Authentication Proxy and FreeRADIUS

#### 1. Enable and Start the Duo Authentication Proxy:

After configuring the Duo Authentication Proxy, start and enable it to run on system boot:

```bash
sudo systemctl enable duoauthproxy
sudo systemctl start duoauthproxy
```

#### 2. Restart FreeRADIUS:

After making configuration changes to FreeRADIUS, restart the service:

```bash
sudo systemctl restart freeradius
```

#### 3. Restart SSH:

To ensure PAM configuration changes are applied, restart the SSH service:

```bash
sudo systemctl restart sshd
```

### Step 7: Test Authentication

1. **Test via SSH:**

Try logging in via SSH from a client machine. You should be prompted for your Duo two-factor authentication after entering your password.

```bash
ssh user@your-server
```

2. **Test with `radtest`:**

Test FreeRADIUS with the `radtest` command to verify the configuration:

```bash
radtest username password 127.0.0.1 0 testing123
```

Check FreeRADIUS logs for any errors related to Duo authentication. Logs are typically located in `/var/log/freeradius/radius.log`.

### Step 8: (Optional) Silent Installation of Duo Authentication Proxy

If you prefer a silent installation of the Duo Authentication Proxy, you can use this command:

```bash
sudo ./duoauthproxy-build/install --install-dir /opt/duoauthproxy --service-user duo_authproxy_svc --log-group duo_authproxy_grp --create-init-script yes --enable-selinux=yes
```

### Conclusion

By following these steps, you will have successfully integrated **Duo Authentication Proxy** with **FreeRADIUS** and **PAM**, enabling two-factor authentication for SSH and other services using FreeRADIUS. Make sure to test thoroughly to ensure that the Duo authentication flow works as expected.

---

### Key Corrections/Clarifications:

1. **FreeRADIUS Installation**: 
   - Ensure that all necessary FreeRADIUS packages (including `freeradius-utils`, `freeradius-ldap`, `freeradius-python`) are installed.

2. **Correct Authentication Module Configuration**:
   - Ensure the `duo` module is added in the `authenticate` section of `/etc/freeradius/3.0/sites-enabled/default`.

3. **Shared Secret**:
   - Ensure the shared secret (`testing123`) matches what is set in both the Duo Authentication Proxy (`authproxy.cfg`) and FreeRADIUS.

4. **PAM Configuration**:
   - Correctly add the `auth required pam_duo.so` line in `/etc/pam.d/sshd` to invoke the Duo PAM module.

5. **Service Restarts**:
   - Ensure to restart all relevant services (`duoauthproxy`, `freeradius`, and `sshd`) after making changes.

By following these detailed steps with the corrected installation instructions, the integration should work smoothly.
