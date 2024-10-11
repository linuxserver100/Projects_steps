Here’s a detailed guide for setting up a continuous deployment (CD) pipeline for a Node.js application using GitLab CI/CD on an Ubuntu server, complete with an example.

### Example Node.js Application

1. **Create a Simple Node.js Application**

   Start by creating a new Node.js application:

   ```bash
   mkdir my-node-app
   cd my-node-app
   npm init -y
   npm install express
   ```

   Create a file named `app.js`:

   ```javascript
   const express = require('express');
   const app = express();
   const PORT = process.env.PORT || 3000;

   app.get('/', (req, res) => {
       res.send('Hello World!');
   });

   app.listen(PORT, () => {
       console.log(`Server is running on port ${PORT}`);
   });
   ```

   Create a `.gitignore` file:

   ```bash
   echo "node_modules" > .gitignore
   ```

   Initialize a Git repository and push to GitLab:

   ```bash
   git init
   git remote add origin <YOUR_GITLAB_REPO_URL>
   git add .
   git commit -m "Initial commit"
   git push -u origin master
   ```

### Step 1: Set Up GitLab CI/CD

2. **Create a `.gitlab-ci.yml` File**

   In the root of your project, create a `.gitlab-ci.yml` file:

   ```yaml
   stages:
     - build
     - deploy

   build:
     image: node:14
     stage: build
     script:
       - npm install

   deploy:
     image: node:14
     stage: deploy
     script:
       - ssh user@your_server_ip 'cd my-node-app && git pull origin master && npm install && pm2 restart my-node-app || pm2 start app.js --name my-node-app'
     only:
       - main
   ```

### Step 2: Set Up Your Ubuntu Server

3. **Prepare Your Server**

   Update your server:

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

   Install Node.js and npm:

   ```bash
   curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
   sudo apt install -y nodejs
   ```

   Install PM2 for process management:

   ```bash
   sudo npm install -g pm2
   ```

   Clone your GitLab repository to your server:

   ```bash
   git clone <YOUR_GITLAB_REPO_URL> /path/to/your/app
   cd /path/to/your/app
   npm install
   pm2 start app.js --name my-node-app
   ```

### Step 3: Configure Nginx

4. **Set Up Nginx**

   Install Nginx:

   ```bash
   sudo apt install nginx -y
   ```

   Create a new Nginx configuration file:

   ```bash
   sudo nano /etc/nginx/sites-available/my-node-app
   ```

   Add the following configuration:

   ```nginx
   server {
       listen 80;

       server_name your_domain_or_ip;

       location / {
           proxy_pass http://localhost:3000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

   Enable the new configuration:

   ```bash
   sudo ln -s /etc/nginx/sites-available/my-node-app /etc/nginx/sites-enabled/
   ```

   Test and restart Nginx:

   ```bash
   sudo nginx -t
   sudo systemctl restart nginx
   ```

### Step 4: Set Up SSH Access

5. **Configure SSH Access**

   Generate SSH keys (if you haven't already):

   ```bash
   ssh-keygen -t rsa -b 2048
   ```
   ```bash
   cat id_rsa.pub >> ~/.ssh/authorized_key
   ```

   Add the public key to your server:

   ```bash
   ssh-copy-id user@your_server_ip
   ```

   Add your private key to GitLab:
   - Go to your GitLab project, then **Settings > CI / CD > Variables** and add:
     - `SSH_PRIVATE_KEY`: Your private key contents.

   Modify your `.gitlab-ci.yml` to include SSH setup:

   ```yaml
   before_script:
     - apt-get update -y
     - apt-get install -y openssh-client
     - eval $(ssh-agent -s)
     - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
     - mkdir -p ~/.ssh
     - chmod 700 ~/.ssh
     - ssh-keyscan your_server_ip >> ~/.ssh/known_hosts
   ```

### Conclusion

Now, when you push to the `master` branch of your GitLab repository, the CI/CD pipeline will automatically build and deploy the latest changes to your Ubuntu server, with Nginx serving as the reverse proxy. Make sure that your server firewall allows traffic on port 80 (HTTP). 

### Example Testing

You can test your setup by making changes to `app.js`, committing, and pushing to the `master` branch:

```bash
echo "New Content" >> app.js
git add app.js
git commit -m "Updated message"
git push origin master
```

After a successful push, visit your server's IP or domain to see the changes reflected.
