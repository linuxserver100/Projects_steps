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
  cp -r /path/to/react-app/build /path/to/backend/build
  
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
🥲😚😜😚🥹🤪🥭😆🤪😜🤪🥹🤪😜🤪🥲☺️🤩☺️🥹☺️🥹☺️🥹☺️🤩☺️😜☺️😜☺️🥲🤩😘😜😘😜😘🥲😘😜😘🤩😚🥲🥲😚🤩😚😜😚😜l🥰😅😅🤣🤩🤣😅🤣😅🤣😅😍🤣😄🤣😄🤣😍🤣😍🤣😍😅🤣😍🤣😍🤣😍🤣😅🤣😂😂🤣😂🙂‍↕️😂🙂‍↕️😔🙂‍↕️😔🙂‍↕️😔🙂‍↕️😔😅🙂‍↕️😅🙂‍↕️😅🙂‍↕️😅🙂‍↕️😅🙂‍↕️😅😅🙂‍↕️😅🙂‍↕️😅🙂‍↕️😅🙂‍↕️😅🙂‍↕️😅😍🙂‍↕️😅🙂‍↕️😔🙂‍↕️😅🙂‍↕️😅🙂‍↕️


To deploy a full-stack React application with a backend using GitLab CI/CD pipelines, and ensure continuous deployment, rollback functionality, SSH key authentication, and Apache as the web server for both frontend and backend, follow these detailed steps:

### Prerequisites:
1. **GitLab repository**: Store your full-stack application (React frontend and backend).
2. **SSH Access to the server**: You'll need SSH access to your server where Apache is running.
3. **Apache installed**: Ensure that Apache is installed on your server.
4. **Server configuration**: Set up SSH key authentication and ensure your server is ready for deployment.
5. **Domain or IP**: You need a domain name or server IP for your application.

---

### 1. **GitLab SSH Key Setup**
You'll need to set up SSH key authentication between your GitLab CI pipeline and your remote server.

1. **Generate SSH Key Pair** on the machine where the pipeline will run (or you can do it on your local machine):
   ```bash
   ssh-keygen -t rsa -b 4096 -C "gitlab-ci-deploy"
   ```
2. **Add Public Key to Remote Server**:
   Copy the public key to the remote server where the application will be deployed:
   ```bash
   ssh-copy-id user@your-server-ip
   ```
3. **Add Private Key to GitLab**:
   - Go to GitLab -> Your Project -> Settings -> CI/CD -> Variables.
   - Add a new variable `SSH_PRIVATE_KEY` and paste the private key (`~/.ssh/id_rsa`) here.
   - Make sure to mask this variable.

### 2. **Configure Apache for Deployment**
For the frontend and backend, you will create separate Apache configurations.

- **Frontend**: React app
- **Backend**: Node.js/Express, Flask, or other backend frameworks

#### 2.1 Apache Configuration for React Frontend
Assume your React app will be deployed at `/var/www/frontend`.

1. Create a new Apache configuration file, e.g., `/etc/apache2/sites-available/frontend.conf`:
   ```apache
   <VirtualHost *:80>
       ServerName frontend.example.com
       DocumentRoot /var/www/frontend

       <Directory /var/www/frontend>
           Options Indexes FollowSymLinks
           AllowOverride All
           Require all granted
       </Directory>

       ErrorLog ${APACHE_LOG_DIR}/frontend-error.log
       CustomLog ${APACHE_LOG_DIR}/frontend-access.log combined
   </VirtualHost>
   ```

2. Enable the site:
   ```bash
   sudo a2ensite frontend.conf
   sudo systemctl reload apache2
   ```

#### 2.2 Apache Configuration for Backend
Assume your backend will be deployed at `/var/www/backend`.

1. Create a new Apache configuration file, e.g., `/etc/apache2/sites-available/backend.conf`:
   ```apache
   <VirtualHost *:80>
       ServerName backend.example.com
       ProxyPreserveHost On
       ProxyPass / http://localhost:5000/
       ProxyPassReverse / http://localhost:5000/

       ErrorLog ${APACHE_LOG_DIR}/backend-error.log
       CustomLog ${APACHE_LOG_DIR}/backend-access.log combined
   </VirtualHost>
   ```

