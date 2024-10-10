
Here's a detailed guide to deploy a full-stack React application (with a Node.js backend) to AWS EKS (Elastic Kubernetes Service) using GitHub Actions, specifically for an Ubuntu server setup.

### Prerequisites

1. **AWS Account**: Create an AWS account.
2. **AWS CLI**: Install and configure AWS CLI:
   ```bash
   sudo apt-get install awscli
   aws configure
   ```
3. **kubectl**: Install `kubectl` for managing Kubernetes clusters:
   ```bash
   sudo apt-get install -y kubectl
   ```
4. **eksctl**: Install `eksctl` for creating EKS clusters:
   ```bash
   curl --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp
   sudo mv /tmp/eksctl /usr/local/bin
   ```
5. **Docker**: Install Docker to build container images:
   ```bash
   sudo apt-get install docker.io
   ```
6. **GitHub Account**: Have a GitHub repository set up for your application.

### Step 1: Create a Full-Stack React Application

#### Backend (Node.js)

1. **Set Up the Backend**:
   ```bash
   mkdir fullstack-app
   cd fullstack-app
   mkdir backend
   cd backend
   npm init -y
   npm install express cors body-parser
   ```

2. **Create `index.js`**:
   ```javascript
   const express = require('express');
   const cors = require('cors');

   const app = express();
   app.use(cors());
   app.use(express.json());

   app.get('/api', (req, res) => {
       res.json({ message: 'Hello from Node.js!' });
   });

   const PORT = process.env.PORT || 5000;
   app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
   ```

#### Frontend (React)

1. **Set Up the Frontend**:
   ```bash
   cd ..
   npx create-react-app frontend
   cd frontend
   ```

2. **Modify `src/App.js`**:
   ```javascript
   import React, { useEffect, useState } from 'react';

   function App() {
       const [message, setMessage] = useState('');

       useEffect(() => {
           fetch('http://backend:5000/api')
               .then(response => response.json())
               .then(data => setMessage(data.message));
       }, []);

       return <div>{message}</div>;
   }

   export default App;
   ```

### Step 2: Dockerize the Applications

#### Backend Dockerfile

1. **Create Dockerfile in Backend**:
   ```Dockerfile
   # backend/Dockerfile
   FROM node:14
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   EXPOSE 5000
   CMD ["node", "index.js"]
   ```

#### Frontend Dockerfile

1. **Create Dockerfile in Frontend**:
   ```Dockerfile
   # frontend/Dockerfile
   FROM node:14 AS build
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   RUN npm run build

   FROM nginx:alpine
   COPY --from=build /app/build /usr/share/nginx/html
   EXPOSE 80
   CMD ["nginx", "-g", "daemon off;"]
   ```

### Step 3: Create Kubernetes Cluster on EKS

1. **Create EKS Cluster**:
   ```bash
   eksctl create cluster --name my-cluster --region us-west-2 --nodegroup-name my-nodes --nodes 2
   ```

2. **Configure kubectl**:
   ```bash
   aws eks --region us-west-2 update-kubeconfig --name my-cluster
   ```

### Step 4: Create Kubernetes Deployment and Service Files

1. **Create a Kubernetes Config Directory**:
   ```bash
   mkdir kubernetes
   ```

2. **Create `deployment.yaml`**:
   ```yaml
   # kubernetes/deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: backend
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: backend
     template:
       metadata:
         labels:
           app: backend
       spec:
         containers:
         - name: backend
           image: <account-id>.dkr.ecr.<region>.amazonaws.com/my-backend:latest
           ports:
           - containerPort: 5000
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: frontend
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: frontend
     template:
       metadata:
         labels:
           app: frontend
       spec:
         containers:
         - name: frontend
           image: <account-id>.dkr.ecr.<region>.amazonaws.com/my-frontend:latest
           ports:
           - containerPort: 80
   ```

3. **Create `service.yaml`**:
   ```yaml
   # kubernetes/service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: backend
   spec:
     type: ClusterIP
     ports:
     - port: 5000
       targetPort: 5000
     selector:
       app: backend
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: frontend
   spec:
     type: LoadBalancer
     ports:
     - port: 80
       targetPort: 80
     selector:
       app: frontend
   ```

### Step 5: Set Up GitHub Actions

1. **Create `.github/workflows/deploy.yml`**:
   ```yaml
   name: Deploy to EKS

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

         - name: Set up Docker Buildx
           uses: docker/setup-buildx-action@v1

         - name: Log in to Amazon ECR
           uses: aws-actions/amazon-ecr-login@v1

         - name: Build and push Docker images
           run: |
             docker build -t my-backend ./backend
             docker tag my-backend:latest <account-id>.dkr.ecr.<region>.amazonaws.com/my-backend:latest
             docker push <account-id>.dkr.ecr.<region>.amazonaws.com/my-backend:latest

             docker build -t my-frontend ./frontend
             docker tag my-frontend:latest <account-id>.dkr.ecr.<region>.amazonaws.com/my-frontend:latest
             docker push <account-id>.dkr.ecr.<region>.amazonaws.com/my-frontend:latest

         - name: Deploy to EKS
           env:
             KUBECONFIG: ${{ secrets.KUBECONFIG }}
           run: |
             kubectl apply -f kubernetes/deployment.yaml
             kubectl apply -f kubernetes/service.yaml
   ```

### Step 6: Configure GitHub Secrets

1. **Add Secrets**:
   - Navigate to your GitHub repository settings and add the following secrets:
     - `AWS_ACCESS_KEY_ID`
     - `AWS_SECRET_ACCESS_KEY`
     - `KUBECONFIG` (base64 encoded kubeconfig file).

