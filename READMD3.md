😂
To set up a continuous deployment pipeline for a full-stack React application using GitLab CI/CD, ensuring that you deploy the contents of the `./build/*` directory, follow these steps:

### Project Structure

Assume your project structure is as follows:

```
my-app/
├── client/          # React frontend
│   ├── build/       # Output directory for the React app
│   ├── package.json
│   └── ...          # Other React files
├── server/          # Node.js backend
│   ├── build/       # Output directory for the Node app (if applicable)
│   ├── package.json
│   └── ...          # Other Node files
├── .gitlab-ci.yml   # GitLab CI/CD configuration
└── ...              # Other project files
```

### Step 1: Create `.gitlab-ci.yml`

Create a `.gitlab-ci.yml` file in the root of your project with the following content:

```yaml
stages:
  - build
  - test
  - deploy

variables:
  NODE_ENV: "production"

cache:
  paths:
    - client/node_modules/
    - server/node_modules/

build_client:
  stage: build
  image: node:14
  script:
    - cd client
    - npm install
    - npm run build
  artifacts:
    paths:
      - client/build

build_server:
  stage: build
  image: node:14
  script:
    - cd server
    - npm install
  artifacts:
    paths:
      - server/build

test:
  stage: test
  image: node:14
  script:
    - cd client
    - npm run test -- --watch=false
    - cd ../server
    - npm run test -- --watch=false

deploy:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache openssh
    - echo "$SSH_PRIVATE_KEY" > private_key && chmod 600 private_key
    - ssh -i private_key -o StrictHostKeyChecking=no user@your-server-ip "rm -rf /path/to/your/app/build/* && cp -r /path/to/repo/client/build/* /path/to/your/app/build/ && cd /path/to/your/app/server && npm install --production && npm run start"
  only:
    - main
```

### Step 2: Configure SSH for Deployment

1. **Generate SSH Key** (if not already created):
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

2. **Add Public Key to Your Server**:
   Copy the contents of `~/.ssh/id_rsa.pub` to your server's `~/.ssh/authorized_keys`.

3. **Add Private Key to GitLab**:
   - Go to your GitLab project.
   - Navigate to **Settings > CI/CD > Variables**.
   - Add a variable called `SSH_PRIVATE_KEY` and paste your private key there.

### Step 3: Update the Deploy Script

Ensure your deploy script correctly points to the paths where your application will be served. The `rm -rf /path/to/your/app/build/*` command removes any existing files in the build directory on your server, and `cp -r /path/to/repo/client/build/* /path/to/your/app/build/` copies the newly built files from the CI environment to your server.

### Step 4: Testing the Pipeline

1. **Push Your Code**:
   Commit and push your code to the `main` branch.

2. **Check Pipelines**:
   Navigate to **CI/CD > Pipelines** in your GitLab project to monitor the pipeline stages. Each stage should execute in order: build the client, build the server, run tests, and deploy.

### Conclusion

With this setup, you now have a complete CI/CD pipeline that builds your full-stack React application, runs tests, and deploys the contents of `./build/*` to your server whenever you push to the `main` branch. Adjust paths and commands in the `.gitlab-ci.yml` as necessary to fit your project’s specific structure and requirements.




😄😄🥭👇🥭😌😄😄😌🥭😍🥭😍😄😍😄🙂‍↕️😂😉🙂‍↕️😉🙂‍↕️😉😔😂😔😂😔😄😔😄😔😄😔😂😔😂😔😄😔😄😔😄😔😄😄😔😂😔😂😔😂😔😄😔😄😔😄😔😄😔😄😔😄😔😄😄😔😄😔🔗🙂‍↕️😚🙂‍↕️😅😔🤩😆🤩😆😅😆😃😆😃😆😭😆🥭😭😆🥭😆😔😄😔😄😂😔😂😔😄😔😄😔😄😔😄😔😚😔😚😔😃😔😃😔😅😔😚😔
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
😭😔😅😃😂😌😃😅😄😂🥭😆😄😃😚😔😭🥴🥴🥴🙂‍↕️😉😌🔑🔐🚫😉🔗🔗😉😌🔑😙🥴🥴😗😚☺️😘☺️😘☺️😄☺️😄☺️😚😘☺️😚😙☺️😚☺️😘😘☺️😁☺️😘☺️😆☺️😄☺️😙☺️😚😚☺️😚☺️😘☺️😘😜😘😝😚😝😚😜😚😚😜😘😝😚😝😙😝🥲😜🥹😜😘😜😚😚😜😚😜🥹😜😚😜😚😜😚😜🥹😜😘😜