2. Enable the proxy modules and the backend site:
   ```bash
   sudo a2enmod proxy
   sudo a2enmod proxy_http
   sudo a2ensite backend.conf
   sudo systemctl reload apache2
   ```

### 3. **GitLab CI/CD Pipeline Setup**

In your GitLab project, create a `.gitlab-ci.yml` file at the root of the repository. This will contain the necessary steps for deployment and rollback.

#### 3.1 Define Stages
Define the stages like `build`, `deploy`, and `rollback`.

```yaml
stages:
  - build
  - deploy
  - rollback
```

#### 3.2 Build Frontend and Backend
The build stage will compile the frontend and prepare the backend for deployment.

```yaml
build-frontend:
  stage: build
  script:
    - cd frontend
    - npm install
    - npm run build
  artifacts:
    paths:
      - frontend/build/

build-backend:
  stage: build
  script:
    - cd backend
    - npm install
    - npm run build
  artifacts:
    paths:
      - backend/
```

#### 3.3 Deploy Frontend and Backend
The deploy stage will use SSH to copy the built files to the server.

```yaml
deploy-frontend:
  stage: deploy
  script:
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - ssh -o StrictHostKeyChecking=no user@your-server-ip "rm -rf /var/www/frontend/*"
    - scp -r frontend/build/* user@your-server-ip:/var/www/frontend/

deploy-backend:
  stage: deploy
  script:
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - ssh -o StrictHostKeyChecking=no user@your-server-ip "rm -rf /var/www/backend/*"
    - scp -r backend/* user@your-server-ip:/var/www/backend/
    - ssh user@your-server-ip "systemctl restart backend.service"
```

#### 3.4 Rollback
In case of a failed deployment, use the rollback stage to revert to a previous version.

```yaml
rollback:
  stage: rollback
  script:
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - ssh user@your-server-ip "rm -rf /var/www/frontend/* && cp -r /var/www/frontend_backup/* /var/www/frontend/"
    - ssh user@your-server-ip "rm -rf /var/www/backend/* && cp -r /var/www/backend_backup/* /var/www/backend/"
    - ssh user@your-server-ip "systemctl restart backend.service"
```

You may want to adjust your rollback logic based on how you back up your files (e.g., using versioned folders).

### 4. **Add Deployment Conditions**
You can add conditions to deploy only on specific branches (e.g., `main` or `production`):

```yaml
deploy-frontend:
  stage: deploy
  only:
    - main
```

### 5. **Backup and Rollback Strategies**
Make sure to include a strategy for keeping backups of previous versions of the frontend and backend in case a rollback is needed. For example:

1. **Backup Before Deployment**:
   Add a step in the deployment process to back up the existing version before deploying the new one:
   ```bash
   ssh user@your-server-ip "cp -r /var/www/frontend /var/www/frontend_backup"
   ssh user@your-server-ip "cp -r /var/www/backend /var/www/backend_backup"
   ```

2. **Rollback in GitLab**:
   Trigger a rollback if the deployment fails. You can automatically check for deployment success and proceed to rollback on failure by using GitLab’s `allow_failure` and `when: on_failure` options.

---

### Final Thoughts:
This setup provides an end-to-end solution for deploying your full-stack React app with GitLab CI/CD pipelines, including continuous deployment, rollback functionality, and Apache server configuration. The process ensures that both the frontend and backend are properly deployed and managed on the server using SSH authentication and GitLab CI variables.
  


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



😀😄😁☺️😆😆🤣🤣😄😆🤣😙🤩😙🤩😘🤩😗🤩😁😌😀😀😌😀😌😀😌😚😚😌😀😌🙂🤩🥹🤩😚😌😚😌😘😌😙😌🥹😌🥹😚😌🥰😌😚🙂‍↕️🥰🙂‍↕️🥰🥴😚🙂‍↕️😊☺️🥴😘😄🤩🥹🤩🤩🥲🤩😙🤩🥹😌😙😚😚😚☺️😙☺️😚☺️😙☺️😚☺️😚😙☺️😙☺️😚☺️😙☺️😙😙☺️😙☺️😙☺️😙☺️😙☺️😙😙☺️😌🥹😌🥲
Creating and deploying a full-stack MERN (MongoDB, Express, React, Node.js) authentication application with password reset functionality, along with setting up a CI/CD pipeline using GitLab CI, involves multiple steps. Below, I’ll break down the entire process into detailed sections, covering backend and frontend development, as well as the CI/CD pipeline setup with rollback functionality.

