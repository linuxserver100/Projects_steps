
Setting up a continuous deployment (CD) pipeline for a React project using GitLab CI/CD on an Ubuntu server involves several steps. Here’s a detailed process:

### Prerequisites

1. **Ubuntu Server**: Make sure you have an Ubuntu server running.
2. **Node.js and npm**: Install Node.js and npm if they are not already installed.
   ```bash
   sudo apt update
   sudo apt install nodejs npm
   ```
3. **GitLab Repository**: You need a GitLab account and a repository for your React project.
4. **SSH Access**: Ensure you have SSH access to your server from your local machine and from GitLab.

### Step 1: Prepare Your React Project

1. **Create a React Project**:
   If you haven't already, create a React application using Create React App:
   ```bash
   npx create-react-app my-app
   cd my-app
   git init
   git add .
   git commit -m "Initial commit"
   ```
2. **Push to GitLab**:
   Create a new repository on GitLab and push your project:
   ```bash
   git remote add origin <your-gitlab-repo-url>
   git push -u origin master
   ```

### Step 2: Configure the Server

1. **Install Nginx**: This will serve your React app.
   ```bash
   sudo apt install nginx
   ```
2. **Set Up Nginx**:
   Create a configuration file for your app:
   ```bash
   sudo nano /etc/nginx/sites-available/my-app
   ```

   Add the following content, replacing `your_domain_or_IP`:
   ```nginx
   server {
       listen 80;
       server_name your_domain_or_IP;

       location / {
           root /var/www/my-app/build;
           try_files $uri $uri/ /index.html;
       }
   }
   ```

   Link the configuration:
   ```bash
   sudo ln -s /etc/nginx/sites-available/my-app /etc/nginx/sites-enabled/
   ```

   Test the Nginx configuration and restart it:
   ```bash
   sudo nginx -t
   sudo systemctl restart nginx
   ```

3. **Set Up Directory for Build**:
   Create a directory for your app:
   ```bash
   sudo mkdir -p /var/www/my-app/build
   ```

### Step 3: Create a GitLab CI/CD Pipeline

1. **Create `.gitlab-ci.yml`**:
   In your project root, create a file named `.gitlab-ci.yml` with the following content:
   ```yaml
   image: node:14

   stages:
     - build
     - deploy

   cache:
     paths:
       - node_modules/

   build:
     stage: build
     script:
       - npm install
       - npm run build
     artifacts:
       paths:
         - build/

   deploy:
     stage: deploy
     script:
       - ssh user@your_server_ip "rm -rf /var/www/my-app/build/*"
       - scp -r build/* user@your_server_ip:/var/www/my-app/build/
     only:
       - master
   ```

   Replace `user` and `your_server_ip` with your actual server username and IP address.

2. **Add SSH Key**:
   Generate an SSH key on your local machine (if you don’t have one):
   ```bash
   ssh-keygen -t rsa -b 2048
   ```
   Add the public key (`~/.ssh/id_rsa.pub`) to the `~/.ssh/authorized_keys` file on your server.

3. **Add SSH Key to GitLab**:
   Go to your GitLab project settings, navigate to CI/CD > Variables, and add a new variable:
   - **Key**: `SSH_PRIVATE_KEY`
   - **Value**: (paste the content of `~/.ssh/id_rsa`)

   Check "Protect variable" if you want to restrict access.

### Step 4: Update the Pipeline to Use SSH Key

Modify the `deploy` section of your `.gitlab-ci.yml` to use the SSH key:
```yaml
deploy:
  stage: deploy
  before_script:
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan -H your_server_ip >> ~/.ssh/known_hosts
  script:
    - ssh user@your_server_ip "rm -rf /var/www/my-app/build/*"
    - scp -r build/* user@your_server_ip:/var/www/my-app/build/
  only:
    - master
```

### Step 5: Commit and Push Changes

1. Commit your changes to `.gitlab-ci.yml`:
   ```bash
   git add .gitlab-ci.yml
   git commit -m "Add CI/CD pipeline"
   git push origin master
   ```

### Step 6: Monitor the Pipeline

1. Go to your GitLab repository, navigate to CI/CD > Pipelines, and monitor the progress of your pipeline.
2. After a successful build and deployment, you should be able to access your React app via your server's domain or IP.

### Conclusion

You’ve now set up a continuous deployment pipeline for your React project using GitLab CI/CD on Ubuntu. This pipeline will automatically build and deploy your application every time you push changes to the `master` branch.
