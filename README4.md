
Setting up a continuous deployment (CD) pipeline for a full-stack React project using GitLab CI/CD on an Ubuntu server involves several steps, including configuring your back end and front end, setting up a CI/CD pipeline, and ensuring your server is properly configured. Below is a detailed process.

### Prerequisites

1. **Ubuntu Server**: Ensure you have an Ubuntu server.
2. **Node.js and npm**: Install Node.js and npm if not already installed:
   ```bash
   sudo apt update
   sudo apt install nodejs npm
   ```
3. **MongoDB**: If your application uses MongoDB, install it:
   ```bash
   sudo apt install mongodb
   ```
4. **GitLab Repository**: Create a GitLab account and repository for your project.
5. **SSH Access**: Ensure SSH access to your server from your local machine and GitLab.

### Step 1: Prepare Your Full-Stack React Project

1. **Create a Full-Stack React Project**:
   Structure your project to include both the front end and back end. For example:
   ```
   my-fullstack-app/
   ├── client/      # React frontend
   ├── server/      # Node.js backend
   └── package.json
   ```
   Initialize the client (React) and server (Node.js) applications:
   ```bash
   # For client
   cd client
   npx create-react-app .

   # For server
   cd ../server
   npm init -y
   npm install express mongoose cors dotenv
   ```

2. **Push to GitLab**:
   Initialize git, add your project to GitLab, and push:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin <your-gitlab-repo-url>
   git push -u origin master
   ```

### Step 2: Configure the Server

1. **Install Nginx**: For serving the React app.
   ```bash
   sudo apt install nginx
   ```
2. **Set Up Nginx**:
   Create an Nginx configuration file for your app:
   ```bash
   sudo nano /etc/nginx/sites-available/my-fullstack-app
   ```
   Add the following configuration:
   ```nginx
   server {
       listen 80;
       server_name your_domain_or_IP;

       location / {
           root /var/www/my-fullstack-app/client/build;
           try_files $uri $uri/ /index.html;
       }

       location /api/ {
           proxy_pass http://localhost:5000; # Adjust based on your server
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

   Link the configuration:
   ```bash
   sudo ln -s /etc/nginx/sites-available/my-fullstack-app /etc/nginx/sites-enabled/
   ```

   Test and restart Nginx:
   ```bash
   sudo nginx -t
   sudo systemctl restart nginx
   ```

3. **Set Up Directory for Build**:
   Create directories for your app:
   ```bash
   sudo mkdir -p /var/www/my-fullstack-app/client/build
   ```

4. **Start Your Node.js Server**:
   Create a basic Express server in the `server` directory (e.g., `server.js`):
   ```javascript
   const express = require('express');
   const cors = require('cors');
   const mongoose = require('mongoose');
   require('dotenv').config();

   const app = express();
   app.use(cors());
   app.use(express.json());

   // Example route
   app.get('/api/health', (req, res) => res.send('OK'));

   const PORT = process.env.PORT || 5000;
   app.listen(PORT, () => {
       console.log(`Server running on port ${PORT}`);
   });

   // Connect to MongoDB if applicable
   mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true });
   ```

5. **Run Your Server**:
   Start the server using a process manager like PM2:
   ```bash
   npm install -g pm2
   pm2 start server.js
   pm2 startup
   pm2 save
   ```

### Step 3: Create a GitLab CI/CD Pipeline

1. **Create `.gitlab-ci.yml`**:
   In the root of your project, create a file named `.gitlab-ci.yml` with the following content:
   ```yaml
   image: node:14

   stages:
     - build
     - deploy

   cache:
     paths:
       - client/node_modules/
       - server/node_modules/

   build_client:
     stage: build
     working_directory: client
     script:
       - npm install
       - npm run build
     artifacts:
       paths:
         - build/

   build_server:
     stage: build
     working_directory: server
     script:
       - npm install

   deploy:
     stage: deploy
     script:
       - ssh user@your_server_ip "rm -rf /var/www/my-fullstack-app/client/build/*"
       - scp -r client/build/* user@your_server_ip:/var/www/my-fullstack-app/client/build/
       - ssh user@your_server_ip "pm2 restart server" # Restart the server
     only:
       - master
   ```

   Replace `user` and `your_server_ip` with your server's details.

2. **Add SSH Key**:
   Generate an SSH key on your local machine if you don’t have one:
   ```bash
   ssh-keygen -t rsa -b 2048
   ```
   Add the public key to your server’s `~/.ssh/authorized_keys`.

3. **Add SSH Key to GitLab**:
   Go to your GitLab project settings, navigate to CI/CD > Variables, and add:
   - **Key**: `SSH_PRIVATE_KEY`
   - **Value**: (content of `~/.ssh/id_rsa`)

### Step 4: Update the Pipeline to Use SSH Key

Modify the `deploy` section in `.gitlab-ci.yml` to use the SSH key:
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
    - ssh user@your_server_ip "rm -rf /var/www/my-fullstack-app/client/build/*"
    - scp -r client/build/* user@your_server_ip:/var/www/my-fullstack-app/client/build/
    - ssh user@your_server_ip "pm2 restart server"
  only:
    - master
```

### Step 5: Commit and Push Changes

1. Commit your changes:
   ```bash
   git add .gitlab-ci.yml
   git commit -m "Add CI/CD pipeline"
   git push origin master
   ```

### Step 6: Monitor the Pipeline

1. Go to your GitLab repository, navigate to CI/CD > Pipelines, and monitor the progress.
2. After a successful build and deployment, access your full-stack application through your server's domain or IP.

### Conclusion

You have successfully set up a continuous deployment pipeline for your full-stack React project using GitLab CI/CD on Ubuntu. This pipeline will automatically build and deploy both your client and server whenever changes are pushed to the `master` branch.
