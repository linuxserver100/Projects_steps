Deploying a full-stack React application via a CI/CD pipeline with rollback functionality and deploying it on a server using Apache and SSH key-based authentication involves several steps. I’ll break down the process into:

1. **Setting up SSH Key Authentication**
2. **Setting up a React Full-Stack App**
3. **Configuring CI/CD Pipeline**
4. **Deploying the App to a Server via Apache**
5. **Adding Rollback Functionality**

---

### 1. **Setting Up SSH Key Authentication**
Ensure that the server you are deploying to uses SSH key authentication for secure and automated access.

1. **Generate SSH Key Pair** (on local machine or CI server):
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

   This will create two files: 
   - `id_rsa`: your private key (keep this secure).
   - `id_rsa.pub`: your public key.

2. **Copy the Public Key to the Server**:
   ```bash
   ssh-copy-id username@server_ip_address
   ```
   This adds your public key to the `~/.ssh/authorized_keys` file on the server.

3. **Test SSH Authentication**:
   Verify that you can access the server without being prompted for a password:
   ```bash
   ssh username@server_ip_address
   ```

### 2. **Setting Up a React Full-Stack App**

#### Frontend (React):
- Create your React app:
  ```bash
  npx create-react-app my-app
  cd my-app
  ```
- Build the app for production:
  ```bash
  npm run build
  ```
  This will create a `build/` directory with a static production-ready bundle.

#### Backend (Node.js/Express):
- Initialize the backend:
  ```bash
  mkdir backend && cd backend
  npm init -y
  npm install express
  ```
- Set up a basic Express server:
  ```js
  const express = require('express');
  const app = express();
  const path = require('path');

  app.use(express.static(path.join(__dirname, 'frontend/build')));

  app.get('*', (req, res) => {
    res.sendFile(path.join(__dirname, 'frontend/build', 'index.html'));
  });

  const PORT = process.env.PORT || 5000;
  app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
  });
  ```
- Link the frontend’s production build to the backend by copying the React build to the backend directory:
  ```bash
  cp -r ../my-app/build ./frontend/build
  ```

### 3. **Configuring CI/CD Pipeline**

Let’s assume you’re using GitHub Actions for CI/CD, but this can be adapted to other platforms like GitLab, CircleCI, etc.

#### GitHub Actions Workflow:
Create a `.github/workflows/deploy.yml` file in your repository.

```yaml
name: Deploy to Server

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '16'

    - name: Install dependencies and build frontend
      run: |
        cd my-app
        npm install
        npm run build
        cp -r build ../backend/frontend/build

    - name: Set up SSH
      uses: webfactory/ssh-agent@v0.5.3
      with:
        ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

    - name: Deploy to server
      run: |
        scp -r ./backend/* username@server_ip:/var/www/my-app
        ssh username@server_ip 'cd /var/www/my-app && npm install --production && pm2 restart all'
```