Here’s a comprehensive guide to setting up a continuous deployment pipeline using GitLab CI/CD for a Node.js application on an Ubuntu server. This guide will cover private key authentication, variable settings, creating a deployer user, setting up GitLab Runner, implementing rollback functionality, and deploying the app so that you can access it in a web browser.

### Step-by-Step Guide

---

### Prerequisites

- An Ubuntu server with SSH access.
- A GitLab account with a repository for your Node.js application.
- Basic knowledge of Git, GitLab CI/CD, and Linux commands.

---

### Step 1: Prepare Your Node.js Application

1. **Create a sample Node.js application**:

   If you don’t have an existing Node.js app, you can create a simple one:

   ```bash
   mkdir my-node-app
   cd my-node-app
   npm init -y
   npm install express
   ```

2. **Create an `index.js` file**:

   ```bash
   touch index.js
   ```

   Add the following code to `index.js`:

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

3. **Test your application locally**:

   ```bash
   node index.js
   ```

   Open your browser and go to `http://localhost:3000` to see if it works.

---

### Step 2: Create a Deployer User on Your Ubuntu Server

1. **SSH into your server**:

   ```bash
   ssh username@your-server-ip
   ```

2. **Create a new user for deployment**:

   ```bash
   sudo adduser deployer
   ```

3. **Add the deployer user to the sudo group**:

   ```bash
   sudo usermod -aG sudo deployer
   ```

4. **Set up SSH access for the deployer user**:

   ```bash
   sudo mkdir /home/deployer/.ssh
   sudo touch /home/deployer/.ssh/authorized_keys
   sudo chmod 700 /home/deployer/.ssh
   sudo chmod 600 /home/deployer/.ssh/authorized_keys
   ```

5. **Copy your SSH public key to the `authorized_keys` file**:

   On your local machine, run:

   ```bash
   cat ~/.ssh/id_rsa.pub
   ```

   Then on your server:

   ```bash
   echo "your-ssh-public-key" | sudo tee -a /home/deployer/.ssh/authorized_keys
   ```

6. **Change ownership of the `.ssh` directory**:

   ```bash
   sudo chown -R deployer:deployer /home/deployer/.ssh
   ```

---

### Step 3: Set Up GitLab CI/CD Variables

1. Go to your GitLab repository.
2. Navigate to **Settings > CI/CD > Variables**.
3. Add the following variables:
   - **Key:** `DEPLOY_USER`, **Value:** `deployer`
   - **Key:** `DEPLOY_HOST`, **Value:** `your-server-ip`
   - **Key:** `DEPLOY_PATH`, **Value:** `/path/to/deploy/my-node-app` (create this directory later)
   - **Key:** `SSH_PRIVATE_KEY`, **Value:** (your private SSH key, use the "Protect variable" option)

---

### Step 4: Create the Deployment Directory on the Server

1. **SSH into your server**:

   ```bash
   ssh deployer@your-server-ip
   ```

2. **Create the deployment directory**:

   ```bash
   mkdir -p /path/to/deploy/my-node-app
   ```

3. **Set permissions**:

   ```bash
   sudo chown -R deployer:deployer /path/to/deploy/my-node-app
   ```

---

### Step 5: Create a `.gitlab-ci.yml` File

In the root of your Node.js application, create a `.gitlab-ci.yml` file with the following content:

```yaml
image: node:14

stages:
  - build
  - deploy
  - rollback

variables:
  NODE_ENV: production

before_script:
  - npm install

build:
  stage: build
  script:
    - npm install
  artifacts:
    paths:
      - node_modules/

deploy:
  stage: deploy
  only:
    - main
  script:
    - mkdir -p ~/.ssh
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add - > /dev/null
    - ssh-keyscan -H $DEPLOY_HOST >> ~/.ssh/known_hosts
    - ssh $DEPLOY_USER@$DEPLOY_HOST "rm -rf $DEPLOY_PATH/*"
    - scp -r ./* $DEPLOY_USER@$DEPLOY_HOST:$DEPLOY_PATH/

    # Start the application using PM2
    - ssh $DEPLOY_USER@$DEPLOY_HOST "cd $DEPLOY_PATH && pm2 start index.js --name my-node-app || pm2 restart my-node-app"

rollback:
  stage: rollback
  only:
    - main
  script:
    - echo "Rolling back to the previous version"
    - ssh $DEPLOY_USER@$DEPLOY_HOST "mv $DEPLOY_PATH-backup $DEPLOY_PATH"
```

