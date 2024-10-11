
Setting up a Continuous Deployment (CD) pipeline using GitLab CI/CD for a Node.js application on Ubuntu involves several key steps. Below is a detailed guide with an example Node.js application.

### Prerequisites

1. **GitLab Account**: Create a GitLab account and a new project.
2. **Ubuntu Server**: Access to an Ubuntu server for deployment.
3. **SSH Access**: Ensure SSH access to your server.
4. **Node.js and npm**: Make sure Node.js and npm are installed on your server.
5. **GitLab Runner**: Optionally set up a GitLab Runner if needed.

### Step 1: Create a Simple Node.js Application

1. **Set up your Node.js project**:

```bash
mkdir my-node-app
cd my-node-app
npm init -y
npm install express
```

2. **Create a simple Express server**. In `index.js`, add:

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

Visit `http://localhost:3000` in your browser to ensure it works.

### Step 2: Push Your Code to GitLab

1. **Initialize Git and push your application**:

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin <your-gitlab-repo-url>
git push -u origin master
```

### Step 3: Create a `.gitlab-ci.yml` File

In the root of your project, create a file named `.gitlab-ci.yml`. This file defines your CI/CD pipeline:

```yaml
stages:
  - build
  - deploy

build:
  image: node:14
  stage: build
  script:
    - npm install
    - npm test # Optional: run tests if you have them
  artifacts:
    paths:
      - node_modules/

deploy:
  stage: deploy
  image: ruby:2.6
  script:
    - apt-get update -y
    - apt-get install -y sshpass
    - sshpass -p $DEPLOY_PASSWORD ssh -o StrictHostKeyChecking=no $DEPLOY_USER@$DEPLOY_HOST "cd my-node-app && git pull origin main && npm install && pm2 restart my-node-app || pm2 start index.js --name my-node-app"
  only:
    - main
```

### Step 4: Configure CI/CD Variables

1. Go to your GitLab project.
2. Navigate to **Settings > CI / CD > Variables**.
3. Add the following variables:
   - `DEPLOY_USER`: Your SSH username.
   - `DEPLOY_PASSWORD`: Your SSH password.
   - `DEPLOY_HOST`: The IP address or hostname of your Ubuntu server.

### Step 5: Set Up Your Ubuntu Server

1. **Install Node.js and PM2**:

```bash
sudo apt update
sudo apt install -y nodejs npm
sudo npm install -g pm2
```

2. **Clone Your Repository**:

```bash
cd /path/to/your/app
git clone <your-gitlab-repo-url> .
```

3. **Start Your Application**:

```bash
pm2 start index.js --name your-app-name
```

### Step 6: Trigger the Pipeline

1. Commit and push a change to your GitLab repository.
2. Go to **CI/CD > Pipelines** in your GitLab project to monitor the progress.
3. If everything is set up correctly, your application should be deployed automatically.

### Step 7: Verify Deployment

1. Access your application in a web browser using your server's IP address and port (default: 3000).
2. Ensure that it responds with "Hello, World!"

### Conclusion

You now have a Continuous Deployment pipeline set up for a Node.js application using GitLab CI/CD on Ubuntu. You can expand this setup by adding testing, notifications, and more advanced deployment strategies.






😔😔😔😁😔💯😁💯😔💯💯👍🤩🤩🤩🤩🔑😄👍👍😔😔😔👍🤩🔑💯💯💯😁🤩💯💯😁👍👍🔑😄😁💯👍🔑🔑😁💯😔😔







Setting up a Continuous Deployment (CD) pipeline for a full-stack React application using GitLab CI/CD on Ubuntu involves several steps, including configuring your GitLab repository, creating a `.gitlab-ci.yml` file, and deploying your application to a server. Below is a detailed guide with an example.

### Prerequisites

1. **GitLab Account**: Create a GitLab account and a new project.
2. **Ubuntu Server**: Have access to an Ubuntu server for deployment.
3. **SSH Access**: Ensure you have SSH access to your server.
4. **Node.js and npm**: Ensure Node.js and npm are installed on your server.
5. **GitLab Runner**: You may optionally set up a GitLab Runner.

### Step 1: Create a Full-Stack React Application

For this example, we'll create a simple full-stack application with a React front-end and an Express back-end.

1. **Set up your project structure**:

```bash
mkdir my-fullstack-app
cd my-fullstack-app
npx create-react-app client
mkdir server
cd server
npm init -y
npm install express cors
```

2. **Create a basic Express server**. In `server/index.js`, add:

```javascript
const express = require('express');
const cors = require('cors');
const app = express();
const PORT = process.env.PORT || 5000;

