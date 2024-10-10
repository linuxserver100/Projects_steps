
Configuring GitLab CI/CD for Docker with Self-signed cert gives x509



Here's a detailed guide to configuring GitLab CI/CD for a web application project using Docker with a self-signed certificate on an Ubuntu 22.04 LTS server. This guide will help you avoid x509 certificate errors during your CI/CD pipeline execution.

### Step 1: Generate a Self-Signed Certificate

1. **Install OpenSSL**:
   Ensure OpenSSL is installed:
   ```bash
   sudo apt update
   sudo apt install openssl
   ```

2. **Create a Self-Signed Certificate**:
   Generate the certificate and key:
   ```bash
   openssl req -x509 -newkey rsa:4096 -keyout myregistry.key -out myregistry.crt -days 365 -nodes -subj "/CN=myregistry.com"
   ```

3. **Set Up Certificate Directory for Docker**:
   Create a directory for the Docker certificate:
   ```bash
   sudo mkdir -p /etc/docker/certs.d/myregistry.com
   ```

4. **Copy the Certificate**:
   Move the certificate to the appropriate directory:
   ```bash
   sudo cp myregistry.crt /etc/docker/certs.d/myregistry.com/ca.crt
   ```

5. **Restart Docker**:
   Restart the Docker service to apply the new certificate:
   ```bash
   sudo systemctl restart docker
   ```

### Step 2: Install GitLab Runner

1. **Install GitLab Runner**:
   If not already installed, use the following commands:
   ```bash
   curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
   sudo apt install gitlab-runner
   ```

### Step 3: Configure GitLab Runner with the Self-Signed Certificate

1. **Copy the Certificate to the GitLab Runner**:
   The GitLab Runner also needs access to the self-signed certificate:
   ```bash
   sudo mkdir -p /etc/docker/certs.d/myregistry.com
   sudo cp myregistry.crt /etc/docker/certs.d/myregistry.com/ca.crt
   ```

2. **Restart GitLab Runner**:
   Restart the GitLab Runner service:
   ```bash
   sudo gitlab-runner restart
   ```

### Step 4: Create a Sample Web Application

1. **Create a Simple Web App**:
   For demonstration, create a simple web app in a directory, e.g., `mywebapp`. Inside this directory, create the following files:

   **Dockerfile**:
   ```dockerfile
   # Use the official Node.js image
   FROM node:14

   # Set the working directory
   WORKDIR /usr/src/app

   # Copy package.json and install dependencies
   COPY package*.json ./
   RUN npm install

   # Copy the application code
   COPY . .

   # Expose the app port
   EXPOSE 3000

   # Start the application
   CMD ["npm", "start"]
   ```

   **package.json**:
   ```json
   {
     "name": "mywebapp",
     "version": "1.0.0",
     "description": "A simple web application",
     "main": "index.js",
     "scripts": {
       "start": "node index.js"
     },
     "dependencies": {
       "express": "^4.17.1"
     }
   }
   ```

   **index.js**:
   ```javascript
   const express = require('express');
   const app = express();
   const PORT = 3000;

   app.get('/', (req, res) => {
     res.send('Hello World from my web app!');
   });

   app.listen(PORT, () => {
     console.log(`Server running on http://localhost:${PORT}`);
   });
   ```

2. **Build the Docker Image Locally** (optional):
   ```bash
   docker build -t myregistry.com/mywebapp:latest .
   ```

### Step 5: Configure Your GitLab CI/CD Pipeline

1. **Create/Edit `.gitlab-ci.yml`**:
   In your project repository, create or modify the `.gitlab-ci.yml` file as follows:

   ```yaml
   stages:
     - build
     - deploy

   build:
     stage: build
     image: docker:latest
     services:
       - docker:dind
     variables:
       DOCKER_TLS_CERTDIR: ""
     script:
       - echo "Building Docker image..."
       - docker build -t myregistry.com/mywebapp:latest .
       - echo "$CI_REGISTRY_PASSWORD" | docker login myregistry.com -u "$CI_REGISTRY_USER" --password-stdin
       - echo "Pushing image to registry..."
       - docker push myregistry.com/mywebapp:latest

   deploy:
     stage: deploy
     script:
       - echo "Deploying application..."
       # Here you would add deployment commands, e.g., to Kubernetes or another platform
   ```

### Step 6: Set Up GitLab CI/CD Variables

1. **Add Variables in GitLab**:
   Navigate to your GitLab project:
   - Go to **Settings > CI/CD > Variables**.
   - Add the following variables:
     - `CI_REGISTRY_USER`: Your Docker registry username.
     - `CI_REGISTRY_PASSWORD`: Your Docker registry password.

### Step 7: Run Your Pipeline

1. **Trigger the Pipeline**:
   Push a change to your repository or manually trigger a pipeline from the GitLab interface.

2. **Monitor Job Logs**:
   Check the job logs for any x509 certificate errors. If configured correctly, the build and push should complete without errors.

### Example Log Output

Upon successful execution, you should see output like this:

```
Building Docker image...
Sending build context to Docker daemon  123.45kB
Step 1/4 : FROM node:14
 ---> 123456789abc
Step 2/4 : WORKDIR /usr/src/app
 ---> Using cache
 ---> 987654321def
Step 3/4 : COPY package*.json ./
 ---> Using cache
 ---> 543216789abc
Step 4/4 : COPY . .
 ---> Using cache
 ---> 987654321def
Successfully built 987654321def
Logging into Docker registry...
Login Succeeded
Pushing image to registry...
The push refers to repository [myregistry.com/mywebapp]
...
latest: digest: sha256:abcd1234 size: 1234
```

### Additional Troubleshooting Tips

- **Verify Certificate Installation**: Ensure the certificate is correctly placed in `/etc/docker/certs.d/myregistry.com/ca.crt`.
- **Check GitLab Runner Logs**: If issues arise, examine the GitLab Runner logs for additional errors.
- **Docker Daemon Logs**: Review Docker daemon logs if you encounter problems connecting to the registry.

By following these steps, you should have a fully functional GitLab CI/CD pipeline that works with a Docker registry secured by a self-signed certificate on an Ubuntu 22.04 LTS server.
