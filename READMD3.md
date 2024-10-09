
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