### Overview

1. **Set up the Project Structure**
2. **Backend Development**
   - Create a Node.js and Express backend
   - Implement MongoDB for user storage
   - Set up authentication with JWT
   - Create password reset functionality
3. **Frontend Development**
   - Create a React frontend
   - Implement registration, login, and reset password pages
4. **Set up CI/CD with GitLab**
   - Configure GitLab CI/CD pipeline
   - Set up SSH key authentication
   - Implement rollback functionality
5. **Deploy the Application**
   - Deploy the backend on Apache
   - Serve the frontend using Apache

### Step 1: Set up the Project Structure

Create a directory for your project and set up two folders: one for the backend and one for the frontend.

```bash
mkdir mern-auth-app
cd mern-auth-app
mkdir backend frontend
```

### Step 2: Backend Development

#### 2.1 Create a Node.js and Express Backend

1. **Navigate to the backend folder**:

   ```bash
   cd backend
   ```

2. **Initialize a Node.js project**:

   ```bash
   npm init -y
   ```

3. **Install necessary packages**:

   ```bash
   npm install express mongoose bcryptjs jsonwebtoken dotenv nodemailer cors
   ```

4. **Create the following file structure**:

   ```plaintext
   backend/
   ├── config/
   │   └── db.js
   ├── controllers/
   │   └── authController.js
   ├── models/
   │   └── User.js
   ├── routes/
   │   └── authRoutes.js
   ├── middleware/
   │   └── authMiddleware.js
   ├── .env
   ├── server.js
   └── package.json
   ```

#### 2.2 Implement MongoDB for User Storage

- **Connect to MongoDB** (`config/db.js`):

```javascript
const mongoose = require('mongoose');

const connectDB = async () => {
    try {
        await mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true });
        console.log('MongoDB Connected');
    } catch (error) {
        console.error(error);
        process.exit(1);
    }
};

module.exports = connectDB;
```

- **User Model** (`models/User.js`):

```javascript
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
    username: { type: String, required: true, unique: true },
    email: { type: String, required: true, unique: true },
    password: { type: String, required: true },
});

module.exports = mongoose.model('User', userSchema);
```

#### 2.3 Set up Authentication with JWT

- **Authentication Controller** (`controllers/authController.js`):

```javascript
const User = require('../models/User');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

// Register user
exports.register = async (req, res) => {
    const { username, email, password } = req.body;

    try {
        const hashedPassword = await bcrypt.hash(password, 10);
        const user = await User.create({ username, email, password: hashedPassword });
        res.status(201).json({ message: 'User registered', user });
    } catch (error) {
        res.status(400).json({ message: error.message });
    }
};

// Login user
exports.login = async (req, res) => {
    const { email, password } = req.body;

    try {
        const user = await User.findOne({ email });
        if (!user || !(await bcrypt.compare(password, user.password))) {
            return res.status(400).json({ message: 'Invalid credentials' });
        }

        const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET, { expiresIn: '1h' });
        res.json({ token });
    } catch (error) {
        res.status(400).json({ message: error.message });
    }
};

// Reset Password
exports.resetPassword = async (req, res) => {
    const { email, newPassword } = req.body;
    const hashedPassword = await bcrypt.hash(newPassword, 10);

    try {
        await User.updateOne({ email }, { password: hashedPassword });
        res.json({ message: 'Password reset successful' });
    } catch (error) {
        res.status(400).json({ message: error.message });
    }
};
```

#### 2.4 Create Routes

- **Routes** (`routes/authRoutes.js`):

```javascript
const express = require('express');
const { register, login, resetPassword } = require('../controllers/authController');

const router = express.Router();

router.post('/register', register);
router.post('/login', login);
router.post('/reset-password', resetPassword);

module.exports = router;
```

#### 2.5 Create the Server

- **Server setup** (`server.js`):