---

### Step 6: Install PM2 (Process Manager)

1. **SSH into your server**:

   ```bash
   ssh deployer@your-server-ip
   ```

2. **Install PM2 globally**:

   ```bash
   sudo npm install -g pm2
   ```

3. **Start the application with PM2 (for the first time)**:

   ```bash
   cd /path/to/deploy/my-node-app
   pm2 start index.js --name my-node-app
   ```

---

### Step 7: Set Up GitLab Runner

1. **Install GitLab Runner on your server**:

   ```bash
   curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
   sudo apt-get install gitlab-runner
   ```

2. **Register the GitLab Runner**:

   ```bash
   sudo gitlab-runner register
   ```

   When prompted, provide the following information:
   - **GitLab instance URL:** (e.g., `https://gitlab.com/`)
   - **Registration token:** (find this in your project's CI/CD settings)
   - **Description:** (e.g., `My GitLab Runner`)
   - **Tags:** (optional)
   - **Executor:** choose `shell` for simplicity.

---

### Step 8: Test the CI/CD Pipeline

1. **Push your changes to the main branch**:

   ```bash
   git add .
   git commit -m "Set up CI/CD pipeline"
   git push origin main
   ```

2. **Monitor the pipeline**:
   - Go to your GitLab repository.
   - Navigate to **CI/CD > Pipelines**.
   - You should see a new pipeline running. Click on it to see detailed logs.

---

### Step 9: Access Your Application in a Web Browser

1. **Set up a web server (like Nginx) to serve your Node.js application**:

   Install Nginx:

   ```bash
   sudo apt update
   sudo apt install nginx
   ```

2. **Configure Nginx**:

   Create a new configuration file for your application:

   ```bash
   sudo nano /etc/nginx/sites-available/my-node-app
   ```

   Add the following configuration:

   ```nginx
   server {
       listen 80;
       server_name your-server-ip;  # or your domain name

       location / {
           proxy_pass http://localhost:3000;  # Port where your Node.js app runs
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

3. **Enable the configuration**:

   ```bash
   sudo ln -s /etc/nginx/sites-available/my-node-app /etc/nginx/sites-enabled/
   ```

4. **Test the Nginx configuration**:

   ```bash
   sudo nginx -t
   ```

5. **Restart Nginx**:

   ```bash
   sudo systemctl restart nginx
   ```

6. **Access your application**:
   - Open your web browser and navigate to `http://your-server-ip` or your domain name.

---

### Step 10: Implement Rollback (Optional)

To implement rollback functionality, modify the deploy stage in your `.gitlab-ci.yml` to create a backup of the current version and roll back to it if needed:

1. **Modify the deploy stage** to create a backup before deploying a new version:

```yaml
deploy:
  stage: deploy
  only:
    - main
  script:
    - mkdir -p ~/.ssh
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add - > /dev/null
    - ssh-keyscan -H $DEPLOY_HOST >> ~/.ssh/known_hosts
    - ssh $DEPLOY_USER@$DEPLOY_HOST "cp -r $DEPLOY_PATH $DEPLOY_PATH-backup"
    - ssh $DEPLOY_USER@$DEPLOY_HOST

 "rm -rf $DEPLOY_PATH/*"
    - scp -r ./* $DEPLOY_USER@$DEPLOY_HOST:$DEPLOY_PATH/
    - ssh $DEPLOY_USER@$DEPLOY_HOST "cd $DEPLOY_PATH && pm2 start index.js --name my-node-app || pm2 restart my-node-app"
```

2. **Implement the rollback stage**:

```yaml
rollback:
  stage: rollback
  only:
    - main
  script:
    - echo "Rolling back to the previous version"
    - ssh $DEPLOY_USER@$DEPLOY_HOST "rm -rf $DEPLOY_PATH && mv $DEPLOY_PATH-backup $DEPLOY_PATH"
```

### Conclusion

You now have a complete continuous deployment pipeline for your Node.js application using GitLab CI/CD on an Ubuntu server! With this setup, every push to the main branch will trigger a deployment, and you can roll back to the previous version if necessary. Additionally, you can access your application through a web browser.

Let me know if you have any questions or need further assistance!



😃😆😘🔑😘🔑😭🔑🥴🔑😉😍👇🔑😊😉🔑😭🔑😭🔑😘🔑😘🔑😃🔑😃🔑😃🔑😃🔑😘🔑😘🔑😃🔑😘🔑🔑😂🥭🔑😂🔑😭🔑😭🔗😘🔑😂🔗😂🔗😭🔗😭😭🔗😚🔗😂🔗😭🔑😂🔑😉👇😌😌😌👇😍😭😍😭😍😘😍😂😍😂😍😂😄🥭😄🥭😭🔑😘🔑😘🔑😭🔑😭😄😂😄🥭😄😂😄😂
Sure! Here’s a detailed guide to setting up a continuous deployment pipeline using GitLab CI/CD for a ReactJS application on an Ubuntu server, covering everything from private key authentication to accessing the application in a web browser.

### Step-by-Step Guide

---

### Prerequisites

- An Ubuntu server with SSH access.
- A GitLab account with a repository for your ReactJS application.
- Basic knowledge of Git, GitLab CI/CD, and Linux commands.

---

### Step 1: Prepare Your ReactJS Application

Ensure your ReactJS application is structured properly and ready for deployment. Here’s a basic folder structure:

```
my-react-app/
│
├── src/
├── public/
├── package.json
└── ...
```

You can create a sample React application if you don’t have one yet:

```bash
npx create-react-app my-react-app
cd my-react-app
```

### Step 2: Create a Deployer User on Your Ubuntu Server

1. **SSH into your server:**

   ```bash
   ssh username@your-server-ip
   ```

2. **Create a new user for deployment:**

   ```bash
   sudo adduser deployer
   ```

3. **Add the deployer user to the sudo group:**

   ```bash
   sudo usermod -aG sudo deployer
   ```

4. **Set up SSH access for the deployer user:**

   ```bash
   sudo mkdir /home/deployer/.ssh
   sudo touch /home/deployer/.ssh/authorized_keys
   sudo chmod 700 /home/deployer/.ssh
   sudo chmod 600 /home/deployer/.ssh/authorized_keys
   ```

5. **Copy your SSH public key to the `authorized_keys` file:**

   On your local machine, run:

   ```bash
   cat ~/.ssh/id_rsa.pub
   ```

   Then on your server:

   ```bash
   echo "your-ssh-public-key" | sudo tee -a /home/deployer/.ssh/authorized_keys
   ```

6. **Change ownership of the `.ssh` directory:**

   ```bash
   sudo chown -R deployer:deployer /home/deployer/.ssh
   ```

---

### Step 3: Set Up GitLab CI/CD Variables

1. Go to your GitLab repository.
2. Navigate to **Settings > CI/CD > Variables**.
3. Add the following variables:
   - **Key:** `DEPLOY_USER`, **Value:** `deployer`
   - **Key:** `DEPLOY_HOST`, **Value:** `your-server-ip`
   - **Key:** `DEPLOY_PATH`, **Value:** `/path/to/deploy/my-react-app` (create this directory later)
   - **Key:** `SSH_PRIVATE_KEY`, **Value:** (your private SSH key, use the "Protect variable" option)

---

### Step 4: Create the Deployment Directory on the Server

1. **SSH into your server:**

   ```bash
   ssh deployer@your-server-ip
   ```

2. **Create the deployment directory:**

   ```bash
   mkdir -p /path/to/deploy/my-react-app
   ```

3. **Set permissions:**

   ```bash
   sudo chown -R deployer:deployer /path/to/deploy/my-react-app
   ```

---

### Step 5: Create a `.gitlab-ci.yml` File

In the root of your ReactJS application, create a `.gitlab-ci.yml` file with the following content:

```yaml
image: node:14

stages:
  - build
  - deploy
  - rollback

variables:
  NODE_ENV: production

before_script:
  - npm install

build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - build/

deploy:
  stage: deploy
  only:
    - main
  script:
    - mkdir -p ~/.ssh
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add - > /dev/null
    - ssh-keyscan -H $DEPLOY_HOST >> ~/.ssh/known_hosts
    - ssh $DEPLOY_USER@$DEPLOY_HOST "rm -rf $DEPLOY_PATH/*"
    - scp -r build/* $DEPLOY_USER@$DEPLOY_HOST:$DEPLOY_PATH/

rollback:
  stage: rollback
  script:
    - echo "Rollback functionality not implemented"
```

---

### Step 6: Set Up GitLab Runner

1. **Install GitLab Runner on your server:**

   ```bash
   curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
   sudo apt-get install gitlab-runner
   ```

2. **Register the GitLab Runner:**

   ```bash
   sudo gitlab-runner register
   ```

   When prompted, provide the following information:
   - **GitLab instance URL:** (e.g., `https://gitlab.com/`)
   - **Registration token:** (find this in your project's CI/CD settings)
   - **Description:** (e.g., `My GitLab Runner`)
   - **Tags:** (optional)
   - **Executor:** choose `shell` for simplicity.

---

### Step 7: Test the CI/CD Pipeline

1. **Push your changes to the main branch:**

   ```bash
   git add .
   git commit -m "Set up CI/CD pipeline"
   git push origin main
   ```

2. **Monitor the pipeline:**
   - Go to your GitLab repository.
   - Navigate to **CI/CD > Pipelines**.
   - You should see a new pipeline running. Click on it to see detailed logs.

---

### Step 8: Access Your Application in a Web Browser

1. **Set up a web server (like Nginx) to serve your React application:**

   Install Nginx:

   ```bash
   sudo apt update
   sudo apt install nginx
   ```

2. **Configure Nginx:**

   Create a new configuration file for your application:

   ```bash
   sudo nano /etc/nginx/sites-available/my-react-app
   ```

   Add the following configuration:

   ```nginx
   server {
       listen 80;
       server_name your-server-ip;  # or your domain name

       location / {
           root /path/to/deploy/my-react-app;
           try_files $uri $uri/ /index.html;
       }
   }
   ```

3. **Enable the configuration:**

   ```bash
   sudo ln -s /etc/nginx/sites-available/my-react-app /etc/nginx/sites-enabled/
   ```

4. **Test the Nginx configuration:**

   ```bash
   sudo nginx -t
   ```

5. **Restart Nginx:**

   ```bash
   sudo systemctl restart nginx
   ```

6. **Access your application:**
   - Open your web browser and navigate to `http://your-server-ip` or your domain name.

---

### Step 9: Implement Rollback (Optional)

To implement rollback functionality, consider keeping previous builds in a separate directory and modifying your `.gitlab-ci.yml`:

```yaml
rollback:
  stage: rollback
  only:
    - main
  script:
    - echo "Rolling back to the previous version"
    - ssh $DEPLOY_USER@$DEPLOY_HOST "mv $DEPLOY_PATH/previous $DEPLOY_PATH/current"
```

### Conclusion

You now have a fully functional continuous deployment pipeline for your ReactJS application using GitLab CI/CD on an Ubuntu server! With this setup, every push to the main branch will trigger a deployment. You can access your application in a web browser and modify the pipeline as needed for additional features. Let me know if you have any questions or need further assistance!



🙂😉😚😘😍😭😃😃😃😉👇👇👇💙🙂🙏👇😂😊🙂‍↕️😌🙂‍↕️😗🥭😅😌🙂😌😆😍😆😭😉👇💙😊😌😅🥭😊😂👇😂👇😂😆💙😆😭😆🥴🚫👇💙🙂🙏😉🙏🥴🙏👇👇💙😀💙😊💙👇💙😉💙😉💙🥴💙
Here’s a comprehensive guide to setting up a continuous deployment pipeline for a React.js application using GitLab CI/CD on an Ubuntu server. This guide will cover private key (.pem) authentication, variable settings, creating a deployer user, setting up GitLab Runner, and implementing rollback functionality. We’ll also use an example project to illustrate the setup.

### Example Project

For this guide, let’s assume you have a simple React.js application. If you don’t have an existing project, you can create a new one using Create React App:

```bash
npx create-react-app my-react-app
cd my-react-app
git init
git add .
git commit -m "Initial commit"
```

### Step 1: Prepare Your Ubuntu Server

1. **Update Packages**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Install Node.js**:
   Install Node.js using NVM (Node Version Manager):
   ```bash
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
   source ~/.nvm/nvm.sh
   nvm install node
   ```

3. **Install Git**:
   ```bash
   sudo apt install git -y
   ```

4. **Install Serve** (optional):
   This is to serve your built React app:
   ```bash
   npm install -g serve
   ```

### Step 2: Create a Deployer User

1. **Create the User**:
   ```bash
   sudo adduser deployer
   ```

2. **Grant Necessary Permissions**:
   If you want the deployer user to run commands with sudo:
   ```bash
   sudo usermod -aG sudo deployer
   ```

3. **Set Up SSH Key Authentication**:
   - On your local machine, generate an SSH key pair (if you don’t have one):
     ```bash
     ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
     ```
   - Copy the public key to the deployer user on the server:
     ```bash
     ssh-copy-id deployer@your_server_ip
     ```

### Step 3: GitLab CI/CD Configuration

1. **Create a `.gitlab-ci.yml` File**:
   In the root of your React project, create a `.gitlab-ci.yml` file with the following content:

   ```yaml
   stages:
     - build
     - deploy

   build:
     stage: build
     image: node:latest
     script:
       - npm install
       - npm run build
     artifacts:
       paths:
         - build/
       expire_in: 1 hour

   deploy:
     stage: deploy
     image: ruby:2.7  # Needed for SSHKit
     script:
       - mkdir -p ~/.ssh
       - echo "$SSH_PRIVATE_KEY" | tr -d '\r' > ~/.ssh/id_rsa
       - chmod 600 ~/.ssh/id_rsa
       - ssh -o StrictHostKeyChecking=no deployer@your_server_ip '
           cd /path/to/your/app && 
           git pull && 
           npm install && 
           npm run build && 
           serve -s build -l 3000
         '
     only:
       - main
   ```

   Replace `/path/to/your/app` with the actual path to your React app on the server.

2. **Set Environment Variables**:
   In your GitLab project, go to **Settings** > **CI / CD** > **Variables** and add the following variable:
   - `SSH_PRIVATE_KEY`: Your private key (.pem) content. Ensure you copy the entire content of the `.pem` file.

### Step 4: Setting Up GitLab Runner

1. **Install GitLab Runner**:
   ```bash
   sudo apt install -y gitlab-runner
   ```

2. **Register the Runner**:
   - On your GitLab project, go to **Settings** > **CI / CD** > **Runners**.
   - Note down the registration token.
   - Run the following command to register the runner:
     ```bash
     sudo gitlab-runner register
     ```
   - Follow the prompts:
     - Enter your GitLab instance URL.
     - Provide the registration token.
     - Choose a description for the runner.
     - Set the tags (optional).
     - Use `shell` as the executor.

### Step 5: Implement Rollback Functionality

To implement rollback functionality, you can modify the deploy stage in your `.gitlab-ci.yml` to include a rollback step. Here’s an example of how to do this:

1. **Modify the deploy job**:
   ```yaml
   deploy:
     stage: deploy
     image: ruby:2.7
     script:
       - mkdir -p ~/.ssh
       - echo "$SSH_PRIVATE_KEY" | tr -d '\r' > ~/.ssh/id_rsa
       - chmod 600 ~/.ssh/id_rsa
       - ssh -o StrictHostKeyChecking=no deployer@your_server_ip '
           cd /path/to/your/app && 
           git pull && 
           npm install && 
           npm run build || 
           (echo "Deployment failed, rolling back..." && git checkout HEAD^ && npm install && npm run build && serve -s build -l 3000)
         '
     only:
       - main
   ```

### Step 6: Finalize Setup and Test

1. **Push Changes**:
   Commit and push your `.gitlab-ci.yml` changes to the repository.

2. **Monitor CI/CD Pipeline**:
   Go to your GitLab project and navigate to **CI/CD** > **Pipelines** to see your pipeline running.

3. **Check Deployment**:
   Once the pipeline completes, verify that your application is running correctly on your server by navigating to `http://your_server_ip:3000`.

4. **Test Rollback**:
   To test the rollback functionality, intentionally introduce an error in your code, push it to the main branch, and observe how the pipeline handles the deployment and rollback.

### Additional Tips

- **Serving the App**: If you’re using `serve`, make sure to adjust your server configuration for better performance. Consider using Nginx or Apache for production environments.
- **Use PM2**: Consider using PM2 for managing your application processes, especially if you want your React app to run continuously and manage logs.
- **Monitor Logs**: Always check your application logs for any issues after deployment. Set up logging for better monitoring.

This guide provides a comprehensive approach to setting up a continuous deployment pipeline for your React.js application using GitLab CI/CD. Adjust paths and configurations as necessary to fit your specific project needs!

😙💯☺️☺️☺️☺️😄🚫😄🔐😀🔑😃🥴😔😔😘👇👇😆😚😚😚😀😄🔐🔐😚😍😭😚😔😉😉😉😘😔😔😔😘😘😘🙂🙂🙂😉😉😉😚😉😚😆😉😉😉😔😔😔😔😔😉🙂🙂😔😔😔


Here’s a comprehensive guide on setting up a continuous deployment pipeline using GitLab CI/CD for a Node.js project on an Ubuntu server. This includes SSH private key authentication, variable settings, creating a deployer user, setting up Gitlab Runner, and implementing rollback functionality.

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