app.use(cors());
app.use(express.json());

app.get('/api/message', (req, res) => {
  res.json({ message: 'Hello from the server!' });
});

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

3. **Modify the client to call the API**. In `client/src/App.js`, update the `useEffect` hook:

```javascript
import React, { useEffect, useState } from 'react';

function App() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    fetch('http://localhost:5000/api/message')
      .then((res) => res.json())
      .then((data) => setMessage(data.message));
  }, []);

  return <div>{message || 'Loading...'}</div>;
}

export default App;
```

### Step 2: Push Your Code to GitLab

1. Initialize Git and push your application to GitLab:

```bash
cd my-fullstack-app
git init
git add .
git commit -m "Initial commit"
git remote add origin <your-gitlab-repo-url>
git push -u origin master
```

### Step 3: Create a `.gitlab-ci.yml` File

In the root of your project, create a file named `.gitlab-ci.yml`. This file defines your CI/CD pipeline:

```yaml
stages:
  - build
  - deploy

build_client:
  image: node:14
  stage: build
  script:
    - cd client
    - npm install
    - npm run build
  artifacts:
    paths:
      - client/build

build_server:
  image: node:14
  stage: build
  script:
    - cd server
    - npm install

deploy:
  stage: deploy
  image: ruby:2.6
  script:
    - apt-get update -y
    - apt-get install -y sshpass
    - sshpass -p $DEPLOY_PASSWORD ssh -o StrictHostKeyChecking=no $DEPLOY_USER@$DEPLOY_HOST "cd my-fullstack-app && git pull origin main && cd server && npm install && pm2 restart my-fullstack-app || npm start & cd .. && if [ -d client ]; then cd client && npm install && npm run build && nm2 restart my-fullstack-app; else echo 'Client directory not found. Skipping client-side deployment'; \fi"
  only:
    - master
```

### Step 4: Configure CI/CD Variables

1. Go to your GitLab project.
2. Navigate to **Settings > CI/CD > Variables**.
3. Add the following variables:
   - `DEPLOY_USER`: Your SSH username.
   - `DEPLOY_PASSWORD`: Your SSH password.
   - `DEPLOY_HOST`: The IP address or hostname of your Ubuntu server.

### Step 5: Set Up Your Ubuntu Server

1. **Install Node.js and PM2**:

```bash
sudo apt update
sudo apt install -y nodejs npm
sudo npm install -g pm2
```

2. **Clone Your Repository**:

```bash
cd /path/to/your/app
git clone <your-gitlab-repo-url> .
```

3. **Start Your Server**:

```bash
cd server
npm install
pm2 start index.js --name your-server-name
```

4. **Set Up Static File Serving for React**:

```bash
cd client
npm install
npm run build
pm2 serve build 3000 --name your-client-name
```

### Step 6: Trigger the Pipeline

1. Commit and push a change to your GitLab repository.
2. Go to **CI/CD > Pipelines** in your GitLab project to monitor the progress.
3. If everything is set up correctly, your application should be deployed automatically.

### Step 7: Verify Deployment

1. Access your application in a web browser using your server's IP address and the ports configured (default: 3000 for React and 5000 for Express).
2. Ensure that the application responds correctly.

### Conclusion

You now have a Continuous Deployment pipeline set up for a full-stack React application using GitLab CI/CD on Ubuntu. You can further enhance this setup with testing, monitoring, and more sophisticated deployment strategies.




