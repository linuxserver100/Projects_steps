😃Here's a detailed guide to setting up a continuous deployment pipeline using GitLab CI/CD for a React.js fullstack application (frontend and backend) on an Ubuntu server. This guide will cover creating a deployer user, setting up private key authentication, managing environment variables, configuring GitLab Runner, and implementing rollback functionality.

### Prerequisites

1. **GitLab Account**: Access to your GitLab project.
2. **Ubuntu Server**: An Ubuntu server for deployment.
3. **GitLab Runner**: Installed and registered on your server.
4. **SSH Access**: SSH access to your server.

### Step 1: Create a Deployer User

1. **SSH into your server**:
   ```bash
   ssh username@your_server_ip
   ```

2. **Create a new user for deployment**:
   ```bash
   sudo adduser deployer
   ```

3. **Optionally grant the user sudo privileges**:
   ```bash
   sudo usermod -aG sudo deployer
   ```

4. **Set up SSH for the deployer user**:
   ```bash
   sudo mkdir -p /home/deployer/.ssh
   sudo chmod 700 /home/deployer/.ssh
   ```

### Step 2: Generate SSH Keys

1. **On your local machine**, generate a new SSH key:
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```
   Save it as `id_rsa_deployer`.

2. **Copy the public key to the server**:
   ```bash
   ssh-copy-id deployer@your_server_ip
   ```

3. **Securely store the private key**.

### Step 3: Configure GitLab CI/CD Variables

1. **Go to your GitLab project**.
2. **Navigate to Settings > CI / CD > Variables** and add the following variables:
   - `DEPLOY_USER`: `deployer`
   - `DEPLOY_HOST`: `your_server_ip`
   - `FRONTEND_PATH`: `/path/to/frontend`
   - `BACKEND_PATH`: `/path/to/backend`
   - `SSH_PRIVATE_KEY`: Paste the content of your `id_rsa_deployer` private key.

### Step 4: Set Up GitLab CI/CD Configuration

Create a `.gitlab-ci.yml` file in the root of your repository:

```yaml
image: node:14

stages:
  - build
  - deploy
  - rollback

before_script:
  - apt-get update && apt-get install -y ssh
  - mkdir -p ~/.ssh
  - echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
  - chmod 600 ~/.ssh/id_rsa
  - ssh-keyscan -H $DEPLOY_HOST >> ~/.ssh/known_hosts

build_frontend:
  stage: build
  script:
    - cd frontend
    - npm install
    - npm run build
  artifacts:
    paths:
      - frontend/build/

build_backend:
  stage: build
  script:
    - cd backend
    - npm install
    - npm run build
  artifacts:
    paths:
      - backend/build/