```javascript
const express = require('express');
const dotenv = require('dotenv');
const cors = require('cors');
const connectDB = require('./config/db');
const authRoutes = require('./routes/authRoutes');

dotenv.config();
connectDB();

const app = express();
app.use(cors());
app.use(express.json());
app.use('/api/auth', authRoutes);

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

#### 2.6 Environment Variables

- Create a `.env` file in the backend folder:

```plaintext
MONGO_URI=mongodb://your_mongo_uri
JWT_SECRET=your_jwt_secret
```

### Step 3: Frontend Development

#### 3.1 Create a React Frontend

1. **Navigate to the frontend folder**:

   ```bash
   cd ../frontend
   ```

2. **Create a React app**:

   ```bash
   npx create-react-app .
   ```

3. **Install Axios for API calls**:

   ```bash
   npm install axios react-router-dom
   ```

#### 3.2 Create Pages

- **Create the following file structure**:

```plaintext
frontend/
├── src/
│   ├── components/
│   │   ├── Login.js
│   │   ├── Register.js
│   │   ├── ResetPassword.js
│   ├── App.js
│   └── index.js
```

#### 3.3 Implement Authentication Pages

- **Login Component** (`components/Login.js`):

```javascript
import React, { useState } from 'react';
import axios from 'axios';

const Login = () => {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');

    const handleLogin = async (e) => {
        e.preventDefault();
        try {
            const response = await axios.post('http://localhost:5000/api/auth/login', { email, password });
            localStorage.setItem('token', response.data.token);
            alert('Login successful!');
        } catch (error) {
            alert('Login failed: ' + error.response.data.message);
        }
    };

    return (
        <form onSubmit={handleLogin}>
            <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" required />
            <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Password" required />
            <button type="submit">Login</button>
        </form>
    );
};

export default Login;
```

- **Register Component** (`components/Register.js`):

```javascript
import React, { useState } from 'react';
import axios from 'axios';

const Register = () => {
    const [username, setUsername] = useState('');
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');

    const handleRegister = async (e) => {
        e.preventDefault();
        try {
            await axios.post('http://localhost:5000/api/auth/register', { username, email, password });
            alert('Registration successful!');
        } catch (error) {
            alert('Registration failed: ' + error.response.data.message);
        }
    };

    return (
        <form onSubmit={handleRegister}>
            <input type="text" value={username} onChange={(e) => setUsername(e.target.value)} placeholder="Username" required />
            <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" required />
            <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Password" required />
            <button type="submit">Register</button>
        </form>
    );
};

export default Register;
```

- **Reset Password Component** (`components/ResetPassword.js`):

```javascript
import React, { useState } from 'react';
import axios from 'axios';

const ResetPassword = () => {
    const [email, setEmail] = useState('');
    const [newPassword, setNewPassword] = useState('');

    const handleReset =

 async (e) => {
        e.preventDefault();
        try {
            await axios.post('http://localhost:5000/api/auth/reset-password', { email, newPassword });
            alert('Password reset successful!');
        } catch (error) {
            alert('Password reset failed: ' + error.response.data.message);
        }
    };

    return (
        <form onSubmit={handleReset}>
            <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" required />
            <input type="password" value={newPassword} onChange={(e) => setNewPassword(e.target.value)} placeholder="New Password" required />
            <button type="submit">Reset Password</button>
        </form>
    );
};

export default ResetPassword;
```

#### 3.4 Configure Routing

- **Update `App.js`**:

```javascript
import React from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import Login from './components/Login';
import Register from './components/Register';
import ResetPassword from './components/ResetPassword';

const App = () => {
    return (
        <Router>
            <Switch>
                <Route path="/login" component={Login} />
                <Route path="/register" component={Register} />
                <Route path="/reset-password" component={ResetPassword} />
            </Switch>
        </Router>
    );
};

export default App;
```

### Step 4: Set up CI/CD with GitLab

#### 4.1 GitLab CI/CD Pipeline Configuration

1. **Create a `.gitlab-ci.yml` file in the root of your project**:

```yaml
stages:
  - build
  - test
  - deploy

build_backend:
  stage: build
  script:
    - cd backend
    - npm install
    - npm run build
  artifacts:
    paths:
      - backend/build/

build_frontend:
  stage: build
  script:
    - cd frontend
    - npm install
    - npm run build
  artifacts:
    paths:
      - frontend/build/