### Step 7: Test Your Deployment

1. **Push Changes to GitHub**:
   - Commit and push your changes to the `main` branch to trigger the GitHub Actions workflow.

2. **Access the Application**:
   - After deployment, find the LoadBalancer's external IP using:
     ```bash
     kubectl get services
     ```
   - Access your frontend application using the LoadBalancer's external IP.

### Conclusion

You have successfully deployed a full-stack React application (with a Node.js backend) to AWS EKS using GitHub Actions. Adjust the configurations and monitor your deployment with `kubectl` commands for any issues.





above examplw with node js application😉😉😁😄😃😔😀😄😁🥴🥴😁😄😄🔑😃🔑

Deploying a full-stack Node.js application to AWS EKS (Elastic Kubernetes Service) using GitHub Actions can be broken down into several detailed steps. This guide is specifically tailored for Ubuntu server users.

### Prerequisites

1. **AWS Account**: Ensure you have an AWS account.
2. **AWS CLI**: Install and configure the AWS CLI:
   ```bash
   sudo apt-get install awscli
   aws configure
   ```
3. **kubectl**: Install `kubectl` for managing Kubernetes:
   ```bash
   sudo apt-get install -y kubectl
   ```
4. **eksctl**: Install `eksctl` for creating EKS clusters:
   ```bash
   curl --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp
   sudo mv /tmp/eksctl /usr/local/bin
   ```
5. **Docker**: Install Docker to build container images:
   ```bash
   sudo apt-get install docker.io
   ```
6. **GitHub Account**: Set up a GitHub repository for your application.

### Step 1: Create a Simple Node.js Application

1. **Create the Application Structure**:
   ```bash
   mkdir fullstack-node-app
   cd fullstack-node-app
   mkdir backend
   cd backend
   npm init -y
   npm install express cors body-parser
   ```

2. **Create `index.js`**:
   ```javascript
   const express = require('express');
   const cors = require('cors');

   const app = express();
   app.use(cors());
   app.use(express.json());

   app.get('/api', (req, res) => {
       res.json({ message: 'Hello from Node.js!' });
   });

   const PORT = process.env.PORT || 5000;
   app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
   ```

### Step 2: Dockerize the Node.js Application

1. **Create a Dockerfile**:
   ```bash
   touch Dockerfile
   ```

2. **Edit the Dockerfile**:
   ```Dockerfile
   # Dockerfile
   FROM node:14
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   EXPOSE 5000
   CMD ["node", "index.js"]
   ```

### Step 3: Create a Kubernetes Cluster on EKS

1. **Create the EKS Cluster**:
   ```bash
   eksctl create cluster --name my-cluster --region us-west-2 --nodegroup-name my-nodes --nodes 2
   ```

2. **Configure kubectl**:
   ```bash
   aws eks --region us-west-2 update-kubeconfig --name my-cluster
   ```

### Step 4: Create Kubernetes Deployment and Service Files

1. **Create a Kubernetes Config Directory**:
   ```bash
   mkdir kubernetes
   cd kubernetes
   ```

2. **Create `deployment.yaml`**:
   ```yaml
   # deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: backend
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: backend
     template:
       metadata:
         labels:
           app: backend
       spec:
         containers:
         - name: backend
           image: <account-id>.dkr.ecr.<region>.amazonaws.com/my-backend:latest
           ports:
           - containerPort: 5000
   ```

3. **Create `service.yaml`**:
   ```yaml
   # service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: backend
   spec:
     type: LoadBalancer
     ports:
     - port: 80
       targetPort: 5000
     selector:
       app: backend
   ```

### Step 5: Set Up GitHub Actions

1. **Create the Workflow Directory**:
   ```bash
   mkdir -p .github/workflows
   ```

2. **Create `deploy.yml`**:
   ```yaml
   name: Deploy to EKS

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

         - name: Set up Docker Buildx
           uses: docker/setup-buildx-action@v1

         - name: Log in to Amazon ECR
           uses: aws-actions/amazon-ecr-login@v1

         - name: Build and push Docker image
           run: |
             docker build -t my-backend ./backend
             docker tag my-backend:latest <account-id>.dkr.ecr.<region>.amazonaws.com/my-backend:latest
             docker push <account-id>.dkr.ecr.<region>.amazonaws.com/my-backend:latest

         - name: Deploy to EKS
           env:
             KUBECONFIG: ${{ secrets.KUBECONFIG }}
           run: |
             kubectl apply -f kubernetes/deployment.yaml
             kubectl apply -f kubernetes/service.yaml
   ```

### Step 6: Configure GitHub Secrets

1. **Add Secrets**:
   - Go to your GitHub repository settings and add the following secrets:
     - `AWS_ACCESS_KEY_ID`
     - `AWS_SECRET_ACCESS_KEY`
     - `KUBECONFIG` (base64 encoded kubeconfig file).

### Step 7: Build and Deploy the Application

1. **Push Changes to GitHub**:
   - Commit and push your code to the `main` branch to trigger the GitHub Actions workflow.

2. **Monitor the Deployment**:
   - After deployment, find the LoadBalancer's external IP:
     ```bash
     kubectl get services
     ```
   - Access your Node.js application using the LoadBalancer's external IP.

### Conclusion

You have successfully deployed a full-stack Node.js application to AWS EKS using GitHub Actions. Adjust your application and configurations as needed, and monitor your deployment using `kubectl` for any troubleshooting.