deploy:
  stage: deploy
  script:
    - ssh $DEPLOY_USER@$DEPLOY_HOST "mkdir -p $FRONTEND_PATH/current && mkdir -p $BACKEND_PATH/current"
    - ssh $DEPLOY_USER@$DEPLOY_HOST "mv $FRONTEND_PATH/current $FRONTEND_PATH/previous || true"
    - scp -r frontend/build/* $DEPLOY_USER@$DEPLOY_HOST:$FRONTEND_PATH/current/
    - ssh $DEPLOY_USER@$DEPLOY_HOST "mv $BACKEND_PATH/current $BACKEND_PATH/previous || true"
    - scp -r backend/build/* $DEPLOY_USER@$DEPLOY_HOST:$BACKEND_PATH/current/
  environment:
    name: production
    url: http://your_server_ip

rollback:
  stage: rollback
  script:
    - ssh $DEPLOY_USER@$DEPLOY_HOST "mv $FRONTEND_PATH/current $FRONTEND_PATH/previous && mv $FRONTEND_PATH/previous $FRONTEND_PATH/current"
    - ssh $DEPLOY_USER@$DEPLOY_HOST "mv $BACKEND_PATH/current $BACKEND_PATH/previous && mv $BACKEND_PATH/previous $BACKEND_PATH/current"
```

### Step 5: Implement Rollback Functionality

1. **Create directories for the application on the server**:
   ```bash
   sudo mkdir -p /path/to/frontend/current
   sudo mkdir -p /path/to/frontend/previous
   sudo mkdir -p /path/to/backend/current
   sudo mkdir -p /path/to/backend/previous
   ```

2. **Ensure the deployer user has the necessary permissions**:
   ```bash
   sudo chown -R deployer:deployer /path/to/frontend /path/to/backend
   ```

### Step 6: Install and Register GitLab Runner

1. **On your server, install GitLab Runner**:
   ```bash
   sudo apt-get install gitlab-runner
   ```

2. **Register the runner**:
   ```bash
   sudo gitlab-runner register
   ```
   Follow the prompts to enter your GitLab instance URL, registration token, description, tags, and executor (e.g., `shell`).

### Step 7: Test the Pipeline

1. **Push your changes to GitLab**. This should trigger the CI/CD pipeline.
2. **Monitor the pipeline** in GitLab under CI/CD > Pipelines to ensure it runs without errors.
3. **Check your application** on the server to confirm the deployment.

### Conclusion

You now have a fully functional CI/CD pipeline set up for your React.js fullstack project using GitLab CI/CD on an Ubuntu server. Customize the `.gitlab-ci.yml` file according to your project’s needs, and follow security best practices for managing SSH keys and access.


😃😆😘🔑😘🔑😭🔑🥴🔑😉😍👇🔑😊😉🔑😭🔑😭🔑😘🔑😘🔑😃🔑😃🔑😃🔑😃🔑😘🔑😘🔑😃🔑😘🔑🔑😂🥭🔑😂🔑😭🔑😭🔗😘🔑😂🔗😂🔗😭🔗😭😭🔗😚🔗😂🔗😭🔑😂🔑😉👇😌😌😌👇😍😭😍😭😍😘😍😂😍😂😍😂😄🥭😄🥭😭🔑😘🔑😘🔑😭🔑😭😄😂😄🥭😄😂😄😂
Here’s a comprehensive guide to setting up a continuous deployment pipeline using GitLab CI/CD for a React.js fullstack project on an Ubuntu server. This guide includes private key (.pem) authentication, variable settings, creating a deployer user, setting up GitLab Runner, and implementing rollback functionality.

### Step 1: Prepare Your Ubuntu Server

1. **Update the Server:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Install Node.js and npm:**
   Install Node.js using NodeSource:
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
   sudo apt install -y nodejs
   ```

3. **Install Git:**
   ```bash
   sudo apt install -y git
   ```

4. **Install PM2 (Process Manager):**
   ```bash
   sudo npm install -g pm2
   ```

### Step 2: Create a Deployer User

1. **Create the User:**
   ```bash
   sudo adduser deployer
   ```

2. **Grant Necessary Permissions:**
   (Optional) Add the deployer user to the sudo group:
   ```bash
   sudo usermod -aG sudo deployer
   ```

### Step 3: Set Up SSH Keys

1. **Generate a Private Key:**
   If you don’t have a private key, create one on your local machine:
   ```bash
   ssh-keygen -t rsa -b 4096 -f ~/.ssh/privatekey.pem -C "your_email@example.com"
   ```

2. **Copy the Public Key to the Server:**
   Log into the server as the deployer user:
   ```bash
   sudo su - deployer
   mkdir -p ~/.ssh
   echo "your-public-key-content" >> ~/.ssh/authorized_keys
   chmod 600 ~/.ssh/authorized_keys
   ```

3. **Test SSH Access:**
   From your local machine, verify:
   ```bash
   ssh -i ~/.ssh/privatekey.pem deployer@your_server_ip
   ```

### Step 4: Set Up GitLab CI/CD Variables

1. **Navigate to Your GitLab Project:**
   Go to `Settings > CI/CD > Variables`.

2. **Add Variables:**
   - **`SSH_PRIVATE_KEY`**: Paste the contents of your private key (e.g., `cat ~/.ssh/privatekey.pem`).
   - **`DEPLOY_USER`**: Set this to `deployer`.
   - **`SERVER_IP`**: Your server's IP address (e.g., `192.168.1.10`).
   - **`APP_DIR`**: Directory for your application (e.g., `/var/www/myapp`).

### Step 5: Install and Configure GitLab Runner

1. **Install GitLab Runner:**
   ```bash
   sudo apt install -y curl
   curl -L https://packages.gitlab.com/install/repositories/gitlab/gitlab-runner/script.deb.sh | sudo bash
   sudo apt install gitlab-runner
   ```

2. **Register the Runner:**
   ```bash
   sudo gitlab-runner register
   ```
   - Enter your GitLab instance URL.
   - Use the token from your project’s settings.
   - Choose `shell` as the executor.

### Step 6: Create `.gitlab-ci.yml`

Create a `.gitlab-ci.yml` file in your project’s root directory:

```yaml
stages:
  - build
  - deploy
  - rollback

build:
  stage: build
  script:
    - npm install
    - npm run build

deploy:
  stage: deploy
  script:
    - echo "$SSH_PRIVATE_KEY" > privatekey.pem
    - chmod 600 privatekey.pem
    - scp -o StrictHostKeyChecking=no -i privatekey.pem -r ./build/* $DEPLOY_USER@$SERVER_IP:$APP_DIR/
    - ssh -o StrictHostKeyChecking=no -i privatekey.pem $DEPLOY_USER@$SERVER_IP 'pm2 restart your-app-name'
  only:
    - main

rollback:
  stage: rollback
  script:
    - ssh -o StrictHostKeyChecking=no -i privatekey.pem $DEPLOY_USER@$SERVER_IP 'pm2 revert your-app-name'
  when: manual
```

### Step 7: Implement Rollback Functionality

Using PM2, you can easily revert to the previous version:
```bash
pm2 revert your-app-name
```
Make sure that your application is managed by PM2.

### Step 8: Test Your Pipeline

1. **Push Changes:**
   Commit and push your changes to the main branch.

2. **Monitor CI/CD Pipeline:**
   Go to your GitLab project and navigate to `CI/CD > Pipelines` to check the deployment process.

3. **Manual Rollback:**
   You can manually trigger the rollback from the pipeline interface as needed.

### Example Directory Structure

Here's an example of how your project directory might look:

```
my-react-app/
│
├── .gitlab-ci.yml
├── package.json
├── src/
│   └── App.js
└── public/
    └── index.html
```

### Conclusion

Following these steps will set up a continuous deployment pipeline for your React.js fullstack application using GitLab CI/CD. Ensure that you monitor your deployments and follow best security practices for your server.



🙂😉😚😘😍😭😃😃😃😉👇👇👇💙🙂🙏👇😂😊🙂‍↕️😌🙂‍↕️😗🥭😅😌🙂😌😆😍😆😭😉👇💙😊😌😅🥭😊😂👇😂👇😂😆💙😆😭😆🥴🚫👇💙🙂🙏😉🙏🥴🙏👇👇💙😀💙😊💙👇💙😉💙😉💙🥴💙
Here's a comprehensive guide for setting up a continuous deployment pipeline using GitLab CI/CD for a Node.js project on an Ubuntu server. This guide covers private key (.pem) authentication, variable settings, creating a deployer user, setting up GitLab Runner, and implementing rollback functionality.

### Step 1: Prepare Your Ubuntu Server

1. **Update the Server:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Install Node.js and npm:**
   Install Node.js from NodeSource:
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
   sudo apt install -y nodejs
   ```

3. **Install Git:**
   ```bash
   sudo apt install -y git
   ```

### Step 2: Create a Deployer User

1. **Create the User:**
   ```bash
   sudo adduser deployer
   ```

2. **Grant Necessary Permissions:**
   (Optional) If your deployment scripts require sudo access:
   ```bash
   sudo usermod -aG sudo deployer
   ```

### Step 3: Set Up SSH Keys

1. **Generate a Private Key:**
   If you don't have a private key, create one:
   ```bash
   ssh-keygen -t rsa -b 4096 -f ~/.ssh/privatekey.pem -C "your_email@example.com"
   ```
   You will have two files: `privatekey.pem` (private key) and `privatekey.pem.pub` (public key).

2. **Copy the Public Key to the Server:**
   Log into the server as the deployer user:
   ```bash
   sudo su - deployer
   mkdir -p ~/.ssh
   echo "your-public-key-content" >> ~/.ssh/authorized_keys
   chmod 600 ~/.ssh/authorized_keys
   ```

3. **Test SSH Access:**
   From your local machine, verify:
   ```bash
   ssh -i ~/.ssh/privatekey.pem deployer@your_server_ip
   ```

### Step 4: Set Up GitLab CI/CD Variables

1. **Navigate to Your GitLab Project:**
   Go to `Settings > CI/CD > Variables`.

2. **Add Variables:**
   - **`SSH_PRIVATE_KEY`**: Paste the contents of your private key (e.g., `cat ~/.ssh/privatekey.pem`).
   - **`DEPLOY_USER`**: Set this to `deployer`.
   - **`SERVER_IP`**: Your server's IP address (e.g., `192.168.1.10`).
   - **`APP_DIR`**: Directory for your application (e.g., `/var/www/myapp`).

### Step 5: Install and Configure GitLab Runner

1. **Install GitLab Runner:**
   ```bash
   sudo apt install -y curl
   curl -L https://packages.gitlab.com/install/repositories/gitlab/gitlab-runner/script.deb.sh | sudo bash
   sudo apt install gitlab-runner
   ```

2. **Register the Runner:**
   ```bash
   sudo gitlab-runner register
   ```
   - Enter your GitLab instance URL.
   - Use the token from your project’s settings.
   - Choose `shell` as the executor.

### Step 6: Create `.gitlab-ci.yml`

Create a `.gitlab-ci.yml` file in your project’s root directory:

```yaml
stages:
  - deploy
  - rollback

deploy:
  stage: deploy
  script:
    - echo "$SSH_PRIVATE_KEY" > privatekey.pem
    - chmod 600 privatekey.pem
    - scp -o StrictHostKeyChecking=no -i privatekey.pem -r ./dist/* $DEPLOY_USER@$SERVER_IP:$APP_DIR/
    - ssh -o StrictHostKeyChecking=no -i privatekey.pem $DEPLOY_USER@$SERVER_IP 'cd $APP_DIR && npm install --production && pm2 restart your-app-name'
  only:
    - main

rollback:
  stage: rollback
  script:
    - ssh -o StrictHostKeyChecking=no -i privatekey.pem $DEPLOY_USER@$SERVER_IP 'pm2 revert your-app-name'
  when: manual
```

### Step 7: Implement Rollback Functionality

Using PM2, you can easily revert to the previous version:
```bash
pm2 revert your-app-name
```
Ensure PM2 is installed and your application is managed by it.

### Step 8: Test Your Pipeline

1. **Push Changes:**
   Commit and push your changes to the main branch.

2. **Monitor CI/CD Pipeline:**
   Go to your GitLab project and navigate to `CI/CD > Pipelines` to check the deployment process.

3. **Manual Rollback:**
   You can manually trigger the rollback from the pipeline interface as needed.

### Example Directory Structure

Here's an example of how your project directory might look:

```
my-node-app/
│
├── .gitlab-ci.yml
├── package.json
├── src/
│   └── app.js
└── dist/
    └── (compiled files)
```

### Conclusion

By following these steps, you will have a complete continuous deployment pipeline for your Node.js application using GitLab CI/CD. Ensure to monitor your deployments and maintain server security practices.

😙💯☺️☺️☺️☺️😄🚫😄🔐😀🔑😃🥴😔😔😘👇👇😆😚😚😚😀😄🔐🔐😚😍😭😚😔😉😉😉😘😔😔😔😘😘😘🙂🙂🙂😉😉😉😚😉😚😆😉😉😉😔😔😔😔😔😉🙂🙂😔😔😔


Here’s a comprehensive guide on setting up a continuous deployment pipeline using GitLab CI/CD for a Node.js project on an Ubuntu server. This includes SSH private key authentication, variable settings, creating a deployer user, setting up GitLab Runner, and implementing rollback functionality.

### Step 1: Prepare Your Ubuntu Server

1. **Update your server:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Install Node.js and npm:**
   ```bash
   curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
   sudo apt install -y nodejs
   ```

3. **Install Git:**
   ```bash
   sudo apt install -y git
   ```

### Step 2: Create a Deployer User

1. **Create a new user for deployment:**
   ```bash
   sudo adduser deployer
   ```

2. **Grant the deployer user permission to use `sudo` (optional):**
   ```bash
   sudo usermod -aG sudo deployer
   ```

### Step 3: Set Up SSH Access with Private Key Authentication

1. **Switch to the deployer user:**
   ```bash
   su - deployer
   ```

2. **Generate SSH keys:**
   ```bash
   ssh-keygen -t rsa -b 4096 -C "deployer@your_server"
   ```
   Press Enter to accept the default file location, and enter a passphrase if desired.

3. **Copy the public key to the authorized keys:**
   ```bash
   cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
   ```

4. **Set appropriate permissions:**
   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

5. **Test SSH access:**
   ```bash
   ssh deployer@your_server_ip
   ```

### Step 4: Set Up GitLab Runner

1. **Install GitLab Runner:**
   ```bash
   sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
   sudo chmod +x /usr/local/bin/gitlab-runner
   sudo gitlab-runner install
   ```

2. **Register the GitLab Runner:**
   ```bash
   sudo gitlab-runner register
   ```
   - Enter your GitLab instance URL and the registration token from your project’s CI/CD settings.
   - Choose `shell` as the executor.

3. **Start GitLab Runner:**
   ```bash
   sudo gitlab-runner start
   ```

### Step 5: Set Up Your Node.js Project

1. **Create a simple Node.js app (if you haven’t already):**
   ```bash
   mkdir ~/my-node-app
   cd ~/my-node-app
   npm init -y
   npm install express
   ```

2. **Create a basic server in `index.js`:**
   ```javascript
   const express = require('express');
   const app = express();
   const PORT = process.env.PORT || 3000;

   app.get('/', (req, res) => {
       res.send('Hello, World!');
   });

   app.listen(PORT, () => {
       console.log(`Server is running on port ${PORT}`);
   });
   ```

3. **Push your project to GitLab:**
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin <your_gitlab_repo_url>
   git push -u origin master
   ```

### Step 6: Configure GitLab CI/CD with Environment Variables

1. **Define environment variables in GitLab:**
   - Go to your project in GitLab.
   - Navigate to **Settings > CI / CD > Variables**.
   - Add the following variables:
     - `DEPLOY_USER` set to `deployer`
     - `SERVER_IP` set to your server's IP
     - `PRIVATE_KEY` set to the contents of your private key file (e.g., `~/.ssh/id_rsa`).

2. **Create a `.gitlab-ci.yml` file in the root of your project:**
   ```yaml
   stages:
     - deploy

   deploy:
     stage: deploy
     script:
       - echo "$PRIVATE_KEY" > private_key.pem
       - chmod 600 private_key.pem
       - ssh -o StrictHostKeyChecking=no -i private_key.pem $DEPLOY_USER@$SERVER_IP "
           cd /path/to/your/app &&
           git pull origin master &&
           npm install &&
           pm2 restart all
         "
     only:
       - master

   rollback:
     stage: deploy
     script:
       - echo "$PRIVATE_KEY" > private_key.pem
       - chmod 600 private_key.pem
       - ssh -o StrictHostKeyChecking=no -i private_key.pem $DEPLOY_USER@$SERVER_IP "
           cd /path/to/your/app &&
           git checkout previous_commit_id &&
           npm install &&
           pm2 restart all
         "
     only:
       - rollback-branch
   ```

### Step 7: Implement Rollback Functionality

1. **Modify the deployment script to save the current commit:**
   ```yaml
   deploy:
     stage: deploy
     script:
       - echo "$PRIVATE_KEY" > private_key.pem
       - chmod 600 private_key.pem
       - ssh -o StrictHostKeyChecking=no -i private_key.pem $DEPLOY_USER@$SERVER_IP "
           cd /path/to/your/app &&
           git rev-parse HEAD > current_commit.txt &&
           git pull origin master &&
           npm install &&
           pm2 restart all
         "
     only:
       - master

   rollback:
     stage: deploy
     script:
       - echo "$PRIVATE_KEY" > private_key.pem
       - chmod 600 private_key.pem
       - ssh -o StrictHostKeyChecking=no -i private_key.pem $DEPLOY_USER@$SERVER_IP "
           cd /path/to/your/app &&
           git checkout \$(cat current_commit.txt) &&
           npm install &&
           pm2 restart all
         "
     only:
       - rollback-branch
   ```

### Step 8: Finalize and Test

1. **Commit your changes to `.gitlab-ci.yml` and push to GitLab:**
   ```bash
   git add .gitlab-ci.yml
   git commit -m "Add CI/CD configuration"
   git push origin master
   ```

2. **Trigger the pipeline in GitLab and monitor for any issues.**

### Conclusion

You have successfully set up a continuous deployment pipeline for a Node.js project using GitLab CI/CD on an Ubuntu server, with SSH private key authentication and environment variables. The setup also includes rollback functionality. Test your pipeline thoroughly to ensure everything is functioning as expected.
😄😄🥴😁😃😁🥴🥴😁😃😃🥹💯🤩🚫😍😍🥹😍💙😌💙😌😙🤩😙🚫😆😚😄😚🤩😚✅😙✅😙💙😙✅😙✅😙💙😙🤩🤩😙💙😙✅😙🔐😙🤩😙💯😙✅😙✅😙💯😙🤩😙💙😙💙✅😙✅😙🔐😙💯💙😙✅😙✅😙✅😙🚫😙🚫😙😄😙💙😙💙😙✅😙🤩😙🤩😙😅😙😊😙😭😙😍☺️🚫😙😁😙💯💯😙
Here's a step-by-step guide to set up a continuous deployment pipeline with GitLab CI/CD on Ubuntu for your project. 

### 1. Create GitLab Repository
1. **Sign in to GitLab**: Go to [GitLab](https://gitlab.com) and log in.
2. **Create a new project**:
   - Click on the "+" icon or "New Project".
   - Select "Create blank project".
   - Fill in the project name (e.g., `my-project`), visibility level, and other settings.
   - Click "Create project".

### 2. Register GitLab Runner
1. **Install GitLab Runner** on your Ubuntu server:
   ```bash
   sudo apt-get update
   sudo apt-get install -y curl
   curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-runner/script.deb.sh | sudo bash
   sudo apt-get install gitlab-runner
   ```

2. **Register the Runner**:
   - Obtain your GitLab instance URL and registration token from your project settings (Settings -> CI/CD -> Runners).
   - Run the registration command:
     ```bash
     sudo gitlab-runner register
     ```
   - Follow the prompts:
     - **Enter your GitLab instance URL**: `https://gitlab.com`
     - **Provide the registration token**: (e.g., `REGISTRATION_TOKEN`)
     - **Set a description for the runner**: `My Runner`
     - **Specify the executor**: `shell` (or `docker` if you prefer).
     - **Optionally, define tags**: `deploy`.

### 3. Create a Deployment User
1. **Create a new user on your server**:
   ```bash
   sudo adduser deployer
   ```
   Follow the prompts to set a password and complete the user creation.

2. **Grant appropriate permissions** (if necessary):
   ```bash
   sudo usermod -aG sudo deployer
   ```

### 4. Setup SSH Key
1. **Generate SSH Key**:
   - Log in as the deployment user:
     ```bash
     su - deployer
     ```
   - Generate an SSH key pair:
     ```bash
     ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
     ```
   - Press Enter to save it in the default location and choose not to set a passphrase.

2. **Add the public key to the server's authorized keys**:
   ```bash
   cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
   ```

3. **Set correct permissions**:
   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

### 5. Storing SSH Key in GitLab
1. **Navigate to your project in GitLab**.
2. **Go to Settings -> CI/CD**.
3. **Expand the Variables section**:
   - Add a new variable:
     - **Key**: `DEPLOY_SSH_KEY`
     - **Value**: Paste the content of `~/.ssh/id_rsa`.
     - **Protect variable**: Check this if you are deploying from protected branches.

### 6. Configure the GitLab CI/CD Pipeline
1. **Create a `.gitlab-ci.yml` file** in the root of your repository:
   ```yaml
   stages:
     - build
     - deploy

   build:
     stage: build
     image: node:14
     script:
       - npm install
       - npm run build
     artifacts:
       paths:
         - build/

   deploy:
     stage: deploy
     image: ruby:2.7
     before_script:
       - echo "$DEPLOY_SSH_KEY" > deploy_key
       - chmod 600 deploy_key
     script:
       - apt-get update -y
       - apt-get install -y sshpass
       - sshpass -p "$DEPLOY_PASSWORD" scp -o StrictHostKeyChecking=no -i deploy_key -r build/ deployer@your-server-ip:/path/to/deployment
     only:
       - main
   ```
   Replace `/path/to/deployment` with the actual path on your server.

### 7. Validate the Deployment
1. **Access your server**: SSH into your server.
   ```bash
   ssh deployer@your-server-ip
   ```
2. **Check the deployed application**:
   - Navigate to the deployment directory:
     ```bash
     cd /path/to/deployment
     ```
   - Ensure the files are present and run your server (e.g., using `serve`):
     ```bash
     npx serve -s build
     ```

### 8. How to Roll Back a Deployment
1. **Keep backup versions**:
   - Create a backup before deploying by copying the current deployment:
     ```bash
     cp -r /path/to/deployment /path/to/deployment_backup
     ```

2. **Rollback Command**:
   - If a rollback is needed, SSH into your server:
     ```bash
     ssh deployer@your-server-ip
     ```
   - Move the current deployment to a backup folder:
     ```bash
     mv /path/to/deployment /path/to/deployment_current_backup
     ```
   - Restore the previous version:
     ```bash
     cp -r /path/to/deployment_backup /path/to/deployment
     ```
3. **Restart the application server** if necessary.

### Conclusion
This guide provides a comprehensive setup for a continuous deployment pipeline using GitLab CI/CD on Ubuntu. Adjust the `.gitlab-ci.yml` file as needed for your specific application requirements.








😆😁😁😃😃😁😆😆😆😄🔑😁🔑🤩🤩😄🔑🔑👍🤩🤩🤩👍👍🔑🔑🔑🔑💯😔😔😀😀💯🔑😄😄

Setting up a continuous deployment pipeline with GitLab CI/CD on Ubuntu involves several steps. Here’s a step-by-step guide, including an example project.

### Step 1: Prerequisites

1. **GitLab Account**: Ensure you have a GitLab account and a project repository.
2. **Ubuntu Server**: You need an Ubuntu server where the application will be deployed. Ensure SSH access is available.
3. **GitLab Runner**: Install GitLab Runner on your server or use a shared runner.

### Step 2: Install GitLab Runner

1. **Add the GitLab Runner Repository**:
   ```bash
   sudo apt-get update
   sudo apt-get install -y curl
   curl -L --output /tmp/gitlab-runner.deb https://gitlab-runner-downloads.s3.amazonaws.com/latest/deb/gitlab-runner_amd64.deb
   sudo dpkg -i /tmp/gitlab-runner.deb
   ```

2. **Start the GitLab Runner**:
   ```bash
   sudo gitlab-runner start
   ```

### Step 3: Register GitLab Runner

1. **Get the Registration Token**: In your GitLab project, navigate to `Settings` > `CI/CD` > `Runners` and copy the registration token.

2. **Register the Runner**:
   ```bash
   sudo gitlab-runner register
   ```
   - Enter your GitLab instance URL (e.g., `https://gitlab.com`).
   - Paste the registration token.
   - Provide a description for the runner.
   - Choose tags for the runner (optional).
   - Select the executor (e.g., `shell` or `docker`).

### Step 4: Create Your Application

For this example, let’s assume we have a simple Node.js application.

1. **Create a Sample Node.js App**:
   ```bash
   mkdir my-node-app
   cd my-node-app
   npm init -y
   npm install express
   ```

2. **Create `app.js`**:
   ```javascript
   const express = require('express');
   const app = express();
   const PORT = process.env.PORT || 3000;

   app.get('/', (req, res) => {
       res.send('Hello, World!');
   });

   app.listen(PORT, () => {
       console.log(`Server running on port ${PORT}`);
   });
   ```

3. **Push the App to GitLab**:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin <your-repo-url>
   git push -u origin master
   ```

### Step 5: Create `.gitlab-ci.yml`

In your project root, create a file named `.gitlab-ci.yml` to define the CI/CD pipeline.

```yaml
stages:
  - build
  - deploy

build:
  stage: build
  image: node:latest
  script:
    - npm install
    - npm run build # Adjust if you have a build script
  artifacts:
    paths:
      - node_modules/

deploy:
  stage: deploy
  script:
    - ssh user@your-server-ip 'cd /path/to/your/app && git pull origin master && npm install && pm2 restart all'
  only:
    - master
```

### Step 6: Set Up Deployment Server

1. **Install PM2 on the Server**:
   ```bash
   sudo npm install -g pm2
   ```

2. **Clone Your Repository**:
   ```bash
   git clone <your-repo-url> /path/to/your/app
   cd /path/to/your/app
   npm install
   pm2 start app.js --name my-node-app
   ```

### Step 7: Configure SSH Access

1. **Generate SSH Key**:
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

2. **Add the Public Key to Your Server**:
   Copy the contents of `~/.ssh/id_rsa.pub` to `~/.ssh/authorized_keys` on your server.

3. **Add SSH Key to GitLab**:
   - Go to your GitLab project.
   - Navigate to `Settings` > `CI/CD` > `Variables`.
   - Add a new variable named `SSH_PRIVATE_KEY` and paste the content of your private key (`~/.ssh/id_rsa`).

### Step 8: Test Your Pipeline

1. **Commit Changes**: Make a change in your application and push it to the `master` branch.
2. **Check Pipeline**: Go to your GitLab project, click on `CI/CD` > `Pipelines`, and observe the running pipeline.

### Conclusion

You’ve set up a basic continuous deployment pipeline with GitLab CI/CD on Ubuntu. You can expand this setup by adding tests, notifications, and different environments as needed.