deploy_backend:
  stage: deploy
  script:
    - ssh user@your-server "cd /path/to/your/backend && git pull && npm install && pm2 restart server"
  only:
    - master

deploy_frontend:
  stage: deploy
  script:
    - ssh user@your-server "cd /path/to/your/frontend && git pull && npm install && pm2 restart frontend"
  only:
    - master
```

#### 4.2 SSH Key Authentication

1. **Generate an SSH key** on your local machine (if you don't have one):

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

2. **Copy the public key**:

```bash
cat ~/.ssh/id_rsa.pub
```

3. **Add the public key to your server's `~/.ssh/authorized_keys`**.

4. **Add the private key to GitLab CI/CD**:
   - Go to your project in GitLab.
   - Navigate to **Settings > CI/CD > Variables**.
   - Add a new variable called `SSH_PRIVATE_KEY` and paste your private key.

5. **Modify the `.gitlab-ci.yml` to use the SSH key**:

```yaml
before_script:
  - eval $(ssh-agent -s)
  - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
  - mkdir -p ~/.ssh
  - chmod 700 ~/.ssh
  - ssh-keyscan -H your-server >> ~/.ssh/known_hosts
  - chmod 644 ~/.ssh/known_hosts
```

#### 4.3 Implement Rollback Functionality

- You can create a rollback stage in the `.gitlab-ci.yml` that will allow you to revert to a previous deployment if the current one fails. Here’s how you might structure that:

```yaml
rollback:
  stage: deploy
  script:
    - ssh user@your-server "cd /path/to/your/backend && git checkout previous_commit_hash"
  when: manual
```

### Step 5: Deploy the Application

#### 5.1 Deploy the Backend on Apache

1. **Install Apache and Node.js on your server** if not already done.

2. **Set up a reverse proxy for your Node.js app**. Create an Apache configuration file (e.g., `backend.conf`):

```apache
<VirtualHost *:80>
    ServerName your_domain_or_IP

    ProxyRequests Off
    ProxyPass /api http://localhost:5000/api
    ProxyPassReverse /api http://localhost:5000/api

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

3. **Enable the proxy modules**:

```bash
sudo a2enmod proxy
sudo a2enmod proxy_http
```

4. **Restart Apache**:

```bash
sudo systemctl restart apache2
```

#### 5.2 Serve the Frontend Using Apache

1. **Build the React app**:

```bash
cd frontend
npm run build
```

2. **Copy the build files to the Apache server**:

```bash
scp -r build/* user@your-server:/var/www/html
```

3. **Create an Apache configuration file for the frontend**:

```apache
<VirtualHost *:80>
    ServerName your_domain_or_IP

    DocumentRoot /var/www/html

    <Directory /var/www/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

4. **Restart Apache**:

```bash
sudo systemctl restart apache2
```

### Conclusion

You now have a full-stack MERN authentication application with registration, login, and password reset functionality. The application is deployed using a CI/CD pipeline with GitLab CI, and you’ve implemented rollback functionality for deployments.

Make sure to secure your server, manage secrets properly, and consider further enhancements such as email verification and user roles based on your project requirements. If you have any specific areas you'd like to explore further, let me know!

😌🥹😌🥹😌🥲😌😊😌😊😌😊🥹😌😊😌🥹😌🥲😌🥲🙂‍↕️😊🤪😊😛🤪🥲🤪🥹🤪😊🤪🥲🤪🥹😝🤪🥲🤪🥹🤪🥹🤪🥹🤪🥹😌🥹😌😊🥰🤪🥰🤪😘🤪😊🤪😊🤪☺️😊🤪😊🤪😊🤪
Creating and deploying a full-stack MERN (MongoDB, Express, React, Node.js) application with authentication and a password reset functionality is a comprehensive task. To accomplish this, we'll detail the entire process step-by-step, covering everything from development to deployment using a CI/CD pipeline with GitLab. This guide will include rollback functionality, SSH key authentication, and deployment using Apache.

### **1. Project Setup**

#### **1.1. Backend Setup (Node.js + Express)**

1. **Initialize the Backend Project**
   ```bash
   mkdir mern-auth-backend
   cd mern-auth-backend
   npm init -y
   npm install express mongoose bcryptjs jsonwebtoken dotenv cors nodemailer
   ```

2. **Create Directory Structure**
   ```bash
   mkdir config controllers models routes middleware
   ```

3. **Setup Environment Variables**
   - Create a `.env` file for sensitive configurations.
   ```plaintext
   PORT=5000
   MONGO_URI=mongodb://<user>:<password>@localhost:27017/mydatabase
   JWT_SECRET=mysecretkey
   EMAIL_USER=myemail@example.com
   EMAIL_PASS=mypassword
   ```

4. **Create Basic Server Setup (`index.js`)**
   ```javascript
   const express = require('express');
   const mongoose = require('mongoose');
   const cors = require('cors');
   const authRoutes = require('./routes/auth');

   require('dotenv').config();

   const app = express();
   app.use(cors());
   app.use(express.json());

   mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true })
       .then(() => console.log("MongoDB connected"))
       .catch(err => console.error(err));

   app.use('/api/auth', authRoutes);

   const PORT = process.env.PORT || 5000;
   app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
   ```

5. **Create User Model (`models/User.js`)**
   ```javascript
   const mongoose = require('mongoose');
   const userSchema = new mongoose.Schema({
       username: { type: String, required: true, unique: true },
       password: { type: String, required: true },
       email: { type: String, required: true, unique: true },
   });
   module.exports = mongoose.model('User', userSchema);
   ```

6. **Setup Authentication Routes (`routes/auth.js`)**
   ```javascript
   const express = require('express');
   const bcrypt = require('bcryptjs');
   const jwt = require('jsonwebtoken');
   const User = require('../models/User');

   const router = express.Router();

   // Register route
   router.post('/register', async (req, res) => {
       const { username, password, email } = req.body;
       const hashedPassword = await bcrypt.hash(password, 10);
       const newUser = new User({ username, password: hashedPassword, email });
       await newUser.save();
       res.status(201).json({ message: 'User registered' });
   });

   // Login route
   router.post('/login', async (req, res) => {
       const { username, password } = req.body;
       const user = await User.findOne({ username });
       if (!user) return res.status(404).send('User not found');
       const isMatch = await bcrypt.compare(password, user.password);
       if (!isMatch) return res.status(400).send('Invalid credentials');
       const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET);
       res.json({ token });
   });

   // Password reset route
   router.post('/reset-password', async (req, res) => {
       const { email } = req.body;
       const user = await User.findOne({ email });
       if (!user) return res.status(404).send('User not found');

       // Send email logic using nodemailer...
       // Generate reset link and send via nodemailer

       res.send('Password reset link sent');
   });

   module.exports = router;
   ```

#### **1.2. Frontend Setup (React)**

1. **Create the React Application**
   ```bash
   npx create-react-app mern-auth-frontend
   cd mern-auth-frontend
   npm install axios react-router-dom
   ```

2. **Setup Directory Structure**
   ```bash
   mkdir src/components src/pages src/services
   ```

3. **Setup Axios for API Requests**
   - Create `src/services/api.js`:
   ```javascript
   import axios from 'axios';

   const api = axios.create({
       baseURL: 'http://localhost:5000/api/auth',
   });

   export default api;
   ```

4. **Create Authentication Pages and Components**
   - For example, `LoginPage`, `RegisterPage`, `ResetPasswordPage` in `src/pages`.

5. **Routing Setup (`src/App.js`)**
   ```javascript
   import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
   import LoginPage from './pages/LoginPage';
   import RegisterPage from './pages/RegisterPage';
   import ResetPasswordPage from './pages/ResetPasswordPage';

   function App() {
       return (
           <Router>
               <Switch>
                   <Route path="/login" component={LoginPage} />
                   <Route path="/register" component={RegisterPage} />
                   <Route path="/reset-password" component={ResetPasswordPage} />
               </Switch>
           </Router>
       );
   }

   export default App;
   ```

### **2. CI/CD Pipeline Setup with GitLab CI**

#### **2.1. Project Structure in GitLab**

1. **Create a GitLab Repository**
   - Push both frontend and backend projects into separate folders in the same repository.

2. **Create `.gitlab-ci.yml` File**
   - This file will define your CI/CD pipeline.
   ```yaml
   stages:
     - build
     - deploy

   build_backend:
     stage: build
     image: node:14
     script:
       - cd mern-auth-backend
       - npm install
       - npm run build
     artifacts:
       paths:
         - mern-auth-backend/build

   build_frontend:
     stage: build
     image: node:14
     script:
       - cd mern-auth-frontend
       - npm install
       - npm run build
     artifacts:
       paths:
         - mern-auth-frontend/build

   deploy_backend:
     stage: deploy
     image: alpine:latest
     script:
       - apk add --no-cache openssh-client
       - ssh -i $SSH_PRIVATE_KEY -o StrictHostKeyChecking=no user@your_server "cd /path/to/your/backend && git pull origin main && npm install && pm2 restart your-app-name"
     only:
       - main

   deploy_frontend:
     stage: deploy
     image: alpine:latest
     script:
       - apk add --no-cache openssh-client
       - ssh -i $SSH_PRIVATE_KEY -o StrictHostKeyChecking=no user@your_server "cd /path/to/your/frontend && git pull origin main && npm install && npm run build"
       - ssh -i $SSH_PRIVATE_KEY -o StrictHostKeyChecking=no user@your_server "cp -r /path/to/your/frontend/build/* /var/www/html/"
     only:
       - main
   ```

#### **2.2. Configure SSH Key Authentication**

1. **Generate SSH Key Pair**
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

2. **Add Public Key to Server**
   ```bash
   ssh-copy-id user@your_server
   ```

3. **Add Private Key to GitLab**
   - Go to your project settings in GitLab > CI/CD > Variables, and add a variable named `SSH_PRIVATE_KEY` with your private key.

### **3. Rollback Functionality**

For rollback functionality, you can keep track of the last deployment. If a deployment fails, you can revert to the last stable version.

1. **Update `.gitlab-ci.yml` to Create a Backup**
   ```yaml
   backup_backend:
     stage: deploy
     script:
       - ssh -i $SSH_PRIVATE_KEY -o StrictHostKeyChecking=no user@your_server "cp -r /path/to/your/backend /path/to/your/backup/backend_backup_$(date +%Y%m%d%H%M%S)"
     only:
       - main

   rollback_backend:
     stage: deploy
     when: manual
     script:
       - ssh -i $SSH_PRIVATE_KEY -o StrictHostKeyChecking=no user@your_server "cp -r /path/to/your/backup/backend_backup_LAST /path/to/your/backend"
     only:
       - main
   ```

### **4. Apache Configuration for Deployment**

1. **Install Apache**
   ```bash
   sudo apt update
   sudo apt install apache2
   ```

2. **Configure Apache for React App**
   - Create a new configuration file for your frontend.
   ```bash
   sudo nano /etc/apache2/sites-available/mern-auth-frontend.conf
   ```

   ```apache
   <VirtualHost *:80>
       ServerName your_domain.com
       DocumentRoot /var/www/html

       <Directory /var/www/html>
           AllowOverride All
       </Directory>

       ErrorLog ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```

3. **Enable Rewrite Module and Your Site**
   ```bash
   sudo a2

enmod rewrite
   sudo a2ensite mern-auth-frontend.conf
   sudo systemctl restart apache2
   ```

4. **Deploy Backend API**
   - Ensure the Node.js application runs using a process manager like PM2.
   ```bash
   npm install pm2 -g
   pm2 start mern-auth-backend/index.js --name your-app-name
   pm2 save
   ```

### **5. Testing the Application**

1. **Run the Application Locally**
   - Start the backend:
   ```bash
   cd mern-auth-backend
   node index.js
   ```

   - Start the frontend:
   ```bash
   cd mern-auth-frontend
   npm start
   ```

2. **Accessing the Application**
   - Open your browser and go to `http://localhost:3000` for the frontend and `http://localhost:5000/api/auth` for the backend API.

### **6. Conclusion**

You now have a full-stack MERN application with authentication, password reset functionality, and a CI/CD pipeline configured with GitLab. The application can be deployed using SSH with rollback capabilities through the CI/CD pipeline, ensuring you can quickly revert changes if needed.

Remember to test thoroughly and make adjustments based on your specific environment and needs!

   