- You will need to set the `SSH_PRIVATE_KEY` in GitHub Secrets (Settings > Secrets).
- Adjust the paths as needed based on your app structure.
- Install [PM2](https://pm2.keymetrics.io/) on your server for process management:
  ```bash
  npm install -g pm2
  ```

### 4. **Deploying the App to a Server via Apache**

1. **Install Apache**:
   On the server, install Apache:
   ```bash
   sudo apt-get update
   sudo apt-get install apache2
   ```

2. **Enable Reverse Proxy Modules** (if you are using Node.js backend):
   ```bash
   sudo a2enmod proxy proxy_http
   sudo systemctl restart apache2
   ```

3. **Configure Virtual Host for Domain**:
   Edit the Apache config to point to your Node.js app:
   ```bash
   sudo nano /etc/apache2/sites-available/000-default.conf
   ```

   Add the following configuration inside `<VirtualHost *:80>`:
   ```apache
   ServerName your-domain.com
   ProxyRequests Off
   ProxyPass / http://localhost:5000/
   ProxyPassReverse / http://localhost:5000/
   ```

4. **Enable Your Virtual Host and Restart Apache**:
   ```bash
   sudo a2ensite 000-default.conf
   sudo systemctl restart apache2
   ```

5. **Set Up Domain Name**:
   - Make sure your domain name is properly pointed to your server IP (via DNS settings).
   - Optionally, install an SSL certificate using [Let's Encrypt](https://letsencrypt.org/):
     ```bash
     sudo apt-get install certbot python3-certbot-apache
     sudo certbot --apache -d your-domain.com
     ```

### 5. **Adding Rollback Functionality**

To implement a rollback mechanism, you can deploy the app to a timestamped directory and symlink the current version. If deployment fails, you can quickly point back to the previous version.

#### Update CI/CD for Rollbacks:

1. **Deploy to a Timestamped Directory**:
   In your GitHub Actions workflow, modify the deployment step:

   ```yaml
   - name: Deploy to server
     run: |
       TIMESTAMP=$(date +%s)
       ssh username@server_ip "mkdir -p /var/www/my-app-$TIMESTAMP"
       scp -r ./backend/* username@server_ip:/var/www/my-app-$TIMESTAMP
       ssh username@server_ip "ln -sfn /var/www/my-app-$TIMESTAMP /var/www/my-app"
       ssh username@server_ip "cd /var/www/my-app && npm install --production && pm2 restart all"
   ```

2. **Rollback Command**:
   If the new version fails, you can manually or automatically rollback by pointing the symlink back to the previous deployment:
   ```bash
   ln -sfn /var/www/my-app-previous-timestamp /var/www/my-app
   pm2 restart all
   ```

---

### Conclusion

By setting up SSH key-based authentication, configuring GitHub Actions for CI/CD, and using Apache as a reverse proxy, you can deploy a full-stack React app securely. The rollback mechanism with timestamped directories ensures that you can quickly revert to a stable version if the deployment fails


😄😁🔑🤪🔑🤪😄😁🤪😁🤪🤩🤪👇🤪😁🤪🙂‍↕️🤪🤩🤪😁🤪🙂‍↕️☺️🙂‍↕️🤪😄😄☺️😁😁🤪🙂‍↕️🙂‍↕️☺️🙂‍↕️☺️🥹☺️🤩😚😚😄👇😅😄😝🤩😝🤩😅🥹😅🥹😄😂🥹😂🥹😂😜😜😚😆😘🥹🥹😘🥲😘😂😘😆😘😜😘🥲😘🙂‍↕️😘🤩😘🤩😘🥲😘😚🔐🥹🤩🔐🤩🔐😄🔐🤩🔐😜🔐🥹🔐🥲🔐😜🔐🥹🤩🔐😜🤩😚😜😚🥹😚🥹😚🤩😚🤩😜😚🥲😚😚😜😚🥲😚🙂‍↕️🙂‍↕️😚😜😚😜😚😜😚😜😚🥹😚🤩🤩😚🤩
😚
🥲😚😜😚🥹🤪🥭😆🤪😜🤪🥹🤪😜🤪🥲☺️🤩☺️🥹☺️🥹☺️🥹☺️🤩☺️😜☺️😜☺️🥲🤩😘😜😘😜😘🥲😘😜😘🤩😚🥲🥲😚🤩😚😜😚😜😚🤩😂😚🤩🔐😆🔐😜🔐😜😚😜😚🥹🥲😚🥲😚😜😚😜😚🥹😚🤩😚😜🔐😜🔐🥹🔐

Deploying a full-stack React application via a CI/CD pipeline with rollback functionality, SSH key authentication, and deployment over Apache using GitLab CI with artifacts can be broken down into the following steps:

1. **Setting up SSH Key Authentication**
2. **Setting up a React Full-Stack App**
3. **Creating a GitLab CI/CD Pipeline with Artifacts**
4. **Deploying the App to a Server via Apache**
5. **Implementing Rollback Functionality with GitLab and Artifacts**

---

### 1. **Setting Up SSH Key Authentication**

1. **Generate SSH Key Pair** (on your local machine or GitLab CI runner):
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

   This will create two files:
   - `id_rsa`: your private key (used in GitLab CI/CD).
   - `id_rsa.pub`: your public key (added to the server).

2. **Copy the Public Key to the Server**:
   ```bash
   ssh-copy-id username@server_ip_address
   ```

3. **Test SSH Authentication**:
   Verify that you can log in to the server without a password:
   ```bash
   ssh username@server_ip_address
   ```

4. **Add Private Key to GitLab**:
   - In your GitLab project, navigate to **Settings** > **CI/CD** > **Variables**.
   - Add a new variable with the key `SSH_PRIVATE_KEY` and the private key (`id_rsa`) as the value.

---

### 2. **Setting Up a React Full-Stack App**

#### Frontend (React):
- Create the React app:
  ```bash
  npx create-react-app my-app
  cd my-app
  ```
- Build the React app for production:
  ```bash
  npm run build
  ```

#### Backend (Node.js/Express):
- Initialize the backend:
  ```bash
  mkdir backend && cd backend
  npm init -y
  npm install express
  ```
- Set up a basic Express server:
  ```js
  const express = require('express');
  const app = express();
  const path = require('path');

  app.use(express.static(path.join(__dirname, 'frontend/build')));

  app.get('*', (req, res) => {
    res.sendFile(path.join(__dirname, 'frontend/build', 'index.html'));
  });

  const PORT = process.env.PORT || 5000;
  app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
  });
  ```
- Copy the React production build into the backend folder:
  ```bash
  cp -r ../my-app/build ./frontend/build
  ```

---

### 3. **Creating a GitLab CI/CD Pipeline with Artifacts**

In GitLab CI, artifacts can be used to store the build output (e.g., React `build/` files) and reuse them across different pipeline stages.

#### GitLab CI Configuration (`.gitlab-ci.yml`):
Create a `.gitlab-ci.yml` file in the root of your repository.

```yaml
stages:
  - build
  - deploy

# Build the frontend (React) and backend
build:
  stage: build
  image: node:16
  script:
    - cd my-app
    - npm install
    - npm run build
    - cd ..
    - cp -r my-app/build backend/frontend/build
  artifacts:
    paths:
      - backend/frontend/build
    expire_in: 1 week

# Deploy to server using SSH
deploy:
  stage: deploy
  image: node:16
  before_script:
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan -H $DEPLOY_SERVER_IP >> ~/.ssh/known_hosts
  script:
    - scp -r ./backend/* $DEPLOY_USER@$DEPLOY_SERVER_IP:/var/www/my-app
    - ssh $DEPLOY_USER@$DEPLOY_SERVER_IP 'cd /var/www/my-app && npm install --production && pm2 restart all'
  only:
    - main
```

- **SSH_PRIVATE_KEY**: This environment variable stores the private SSH key, which should be added in GitLab CI/CD variables.
- **DEPLOY_USER** and **DEPLOY_SERVER_IP**: Add these environment variables in GitLab CI/CD to define your deployment server credentials.
  
#### Artifacts Explained:
- In the `build` job, the React app is built and stored as artifacts in the `backend/frontend/build` folder.
- The `deploy` job reuses these artifacts to deploy to the server.

---

### 4. **Deploying the App to a Server via Apache**

1. **Install Apache** on the server:
   ```bash
   sudo apt-get update
   sudo apt-get install apache2
   ```

2. **Enable Reverse Proxy Modules** for forwarding requests to your Node.js backend:
   ```bash
   sudo a2enmod proxy proxy_http
   sudo systemctl restart apache2
   ```

3. **Configure Apache Virtual Host**:
   Edit the Apache config to reverse proxy requests to your Node.js backend:
   ```bash
   sudo nano /etc/apache2/sites-available/000-default.conf
   ```

   Add the following configuration:
   ```apache
   <VirtualHost *:80>
     ServerName your-domain.com
     ProxyRequests Off
     ProxyPass / http://localhost:5000/
     ProxyPassReverse / http://localhost:5000/
   </VirtualHost>
   ```

4. **Enable Your Site and Restart Apache**:
   ```bash
   sudo a2ensite 000-default.conf
   sudo systemctl restart apache2
   ```

5. **Set Up Domain Name**:
   - Ensure that your domain points to your server IP via DNS settings.
   - Optionally, you can install an SSL certificate using [Let's Encrypt](https://letsencrypt.org/):
     ```bash
     sudo apt-get install certbot python3-certbot-apache
     sudo certbot --apache -d your-domain.com
     ```

---

### 5. **Implementing Rollback Functionality with GitLab and Artifacts**

For rollback functionality, the key is to keep previous deployment versions and have the ability to switch between them quickly.

#### Steps for Rollback with GitLab Artifacts:

1. **Deploy to Timestamped Directories**:
   Update your GitLab `deploy` job to deploy to a timestamped directory:
   ```yaml
   deploy:
     stage: deploy
     image: node:16
     before_script:
       - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
       - eval $(ssh-agent -s)
       - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
       - mkdir -p ~/.ssh
       - chmod 700 ~/.ssh
       - ssh-keyscan -H $DEPLOY_SERVER_IP >> ~/.ssh/known_hosts
     script:
       - TIMESTAMP=$(date +%Y%m%d%H%M%S)
       - ssh $DEPLOY_USER@$DEPLOY_SERVER_IP "mkdir -p /var/www/my-app-$TIMESTAMP"
       - scp -r ./backend/* $DEPLOY_USER@$DEPLOY_SERVER_IP:/var/www/my-app-$TIMESTAMP
       - ssh $DEPLOY_USER@$DEPLOY_SERVER_IP "ln -sfn /var/www/my-app-$TIMESTAMP /var/www/my-app"
       - ssh $DEPLOY_USER@$DEPLOY_SERVER_IP "cd /var/www/my-app && npm install --production && pm2 restart all"
   ```

2. **Rollback Command**:
   To rollback to a previous version, update the symlink to point to an older deployment:
   ```bash
   ssh $DEPLOY_USER@$DEPLOY_SERVER_IP "ln -sfn /var/www/my-app-previous-timestamp /var/www/my-app"
   ssh $DEPLOY_USER@$DEPLOY_SERVER_IP "pm2 restart all"
   ```

3. **Automating Rollbacks**:
   You can create a separate GitLab job for manual rollback, where you specify which previous version (timestamp) to roll back to:
   ```yaml
   rollback:
     stage: deploy
     script:
       - ssh $DEPLOY_USER@$DEPLOY_SERVER_IP "ln -sfn /var/www/my-app-$ROLLBACK_TIMESTAMP /var/www/my-app"
       - ssh $DEPLOY_USER@$DEPLOY_SERVER_IP "pm2 restart all"
     when: manual
     only:
       - main
   ```

   You can trigger this manually via the GitLab pipeline UI, and set the `ROLLBACK_TIMESTAMP` as an environment variable.

---

### Conclusion

With this approach, you’ve set up a GitLab CI/CD pipeline that builds, deploys, and includes rollback functionality using SSH key authentication, Apache, and domain configuration. You can deploy the full-stack React app with confidence, knowing that you can roll back to a previous version if needed using GitLab artifacts.


😗☺️😙☺️😚😁☺️😘😍😆😂😁😚😍😄😂😙😂😙😍😙😍😚😍😚😍😘😍😚🤩😙🤩😙🤩😚🤩😚🤩😚🤩😚😚🤩😚🤩😚🤩😚🤩😘🤩😘🤩😚🤩😚🤩😚🤩😚🤩😚🤩😚🤩😚🤩😁🤩😚🤩😚😚🤩😚🤩😚🤩😚🤩😁🤩😁🤩😁🤩😆🤩😆🤩😁🤩🤩😚🤩😚😁🤩😁🤩😁🤩😆🤩😚🤩😚🤩😁🤩😆🤩🤩😚🤩😁🤩😁🤩😁🤩😚🤩😚🤩😁🤩😁😍😘🤩😙🤩😗🤩😄🤩😁🤩😆🤩😚🤩😚😁🤩😁🤩😘🤩😙🤩😗🤩😉🤣🙃🤩🙂🤩🥲🤩🙂🤩😙🤩
🤪🤪🥰😆😊

To deploy a full-stack React application via GitLab CI/CD with rollback functionality, SSH key authentication, and deployment over Apache for both frontend and backend, while ensuring artifacts are included in the pipeline, follow these steps:

### Step-by-Step Guide:

1. **SSH Key Authentication Setup**
2. **React Full-Stack Application Setup**
3. **Apache Configuration for Backend and Frontend with a Domain**
4. **CI/CD Pipeline Configuration with GitLab**
5. **Adding Rollback Functionality**

---

### 1. **SSH Key Authentication Setup**

Before setting up the pipeline, make sure you can SSH into your server using SSH keys.

- **Generate SSH Key Pair**:
  On your local machine or CI runner:
  ```bash
  ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
  ```

- **Copy the Public Key to the Server**:
  ```bash
  ssh-copy-id username@server_ip_address
  ```

- **Test the Connection**:
  ```bash
  ssh username@server_ip_address
  ```

- **Store SSH Private Key in GitLab**:
  - Go to your GitLab project > Settings > CI/CD > Variables.
  - Add a new variable `SSH_PRIVATE_KEY` with the value of your private SSH key (`id_rsa`).
  - Optionally, add `DEPLOY_USER` and `DEPLOY_SERVER_IP` variables for better script management.

---

### 2. **React Full-Stack Application Setup**

#### **Frontend: React**
- Create your React app:
  ```bash
  npx create-react-app my-app
  cd my-app
  ```

- Build the production version:
  ```bash
  npm run build
  ```

#### **Backend: Node.js/Express**
- Initialize a backend directory:
  ```bash
  mkdir backend && cd backend
  npm init -y
  npm install express
  ```

- Basic Express server setup:
  ```js
  const express = require('express');
  const path = require('path');
  const app = express();

  // Serve static files from the React frontend app
  app.use(express.static(path.join(__dirname, 'frontend/build')));

  // Handle React routing, return all requests to the React app
  app.get('*', (req, res) => {
    res.sendFile(path.join(__dirname, 'frontend/build', 'index.html'));
  });

  const PORT = process.env.PORT || 5000;
  app.listen(PORT, () => {
    console.log(`Server listening on port ${PORT}`);
  });
  ```

- **Copy the React production build to the backend**:
  ```bash
  cp -r ../my-app/build ./frontend/build
  ```

---

### 3. **Apache Configuration for Backend and Frontend with a Domain**

#### **Install Apache on the Server**:
On your server, install Apache:
```bash
sudo apt-get update
sudo apt-get install apache2
```

#### **Enable Apache Proxy Modules**:
You will need to enable Apache modules for proxying requests from Apache to the Node.js backend.

```bash
sudo a2enmod proxy proxy_http
sudo systemctl restart apache2
```

#### **Virtual Host Configuration for Apache**:
Configure Apache to serve both your frontend (React) and backend (Node.js) through a domain name.

1. **Create a virtual host file**:
   ```bash
   sudo nano /etc/apache2/sites-available/your-domain.com.conf
   ```

2. **Add the following configuration** to route requests:
   ```apache
   <VirtualHost *:80>
       ServerName your-domain.com
       DocumentRoot /var/www/my-app/frontend/build

       ProxyRequests Off
       ProxyPreserveHost On

       # Proxy for Backend (Node.js server running on port 5000)
       ProxyPass /api http://localhost:5000/api
       ProxyPassReverse /api http://localhost:5000/api

       # Serve React Frontend
       <Directory /var/www/my-app/frontend/build>
           Options Indexes FollowSymLinks
           AllowOverride All
           Require all granted
       </Directory>

       # Handle React Router (if using client-side routing)
       ErrorDocument 404 /index.html

       # Log files
       ErrorLog ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```

3. **Enable the new site**:
   ```bash
   sudo a2ensite your-domain.com.conf
   sudo systemctl restart apache2
   ```

4. **Update DNS**:
   Make sure your domain name (e.g., `your-domain.com`) points to your server's IP address using your DNS provider.

5. **(Optional) SSL with Let’s Encrypt**:
   You can secure your site with SSL using [Certbot](https://certbot.eff.org/):
   ```bash
   sudo apt-get install certbot python3-certbot-apache
   sudo certbot --apache -d your-domain.com
   ```

---

### 4. **GitLab CI/CD Pipeline Configuration with Artifacts**

Create a `.gitlab-ci.yml` file in the root of your project to define your GitLab CI/CD pipeline.

#### **GitLab CI/CD Pipeline:**

```yaml
stages:
  - build
  - deploy

variables:
  BACKEND_DIR: backend
  FRONTEND_DIR: my-app

# Build the frontend and backend
build:
  stage: build
  image: node:16
  script:
    # Build the frontend React app
    - cd $FRONTEND_DIR
    - npm install
    - npm run build
    - cd ..

    # Copy the built React app into the backend
    - cp -r $FRONTEND_DIR/build $BACKEND_DIR/frontend/build

    # Package backend (if needed)
    - cd $BACKEND_DIR
    - npm install

  artifacts:
    paths:
      - $BACKEND_DIR/frontend/build
    expire_in: 1 week

# Deploy to server using SSH
deploy:
  stage: deploy
  image: node:16
  before_script:
    # Install SSH Agent and configure SSH key
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan -H $DEPLOY_SERVER_IP >> ~/.ssh/known_hosts

  script:
    # Create timestamped deployment directory
    - TIMESTAMP=$(date +%Y%m%d%H%M%S)
    - ssh $DEPLOY_USER@$DEPLOY_SERVER_IP "mkdir -p /var/www/my-app-$TIMESTAMP"

    # Copy backend files (including frontend build) to the server
    - scp -r $BACKEND_DIR/* $DEPLOY_USER@$DEPLOY_SERVER_IP:/var/www/my-app-$TIMESTAMP

    # Symlink new deployment to "current" directory
    - ssh $DEPLOY_USER@$DEPLOY_SERVER_IP "ln -sfn /var/www/my-app-$TIMESTAMP /var/www/my-app"

    # Install dependencies on the server and restart the backend
    - ssh $DEPLOY_USER@$DEPLOY_SERVER_IP "cd /var/www/my-app && npm install --production && pm2 restart all"
    
  only:
    - main
```

### Key Elements of the Pipeline:

- **Build Job**:
  - Builds the React frontend.
  - Copies the built React files (`build/`) into the backend folder.
  - Bundles the backend, ready for deployment.

- **Artifacts**:
  - The frontend `build` directory is saved as an artifact, making it available for the deploy job.

- **Deploy Job**:
  - Uses SSH key authentication (configured using `SSH_PRIVATE_KEY` in GitLab CI/CD settings).
  - Deploys the app to a timestamped directory on the server.
  - Creates a symbolic link to point to the latest version (`/var/www/my-app`).
  - Uses `PM2` to restart the Node.js backend server.

### 5. **Adding Rollback Functionality**

For rollback functionality, you will deploy the app to timestamped directories and use symbolic links to point to the current version. If something goes wrong, you can point the symlink back to the previous version.

#### **Steps for Rollback**:

1. **Deploy to Timestamped Directories**:
   Each deployment creates a directory named with the current timestamp, such as `/var/www/my-app-20241022`.

2. **Symbolic Link (`ln -sfn`)**:
   The symbolic link `/var/www/my-app` always points to the latest deployment. During each deployment, the symlink is updated to point to the new directory.

3. **Manual Rollback**:
   If the new deployment fails, you can point the symbolic link back to the previous deployment by SSHing into the server and running:
   ```bash
   ln -sfn /var/www/my-app-previous-timestamp /var/www/my-app
   pm2 restart all
   ```

4. **Automated Rollback Job (Optional)**:
   You can create a separate rollback job in GitLab for manual rollback:
   ```yaml
   rollback:
     stage: deploy
     script:
       - ssh $DEPLOY_USER@$DEPLOY_SERVER_IP "ln -sfn /var/www/my-app-$ROLLBACK_TIMESTAMP /var/www/my-app"
       - ssh $DEPLOY_USER@$DEPLOY_SERVER_IP "pm2 restart all"
     when:


   😊🥲😊🥲🥰🙂🤪🥲🤩🤩🥲😊🥲🥰🥲🤪🥲🤩🥲🤪🥲🤪🥲🤩🤩🥲🤩🥲🤩🥲🤩🥲🤩🥲🤪🙂🥰🙂🤪🙂🤩🙂🤩🙂🤪🙂🥰🙂🥰🙂🤪🙂🥰🙂🥰🙂🤪🙂🤩🙂🥰🙂🤪🙂🤪🙂

   🔐😔🔐😔😉😔😉😅🥰😄🥰🥰😄🥰😄🥰😄🤩😄🤩😆🤩😆🤩😆🥰😙🤪😙🙂😙🤪😙🥰😙🤩🤩😙🤩😙😆🥰😆🤪🤪😆🤩😆😆😉😊😉😚☺️😊😉😊😉😊🥰😊🥰😊🤩🤩😆🥰



Creating and deploying a full-stack React client and server application via a CI/CD pipeline using GitLab CI, with rollback functionality, SSH key authentication, and deployment over Apache involves several steps. Below is a detailed guide:

### Prerequisites

1. **GitLab Repository**: Ensure your code is hosted in a GitLab repository.
2. **Apache Server**: An Apache server must be set up on the target machine.
3. **Node.js and npm**: Install Node.js and npm on the server to run your application.
4. **SSH Access**: Ensure you can access your server via SSH with the necessary permissions.
5. **GitLab Runner**: Make sure you have a GitLab Runner set up.

### Step 1: Project Structure

Ensure your project has a structure similar to this:

```
/my-app
  ├── /client      # React frontend
  ├── /server      # Express backend
  ├── .gitlab-ci.yml
  └── README.md
```

### Step 2: Configure SSH Key Authentication

1. **Generate SSH Key**: On your local machine, generate an SSH key pair if you haven't already:

   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

   Follow the prompts to save the key (default is `~/.ssh/id_rsa`).

2. **Add Public Key to Server**: Copy the public key (`~/.ssh/id_rsa.pub`) to the server's `~/.ssh/authorized_keys`.

3. **Add Private Key to GitLab**: In your GitLab repository, go to **Settings > CI / CD > Variables**, and add a new variable:

   - **Key**: `SSH_PRIVATE_KEY`
   - **Value**: Paste your private key contents.

   Ensure to mask the variable.

### Step 3: GitLab CI/CD Configuration

Create a `.gitlab-ci.yml` file in the root of your repository with the following content:

```yaml
image: node:14

stages:
  - build
  - deploy
  - rollback

variables:
  APP_NAME: "my-app"
  DEPLOY_PATH: "/var/www/html/my-app" # Change this to your Apache server's path

before_script:
  - apt-get update && apt-get install -y openssh-client
  - mkdir -p ~/.ssh
  - echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
  - chmod 600 ~/.ssh/id_rsa
  - ssh-keyscan -H your-server-ip >> ~/.ssh/known_hosts

build_client:
  stage: build
  script:
    - cd client
    - npm install
    - npm run build
  artifacts:
    paths:
      - client/build

build_server:
  stage: build
  script:
    - cd server
    - npm install
  artifacts:
    paths:
      - server

deploy:
  stage: deploy
  script:
    - ssh user@your-server-ip "mkdir -p $DEPLOY_PATH"
    - scp -r client/build user@your-server-ip:$DEPLOY_PATH/client
    - scp -r server user@your-server-ip:$DEPLOY_PATH/server
    - ssh user@your-server-ip "cd $DEPLOY_PATH && npm install --production && pm2 restart all" # or your preferred process manager
  only:
    - main

rollback:
  stage: rollback
  script:
    - ssh user@your-server-ip "cd $DEPLOY_PATH && git checkout HEAD~1"
    - ssh user@your-server-ip "cd $DEPLOY_PATH && npm install --production && pm2 restart all"
  when: manual
  only:
    - main
```

### Explanation of the CI/CD Pipeline

- **Stages**: Defines three stages: build, deploy, and rollback.
- **Variables**: Sets environment variables for the app name and deployment path.
- **before_script**: Installs SSH client, configures SSH key, and adds server to known hosts.
- **build_client**: Builds the React client and saves the output in artifacts.
- **build_server**: Installs server dependencies and saves them as artifacts.
- **deploy**: Deploys the built React app and server to the specified Apache path.
- **rollback**: Allows manual rollback to the previous deployment.

### Step 4: Configure Apache Server

Make sure your Apache server is configured to serve your React app and handle API requests. For example, you might need to update the `httpd.conf` or a virtual host configuration:

```apache
<VirtualHost *:80>
    DocumentRoot "/var/www/html/my-app/client"
    <Directory "/var/www/html/my-app/client">
        AllowOverride All
        Require all granted
    </Directory>

    ProxyPass /api http://localhost:5000/api
    ProxyPassReverse /api http://localhost:5000/api
</VirtualHost>
```

### Step 5: Commit and Push Changes

Once you have everything set up, commit and push your changes to the main branch. This should trigger the pipeline, and your application should be deployed automatically.

### Step 6: Rollback (if needed)

If something goes wrong, you can manually trigger the rollback stage from the GitLab CI/CD interface. This will revert to the previous version of your application.

### Conclusion

With this setup, you have a CI/CD pipeline for deploying a full-stack React application using GitLab CI. This process includes rollback functionality, SSH key authentication, and deployment over Apache, ensuring a smooth deployment experience. Make sure to adjust paths and configurations according to your specific environment.




   


