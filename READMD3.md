🔑

Here’s a detailed guide to setting up a continuous deployment pipeline with GitLab CI/CD for a Node.js project on an Ubuntu server. This guide covers setting up the deployer, deployment process, and GitLab Runner.

### Step 1: Create Your Node.js Project

1. **Create Project Directory**:
   ```bash
   mkdir my-node-app
   cd my-node-app
   ```

2. **Initialize the Project**:
   ```bash
   npm init -y
   ```

3. **Install Express**:
   ```bash
   npm install express
   ```

4. **Create Basic Server**:
   Create a file named `app.js`:
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

5. **Test Locally**:
   Run your application to ensure it works:
   ```bash
   node app.js
   ```

### Step 2: Push the Project to GitLab

1. **Create a New Repository** on GitLab.

2. **Initialize Git and Push**:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin <your-gitlab-repo-url>
   git push -u origin main
   ```

### Step 3: Set Up the Deployer

1. **Create a Deployer User**:
   ```bash
   sudo adduser deployer
   ```

2. **Set Up SSH Key for Deployer**:
   Generate an SSH key for the deployer:
   ```bash
   sudo -u deployer ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

3. **Copy Public Key to Authorized Keys**:
   Append the public key to `~/.ssh/authorized_keys`:
   ```bash
   cat /home/deployer/.ssh/id_rsa.pub | sudo tee -a /home/deployer/.ssh/authorized_keys
   ```

4. **Set Permissions**:
   ```bash
   sudo chown -R deployer:deployer /home/deployer/.ssh
   sudo chmod 700 /home/deployer/.ssh
   sudo chmod 600 /home/deployer/.ssh/authorized_keys
   ```

5. **Test SSH Access**:
   Ensure you can SSH into the server using the deployer account:
   ```bash
   ssh deployer@your-server-ip
   ```

### Step 4: Install PM2 for Process Management

1. **Switch to Deployer User**:
   ```bash
   sudo su - deployer
   ```

2. **Install PM2 Globally**:
   ```bash
   npm install -g pm2
   ```

3. **Start Your Application with PM2**:
   In your application directory:
   ```bash
   pm2 start app.js --name my-node-app
   ```

4. **Save PM2 Process List**:
   To ensure PM2 restarts your app on server reboot:
   ```bash
   pm2 save
   pm2 startup
   ```

### Step 5: Set Up GitLab Runner

1. **Install GitLab Runner**:
   On your server, run:
   ```bash
   sudo apt-get update
   sudo apt-get install -y gitlab-runner
   ```

2. **Register the GitLab Runner**:
   ```bash
   sudo gitlab-runner register
   ```
   - **GitLab Instance URL**: Enter your GitLab URL (e.g., `https://gitlab.com`).
   - **Registration Token**: Obtain this from your GitLab project (Settings > CI/CD > Runners).
   - **Description**: Provide a description (e.g., `My Node.js Runner`).
   - **Tags**: Add tags if needed (e.g., `node`).
   - **Executor**: Choose `shell`.

### Step 6: Create `.gitlab-ci.yml` for CI/CD

1. **Create `.gitlab-ci.yml`**:
   In the root of your project, create a file named `.gitlab-ci.yml`:
   ```yaml
 
   stages:
     - build
     - deploy

   before_script:
    - apt-get update -y
    - apt-get install -y openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan 98.84.112.138 >> ~/.ssh/known_hosts
  

   build:
     stage: build
     script:
       - npm install

   deploy:
     stage: deploy
     environment:
       name: production
       url: http://your-server-ip:3000
     script:
       - ssh deployer@your-server-ip 'cd /path/to/your/app && git pull origin main && npm install && pm2 restart my-node-app'
     only:
       - main
   ```

### Step 7: Commit and Push Your CI/CD Configuration

1. **Commit Your Changes**:
   ```bash
   git add .gitlab-ci.yml
   git commit -m "Add CI/CD configuration"
   git push origin main
   ```

### Step 8: Test Your Pipeline

1. **Check the Pipeline**:
   Go to your GitLab repository, navigate to **CI/CD > Pipelines**, and ensure the pipeline runs successfully.

2. **Access Your Application**:
   Open a web browser and visit `http://your-server-ip:3000` to see your Node.js application running.

### Step 9: Rollback Deployment (Optional)

To implement a rollback strategy, you can modify the `.gitlab-ci.yml` to include a rollback job.

1. **Modify `.gitlab-ci.yml`**:
   ```yaml
   stages:
     - build
     - deploy
     - rollback


   before_script:
   - apt-get update -y
   - apt-get install -y openssh-client
   - eval $(ssh-agent -s)
   - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
   - mkdir -p ~/.ssh
   - chmod 700 ~/.ssh
   - ssh-keyscan 98.84.112.138 >> ~/.ssh/known_hosts
  

   build:
     stage: build
     script:
       - npm install

   deploy:
     stage: deploy
     environment:
       name: production
       url: http://your-server-ip:3000
     script:
       - ssh deployer@your-server-ip 'cd /path/to/your/app && git pull origin main && npm install && pm2 restart my-node-app'
     only:
       - main

   rollback:
     stage: rollback
     script:
       - ssh deployer@your-server-ip 'cd /path/to/your/app && git checkout HEAD~1 && npm install && pm2 restart my-node-app'
     when: manual
     environment:
       name: production
   ```

2. **Commit and Push**:
   ```bash
   git add .gitlab-ci.yml
   git commit -m "Add rollback job"
   git push origin main
   ```

### Summary

- **Deployer User**: Configured for deployment using SSH.
- **PM2**: Manages your Node.js application.
- **GitLab Runner**: Executes CI/CD jobs.
- **Continuous Deployment Pipeline**: Automatically deploys updates and includes a rollback option.

### Conclusion

You have successfully set up a continuous deployment pipeline for your Node.js application using GitLab CI/CD on an Ubuntu server. You can deploy updates automatically, manage your application with PM2, and roll back deployments if needed. Adjust configurations as necessary for your specific requirements.


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
