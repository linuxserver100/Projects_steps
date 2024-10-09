
Setting up a Continuous Deployment (CD) pipeline for a Node.js project using GitLab CI/CD on an Ubuntu server involves a series of steps. Below is a detailed, step-by-step process.

### Step 1: Prerequisites

1. **Ubuntu Server**: Ensure you have an Ubuntu server set up.
2. **Node.js and npm**: Install Node.js and npm on your server.
   ```bash
   sudo apt update
   sudo apt install -y nodejs npm
   ```
3. **GitLab Account**: Have access to a GitLab repository.
4. **SSH Access**: Ensure you can SSH into your Ubuntu server.

### Step 2: Prepare Your Node.js Application

1. **Create a Node.js Project**:
   - Navigate to your project directory and initialize your Node.js application.
   ```bash
   mkdir my-node-app
   cd my-node-app
   npm init -y
   ```

2. **Add a Simple Application**:
   Create an `app.js` file with a simple server:
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

3. **Add Dependencies**:
   Install Express:
   ```bash
   npm install express
   ```

### Step 3: Create `.gitlab-ci.yml`

1. **Add a GitLab CI/CD Configuration File**:
   Create a `.gitlab-ci.yml` file in your project root:
   ```yaml
   stages:
     - build
     - test
     - deploy

   cache:
     paths:
       - node_modules/

   before_script:
     - npm install

   build:
     stage: build
     script:
       - echo "Building the application"

   test:
     stage: test
     script:
       - npm test

   deploy:
     stage: deploy
     environment:
       name: production
       url: http://your-app-url
     script:
       - ssh user@your-server-ip "cd /path/to/your/app && git pull origin main && npm install && pm2 restart your-app"
     only:
       - main
   ```

### Step 4: Set Up SSH Keys

1. **Generate SSH Keys**:
   On your local machine (or your CI runner), run:
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```
   Leave the passphrase empty.

2. **Add SSH Key to GitLab**:
   - Copy the contents of `~/.ssh/id_rsa.pub` and go to your GitLab repository. Under **Settings > CI/CD > Variables**, add a new variable:
     - **Key**: `SSH_PRIVATE_KEY`
     - **Value**: Paste the private key from `~/.ssh/id_rsa`.

3. **Add the SSH Key to the Server**:
   - Copy the public key to your server:
   ```bash
   ssh-copy-id user@your-server-ip
   ```

### Step 5: Install and Configure PM2

1. **Install PM2**:
   On your server, install PM2 globally:
   ```bash
   npm install -g pm2
   ```

2. **Start Your Application**:
   Start your Node.js app with PM2:
   ```bash
   cd /path/to/your/app
   pm2 start app.js --name your-app
   ```

3. **Save PM2 Process List**:
   Save the PM2 process list so it can be resurrected after a reboot:
   ```bash
   pm2 save
   ```

### Step 6: Configure GitLab CI/CD Variables

1. **Environment Variables**:
   If your application uses environment variables, add them in GitLab under **Settings > CI/CD > Variables**.

### Step 7: Push Changes to GitLab

1. **Commit and Push Changes**:
   Commit your changes and push them to GitLab:
   ```bash
   git add .
   git commit -m "Set up CI/CD pipeline"
   git push origin main
   ```

### Step 8: Monitor the Pipeline

1. **Check GitLab CI/CD**:
   Go to your GitLab repository and navigate to **CI/CD > Pipelines**. You should see your pipeline running.

### Step 9: Debugging

- If there are any issues, you can check the logs for each job in the pipeline for debugging.

### Conclusion

You have successfully set up a Continuous Deployment pipeline for your Node.js project using GitLab CI/CD on an Ubuntu server. This pipeline will automatically build, test, and deploy your application whenever changes are pushed to the main branch. You can customize it further based on your project's needs.
