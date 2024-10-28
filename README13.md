🥹
Creating a full-stack React application with backend support, CI/CD pipeline, rollback functionality, and server deployment using Apache with SSH key-based authentication requires a systematic approach. Here’s a high-level guide to build this setup.

---

### 1. **Project Setup**

   - **Frontend**: React (create a React app using `npx create-react-app frontend`).
   - **Backend**: Node.js + Express (create an `api` folder with Express).
   - **Directory Structure**:
     ```
     my-app/
     ├── frontend/
     └── backend/
     └── .github/workflows/
     └── scripts/
     └── deploy/
     ```

---

### 2. **React (Frontend)**

   - **Initialize React App**:
     ```bash
     cd my-app/frontend
     npx create-react-app .
     ```

   - **Proxy Setup for Local Development**:
     Set up a proxy in the `package.json` file of the React app:
     ```json
     "proxy": "http://localhost:5000"
     ```

---

### 3. **Express Backend**

   - **Initialize Backend**:
     ```bash
     cd ../backend
     npm init -y
     npm install express cors dotenv
     ```

   - **Basic Server Configuration (backend/index.js)**:
     ```javascript
     const express = require("express");
     const cors = require("cors");

     const app = express();
     app.use(cors());
     app.use(express.json());

     app.get("/api", (req, res) => {
       res.json({ message: "Hello from the backend!" });
     });

     const PORT = process.env.PORT || 5000;
     app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
     ```

---

### 4. **Apache Configuration & Deployment**

   - **Server Setup**:
     - Install Node.js, Apache, and necessary dependencies on your server.
     - Enable Apache modules if needed for reverse proxy:
       ```bash
       sudo a2enmod proxy proxy_http
       ```

   - **Apache Virtual Host for Node.js App**:
     Configure the Apache site to proxy requests to the backend and serve the frontend:
     ```apache
     <VirtualHost *:80>
       ServerName myapp.com

       ProxyPreserveHost On
       ProxyPass /api http://localhost:5000/api
       ProxyPassReverse /api http://localhost:5000/api

       DocumentRoot /var/www/myapp/frontend/build
       <Directory "/var/www/myapp/frontend/build">
         Options Indexes FollowSymLinks
         AllowOverride All
         Require all granted
       </Directory>
     </VirtualHost>
     ```

---

### 5. **CI/CD Pipeline with GitHub Actions**

   - **Create GitHub Actions Workflow**:
     In `.github/workflows/deploy.yml`:
     ```yaml
     name: CI/CD Pipeline

     on:
       push:
         branches:
           - main

     jobs:
       build:
         runs-on: ubuntu-latest

         steps:
           - name: Checkout code
             uses: actions/checkout@v2

           - name: Set up Node.js
             uses: actions/setup-node@v2
             with:
               node-version: '14'

           - name: Install frontend dependencies
             working-directory: ./frontend
             run: npm install

           - name: Build frontend
             working-directory: ./frontend
             run: npm run build

           - name: Install backend dependencies
             working-directory: ./backend
             run: npm install

           - name: Deploy to Server
             env:
               HOST: ${{ secrets.SERVER_HOST }}
               USER: ${{ secrets.SERVER_USER }}
               KEY: ${{ secrets.SSH_PRIVATE_KEY }}
             run: |
               ssh -i "$KEY" -o StrictHostKeyChecking=no $USER@$HOST 'bash -s' < ./scripts/deploy.sh
     ```

   - **Deployment Script (`scripts/deploy.sh`)**:
     ```bash
     #!/bin/bash

     # Define paths
     FRONTEND_PATH="/var/www/myapp/frontend"
     BACKEND_PATH="/var/www/myapp/backend"

     # Frontend deployment
     echo "Deploying frontend..."
     rm -rf $FRONTEND_PATH/build
     mv ~/frontend/build $FRONTEND_PATH

     # Backend deployment
     echo "Deploying backend..."
     pm2 restart server || pm2 start backend/index.js --name server
     ```

   - **SSH Key-Based Authentication**:
     1. Generate an SSH key pair and add the public key to the server.
     2. Store the private key in GitHub Secrets (`SSH_PRIVATE_KEY`) for secure use.

---

### 6. **Rollback Functionality**

   - **Implement Versioning in CI/CD**:
     Maintain backups of the previous deployment (e.g., `frontend/build` and `backend`) before overwriting.
   
   - **Create a Rollback Script**:
     ```bash
     #!/bin/bash
     echo "Rolling back..."
     cp -R /var/www/myapp/frontend/backup /var/www/myapp/frontend/build
     cp -R /var/www/myapp/backend/backup /var/www/myapp/backend
     pm2 restart server
     ```

   - **Automated Rollback**: Configure GitHub Actions to detect failed deployments and trigger this rollback script.

---

This setup provides a structured approach for building, deploying, and managing rollback functionality for your application using Apache with secure SSH-based deployments. Let me know if you’d like additional details on any step!

🥹☺️🤪😆🤪😆😘😆😝😆😛😆😂😍🙂😍😗😅😙😍😙😍😙😍😍😙😂😙😆😙😆😙😆😙😆😙😂😙😂😙😍😙😍😙😅😙😅🥴😃😃😙😅😙😍😙😂😙😂😙😂😙😍😙😅😙😅😙😅😙😅😅😙😍😙😙😍😍😙😂😙😆😙☺️😙☺️😙☺️🙂😂😙😂😙😍😙🙂😅😃🤪🙃🤪😔🤪🤩🤪😅🤪😍🤪😂🤪😆😗😆😗😆😗😆😗😍😗😍😗🤩😗🤩😗🤩😗🤩😗🤩🤪🤩🙂🤩🙂🤩🙂🤩🙂🤩🤩🙂🤩🙂🙂🙂

Creating a full-stack MERN application (MongoDB, Express, React, Node.js) with user authentication, a CI/CD pipeline using GitLab, and deploying to a server with Apache2 and SSH key-based authentication involves several steps. Below is a step-by-step guide:

### Step 1: Set Up the Project Structure

1. **Create Project Directory**:
   ```bash
   mkdir mern-auth-app
   cd mern-auth-app
   ```

2. **Initialize Backend**:
   ```bash
   mkdir backend
   cd backend
   npm init -y
   npm install express mongoose bcryptjs jsonwebtoken cors dotenv
   ```

3. **Initialize Frontend**:
   Open a new terminal or command prompt, then:
   ```bash
   cd ..
   npx create-react-app frontend
   cd frontend
   npm install axios react-router-dom
   ```

### Step 2: Build the Backend

1. **Create the Server**:
   In the `backend` directory, create an `index.js` file:

   ```javascript
   const express = require('express');
   const mongoose = require('mongoose');
   const cors = require('cors');
   const dotenv = require('dotenv');

   dotenv.config();
   const app = express();
   app.use(cors());
   app.use(express.json());

   mongoose.connect(process.env.MONGODB_URI, { useNewUrlParser: true, useUnifiedTopology: true })
       .then(() => console.log('MongoDB connected'))
       .catch(err => console.log(err));

   const PORT = process.env.PORT || 5000;
   app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
   ```

2. **Create User Model**:
   Create a `models` directory and a `User.js` file:

   ```javascript
   const mongoose = require('mongoose');

   const UserSchema = new mongoose.Schema({
       username: { type: String, required: true, unique: true },
       password: { type: String, required: true }
   });

   module.exports = mongoose.model('User', UserSchema);
   ```

3. **Implement Authentication Routes**:
   Create a `routes` directory and an `auth.js` file:

   ```javascript
   const router = require('express').Router();
   const User = require('../models/User');
   const bcrypt = require('bcryptjs');
   const jwt = require('jsonwebtoken');

   router.post('/register', async (req, res) => {
       const { username, password } = req.body;
       const hashedPassword = await bcrypt.hash(password, 10);
       const user = new User({ username, password: hashedPassword });
       await user.save();
       res.status(201).send('User registered');
   });

   router.post('/login', async (req, res) => {
       const { username, password } = req.body;
       const user = await User.findOne({ username });
       if (!user) return res.status(400).send('User not found');
       const isMatch = await bcrypt.compare(password, user.password);
       if (!isMatch) return res.status(400).send('Invalid credentials');
       const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET, { expiresIn: '1h' });
       res.json({ token });
   });

   module.exports = router;
   ```

4. **Connect Routes**:
   In `index.js`, connect the routes:

   ```javascript
   const authRoutes = require('./routes/auth');
   app.use('/api/auth', authRoutes);
   ```

5. **Environment Variables**:
   Create a `.env` file in the `backend` directory:

   ```
   MONGODB_URI=mongodb://<your_mongodb_uri>
   JWT_SECRET=<your_jwt_secret>
   ```

### Step 3: Build the Frontend

1. **Set Up Routing**:
   In `frontend/src`, create `pages` directory and create `Login.js` and `Register.js` files.

   **Login.js**:
   ```javascript
   import React, { useState } from 'react';
   import axios from 'axios';
   import { useHistory } from 'react-router-dom';

   const Login = () => {
       const [username, setUsername] = useState('');
       const [password, setPassword] = useState('');
       const history = useHistory();

       const handleLogin = async (e) => {
           e.preventDefault();
           try {
               const response = await axios.post('http://localhost:5000/api/auth/login', { username, password });
               localStorage.setItem('token', response.data.token);
               history.push('/dashboard');
           } catch (error) {
               console.error('Login failed', error);
           }
       };

       return (
           <form onSubmit={handleLogin}>
               <input type="text" value={username} onChange={(e) => setUsername(e.target.value)} placeholder="Username" required />
               <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Password" required />
               <button type="submit">Login</button>
           </form>
       );
   };

   export default Login;
   ```

   **Register.js**:
   ```javascript
   import React, { useState } from 'react';
   import axios from 'axios';

   const Register = () => {
       const [username, setUsername] = useState('');
       const [password, setPassword] = useState('');

       const handleRegister = async (e) => {
           e.preventDefault();
           await axios.post('http://localhost:5000/api/auth/register', { username, password });
           // Redirect to login or dashboard
       };

       return (
           <form onSubmit={handleRegister}>
               <input type="text" value={username} onChange={(e) => setUsername(e.target.value)} placeholder="Username" required />
               <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Password" required />
               <button type="submit">Register</button>
           </form>
       );
   };

   export default Register;
   ```

2. **Setup App Component**:
   Update `App.js` to use React Router:

   ```javascript
   import React from 'react';
   import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
   import Login from './pages/Login';
   import Register from './pages/Register';

   const App = () => {
       return (
           <Router>
               <Switch>
                   <Route path="/login" component={Login} />
                   <Route path="/register" component={Register} />
               </Switch>
           </Router>
       );
   };

   export default App;
   ```

### Step 4: CI/CD Setup with GitLab

1. **Create GitLab Repository**:
   Push your code to a new GitLab repository.

2. **Create `.gitlab-ci.yml`**:
   In the root of your project, create a `.gitlab-ci.yml` file:

   ```yaml
   image: node:14

   stages:
     - build
     - test
     - deploy

   build:
     stage: build
     script:
       - cd frontend
       - npm install
       - npm run build
       - cd ../backend
       - npm install
     artifacts:
       paths:
         - frontend/build

   test:
     stage: test
     script:
       - cd backend
       - npm test

   deploy:
     stage: deploy
     script:
       - apt-get update -y
       - apt-get install sshpass -y
       - sshpass -e scp -r frontend/build/* user@your_server:/var/www/html
       - ssh user@your_server 'cd /var/www/html && pm2 restart all'
     only:
       - main
   ```

   Replace `user@your_server` with your SSH credentials.

3. **Environment Variables**:
   In GitLab, navigate to **Settings > CI/CD > Variables** and add:
   - `SSH_PRIVATE_KEY`: Your private SSH key.

### Step 5: Deploying to Server with Apache2

1. **Setup Apache**:
   Install Apache on your server:

   ```bash
   sudo apt update
   sudo apt install apache2
   ```

2. **Enable Proxy**:
   Enable the required Apache modules:

   ```bash
   sudo a2enmod proxy
   sudo a2enmod proxy_http
   sudo a2enmod rewrite
   ```

3. **Configure Virtual Host**:
   Create a new configuration file:

   ```bash
   sudo nano /etc/apache2/sites-available/mern-auth-app.conf
   ```

   Add the following configuration:

   ```apache
   <VirtualHost *:80>
       ServerName your_domain_or_ip

       DocumentRoot /var/www/html
       <Directory /var/www/html>
           AllowOverride All
       </Directory>

       ProxyPass /api http://localhost:5000
       ProxyPassReverse /api http://localhost:5000
   </VirtualHost>
   ```

4. **Enable Site and Restart Apache**:
   ```bash
   sudo a2ensite mern-auth-app
   sudo systemctl restart apache2
   ```

### Step 6: Implement Rollback Functionality

1. **Store Previous Builds**:
   Modify the deploy stage in `.gitlab-ci.yml` to keep a backup of previous builds. Add to the deploy script:

   ```yaml
   - ssh user@your_server 'cp -r /var/www/html /var/www/html_backup'
   ```

2. **Rollback Command**:
   Add a rollback command that can be

 executed on your server to restore the previous version:

   ```bash
   sudo cp -r /var/www/html_backup/* /var/www/html/
   ```

### Conclusion

You now have a basic full-stack MERN application with login and registration, set up with a CI/CD pipeline using GitLab, and deployed to a server with Apache2. You also implemented rollback functionality in case of deployment issues. 

### Next Steps

- Secure your application (e.g., add HTTPS).
- Add more features (e.g., profile management).
- Optimize and test your application. 

Feel free to ask if you need more details on any step!



😀😃😄😅😄😅😁😁😍😆😂😄😂😁😁😄☺️😁☺️😁😄☺️😁☺️😁😊😄🥹😁🥹😄😛😝😄😁😊😁☺️😄☺️😁☺️😁☺️😁☺️😁☺️😁☺️😁☺️😁☺️😁☺️😄☺️😁☺️😁☺️😁☺️😁☺️😁☺️😁😁☺️😁☺️😁☺️😁☺️😁☺️😚☺️😚☺️😘☺️😆😊😘☺️😚☺️😚☺️😁☺️😘☺️😘☺️😘☺️😚☺️😚☺️😘☺️😆☺️😘☺️😘😘☺️😘☺️😘☺️😆☺️😆☺️😆☺️😘☺️😆☺️☺️😆☺️😆☺️😆☺️😆☺️😆☺️😆☺️😆☺️😆☺️Creating a full-stack MERN (MongoDB, Express, React, Node.js) application with user authentication (login and register), and deploying it with a CI/CD pipeline involves several steps. Below is a comprehensive guide to achieve this, covering both the backend and frontend setup, CI/CD deployment with GitHub Actions, and server configuration with Apache2 and SSH key-based authentication.

### Step 1: Set Up the Project Structure

1. **Create the Project Directory:**
   ```bash
   mkdir mern-auth-app
   cd mern-auth-app
   ```

2. **Initialize Git Repository:**
   ```bash
   git init
   ```

3. **Create Backend and Frontend Directories:**
   ```bash
   mkdir backend frontend
   ```

### Step 2: Backend Setup

1. **Navigate to the Backend Directory:**
   ```bash
   cd backend
   ```

2. **Initialize a Node.js Project:**
   ```bash
   npm init -y
   ```

3. **Install Required Packages:**
   ```bash
   npm install express mongoose bcryptjs jsonwebtoken cors dotenv
   ```

4. **Create the Following File Structure:**
   ```bash
   mkdir config models routes middleware
   touch server.js
   ```

5. **Set Up Environment Variables:**
   Create a `.env` file in the `backend` directory:
   ```
   MONGODB_URI=<your_mongodb_uri>
   JWT_SECRET=<your_jwt_secret>
   PORT=5000
   ```

6. **Create the Server:**
   In `server.js`, add the following code to set up the Express server:
   ```javascript
   const express = require('express');
   const mongoose = require('mongoose');
   const cors = require('cors');
   const dotenv = require('dotenv');

   dotenv.config();
   const app = express();
   const PORT = process.env.PORT || 5000;

   app.use(cors());
   app.use(express.json());

   mongoose.connect(process.env.MONGODB_URI, { useNewUrlParser: true, useUnifiedTopology: true })
       .then(() => console.log('MongoDB connected'))
       .catch(err => console.error(err));

   app.listen(PORT, () => {
       console.log(`Server is running on http://localhost:${PORT}`);
   });
   ```

### Step 3: Create User Model and Routes

1. **Create User Model:**
   In `models/User.js`:
   ```javascript
   const mongoose = require('mongoose');

   const userSchema = new mongoose.Schema({
       username: { type: String, required: true, unique: true },
       password: { type: String, required: true },
   });

   module.exports = mongoose.model('User', userSchema);
   ```

2. **Create Authentication Routes:**
   In `routes/auth.js`:
   ```javascript
   const express = require('express');
   const bcrypt = require('bcryptjs');
   const jwt = require('jsonwebtoken');
   const User = require('../models/User');

   const router = express.Router();

   // Register
   router.post('/register', async (req, res) => {
       const { username, password } = req.body;
       const hashedPassword = await bcrypt.hash(password, 10);
       const user = new User({ username, password: hashedPassword });

       try {
           await user.save();
           res.status(201).send('User registered');
       } catch (error) {
           res.status(400).send('Error registering user');
       }
   });

   // Login
   router.post('/login', async (req, res) => {
       const { username, password } = req.body;
       const user = await User.findOne({ username });

       if (user && (await bcrypt.compare(password, user.password))) {
           const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET, { expiresIn: '1h' });
           res.json({ token });
       } else {
           res.status(400).send('Invalid credentials');
       }
   });

   module.exports = router;
   ```

3. **Use Routes in `server.js`:**
   ```javascript
   const authRoutes = require('./routes/auth');
   app.use('/api/auth', authRoutes);
   ```

### Step 4: Frontend Setup

1. **Navigate to the Frontend Directory:**
   ```bash
   cd ../frontend
   ```

2. **Create a React App:**
   ```bash
   npx create-react-app .
   ```

3. **Install Axios for API Calls:**
   ```bash
   npm install axios react-router-dom
   ```

4. **Set Up Basic Routing:**
   In `src/App.js`, add the following:
   ```javascript
   import React from 'react';
   import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
   import Register from './components/Register';
   import Login from './components/Login';

   const App = () => {
       return (
           <Router>
               <Switch>
                   <Route path="/register" component={Register} />
                   <Route path="/login" component={Login} />
               </Switch>
           </Router>
       );
   };

   export default App;
   ```

5. **Create Register and Login Components:**
   In `src/components/Register.js`:
   ```javascript
   import React, { useState } from 'react';
   import axios from 'axios';

   const Register = () => {
       const [username, setUsername] = useState('');
       const [password, setPassword] = useState('');

       const handleSubmit = async (e) => {
           e.preventDefault();
           await axios.post('http://localhost:5000/api/auth/register', { username, password });
           alert('User registered');
       };

       return (
           <form onSubmit={handleSubmit}>
               <input type="text" placeholder="Username" value={username} onChange={(e) => setUsername(e.target.value)} />
               <input type="password" placeholder="Password" value={password} onChange={(e) => setPassword(e.target.value)} />
               <button type="submit">Register</button>
           </form>
       );
   };

   export default Register;
   ```

   In `src/components/Login.js`:
   ```javascript
   import React, { useState } from 'react';
   import axios from 'axios';

   const Login = () => {
       const [username, setUsername] = useState('');
       const [password, setPassword] = useState('');

       const handleSubmit = async (e) => {
           e.preventDefault();
           const response = await axios.post('http://localhost:5000/api/auth/login', { username, password });
           localStorage.setItem('token', response.data.token);
           alert('Logged in');
       };

       return (
           <form onSubmit={handleSubmit}>
               <input type="text" placeholder="Username" value={username} onChange={(e) => setUsername(e.target.value)} />
               <input type="password" placeholder="Password" value={password} onChange={(e) => setPassword(e.target.value)} />
               <button type="submit">Login</button>
           </form>
       );
   };

   export default Login;
   ```

### Step 5: Configure GitHub Repository

1. **Push Code to GitHub:**
   ```bash
   git add .
   git commit -m "Initial commit"
   git remote add origin <your_github_repo_url>
   git push -u origin master
   ```

### Step 6: Set Up CI/CD with GitHub Actions

1. **Create a GitHub Actions Workflow:**
   In your GitHub repository, create a `.github/workflows/deploy.yml` file:
   ```yaml
   name: Deploy MERN App

   on:
     push:
       branches:
         - master

   jobs:
     build-and-deploy:
       runs-on: ubuntu-latest
       steps:
         - name: Checkout code
           uses: actions/checkout@v2

         - name: Set up Node.js
           uses: actions/setup-node@v2
           with:
             node-version: '14'

         - name: Install backend dependencies
           run: |
             cd backend
             npm install

         - name: Build frontend
           run: |
             cd frontend
             npm install
             npm run build

         - name: Deploy to Server
           env:
             SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
             SERVER_USER: ${{ secrets.SERVER_USER }}
             SERVER_IP: ${{ secrets.SERVER_IP }}
           run: |
             echo "$SSH_PRIVATE_KEY" > private_key
             chmod 600 private_key
             scp -i private_key -r ./frontend/build $SERVER_USER@$SERVER_IP:/var/www/html
             scp -i private_key -r ./backend $SERVER_USER@$SERVER_IP:/var/www/backend
             ssh -i private_key $SERVER_USER@$SERVER_IP 'pm2 restart backend' # assuming pm2 is used for process management
   ```

2. **Add Secrets to GitHub:**
   - Go to your GitHub repository > Settings > Secrets and Variables > Actions.
   - Add the following secrets:
     - `SSH_PRIVATE_KEY`: Your private SSH key for server access.
     - `SERVER_USER`: Your server username (e.g., `ubuntu`).
     - `SERVER_IP`: Your server's IP address.

### Step 7: Server Configuration with Apache2 and SSH Key-based Authentication

1. **Install Apache and Node.js on Your Server:**
   ```bash
   sudo apt update
   sudo apt install apache2
   sudo apt install nodejs npm
   ```

2. **

Enable Apache to Serve Your React App:**
   Create a virtual host for your React app:
   ```bash
   sudo nano /etc/apache2/sites-available/mern-app.conf
   ```
   Add the following configuration:
   ```apache
   <VirtualHost *:80>
       ServerAdmin webmaster@localhost
       DocumentRoot /var/www/html
       ErrorLog ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined

       <Directory /var/www/html>
           Options Indexes FollowSymLinks
           AllowOverride All
           Require all granted
       </Directory>
   </VirtualHost>
   ```

3. **Enable the Site and Restart Apache:**
   ```bash
   sudo a2ensite mern-app.conf
   sudo systemctl restart apache2
   ```

4. **Set Up SSH Key-based Authentication:**
   - Generate SSH keys on your local machine (if not already done):
     ```bash
     ssh-keygen -t rsa -b 4096
     ```
   - Copy the public key to your server:
     ```bash
     ssh-copy-id your_user@server_ip
     ```

5. **Test Your Application:**
   Visit your server's IP address in a browser to see the React app. 

### Step 8: Rollback Functionality

1. **Implement Rollback in CI/CD:**
   Modify your CI/CD workflow to keep previous builds and allow rollback:
   ```yaml
   - name: Backup Current Build
     run: ssh -i private_key $SERVER_USER@$SERVER_IP 'mv /var/www/html /var/www/html_backup_$(date +%Y%m%d%H%M%S)'
   ```

2. **Revert to Previous Build:**
   To rollback, you can run a command via SSH to rename the previous backup back to `html`:
   ```bash
   ssh -i private_key $SERVER_USER@$SERVER_IP 'mv /var/www/html_backup_* /var/www/html'
   ```

### Conclusion

This guide provides a step-by-step process to create a full-stack MERN application with user authentication, and deploy it using a CI/CD pipeline to a server with Apache2 and SSH key-based authentication. You can customize and enhance the application further based on your requirements.

😙😙😃🥴😃😚😃😂😂🙃😂☺️😂😗☺️😗🤩😌😃😁😛😁🤪🥲😁🥲🥹👇🥹🔐😆🔐😁🔐😆🔐😆😗🔐😂👇😆🥲😆🥲😗🥲🤪🥲😆🥲😚😆🥲😗🥲😆🥲🤪🥲🤪😛😁🥲😗😄🤪🤪😃🤪😃😆😃😗😃😂😃😗😗😃😗😃😌😃🤪😃😃😚😄😂😗😛😗😛🤪🤪😃😆😃😆😃😗😃🤪😃🤪😃😁😃😆😆😃😗😃😊😃😌😃🤪😃🤪😃🤪😃😗😃😗😃😆😃😆😆😃😆😃😆😃😃😃😁
Deploying a full-stack React application with a CI/CD pipeline, rollback functionality, and server deployment using Apache and SSH key-based authentication is a multi-step process. Below, I've outlined a comprehensive guide to accomplish this task.

### Prerequisites

1. **Basic Understanding of Full-Stack Applications**: Familiarity with React for frontend and Node.js/Express for backend.
2. **Server Access**: A server with Apache installed (like Ubuntu).
3. **Domain Name**: Optional but recommended for production.
4. **SSH Key Pair**: Set up for secure access to your server.
5. **Git**: Ensure Git is installed on your local machine and server.
6. **Node.js and NPM**: Installed on your server.

### Step 1: Setting Up the Project Structure

1. **Create Project Structure**:
   ```bash
   mkdir my-fullstack-app
   cd my-fullstack-app
   mkdir frontend backend
   ```

2. **Initialize Frontend and Backend**:
   - **Frontend (React)**:
     ```bash
     cd frontend
     npx create-react-app .
     ```
   - **Backend (Node.js/Express)**:
     ```bash
     cd ../backend
     npm init -y
     npm install express cors body-parser
     ```

3. **Set Up Basic Express Server**:
   Create `index.js` in `backend`:
   ```javascript
   const express = require('express');
   const cors = require('cors');
   const bodyParser = require('body-parser');

   const app = express();
   const PORT = process.env.PORT || 5000;

   app.use(cors());
   app.use(bodyParser.json());

   app.get('/', (req, res) => {
       res.send('Backend is running');
   });

   app.listen(PORT, () => {
       console.log(`Server is running on http://localhost:${PORT}`);
   });
   ```

### Step 2: Set Up Git Repository

1. **Initialize Git**:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   ```

2. **Create Remote Repository** (e.g., GitHub, GitLab) and push your code:
   ```bash
   git remote add origin <repository-url>
   git push -u origin master
   ```

### Step 3: Configure CI/CD Pipeline

We'll use GitHub Actions for this example.

1. **Create a `.github/workflows/deploy.yml`** in your repository:
   ```yaml
   name: Deploy to Server

   on:
     push:
       branches:
         - main  # Change to your main branch if different

   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         - name: Checkout Code
           uses: actions/checkout@v2

         - name: Set up Node.js
           uses: actions/setup-node@v2
           with:
             node-version: '14'  # Use your desired Node version

         - name: Install Dependencies (Frontend)
           run: |
             cd frontend
             npm install

         - name: Build Frontend
           run: |
             cd frontend
             npm run build

         - name: Install Dependencies (Backend)
           run: |
             cd backend
             npm install

         - name: Deploy to Server
           env:
             SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
           run: |
             echo "$SSH_PRIVATE_KEY" > private_key
             chmod 600 private_key
             ssh -o StrictHostKeyChecking=no -i private_key user@your_server_ip "
               cd /path/to/your/deployment/directory &&
               git pull origin main &&
               cd frontend &&
               npm install &&
               npm run build &&
               cd ../backend &&
               npm install &&
               pm2 restart all || pm2 start index.js
             "
   ```

2. **Store SSH Key in GitHub Secrets**:
   - Generate an SSH key on your local machine:
     ```bash
     ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
     ```
   - Add the public key to your server's `~/.ssh/authorized_keys`.
   - Add the private key (`id_rsa`) to your GitHub repository secrets as `SSH_PRIVATE_KEY`.

### Step 4: Deploy on Apache

1. **Configure Apache for Reverse Proxy**:
   - Enable necessary modules:
     ```bash
     sudo a2enmod proxy
     sudo a2enmod proxy_http
     ```

2. **Create Apache Configuration**:
   Create a new configuration file for your app (e.g., `/etc/apache2/sites-available/myapp.conf`):
   ```apache
   <VirtualHost *:80>
       ServerName your_domain_or_IP

       ProxyPreserveHost On
       ProxyPass /api http://localhost:5000
       ProxyPassReverse /api http://localhost:5000

       DocumentRoot /path/to/your/deployment/directory/frontend/build
       <Directory /path/to/your/deployment/directory/frontend/build>
           Options Indexes FollowSymLinks
           AllowOverride All
           Require all granted
       </Directory>

       ErrorLog ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost
   ```

3. **Enable the Site and Restart Apache**:
   ```bash
   sudo a2ensite myapp.conf
   sudo systemctl restart apache2
   ```

### Step 5: Rollback Functionality

1. **Set Up Deployment Directory**:
   - Use a versioned directory structure:
     ```bash
     mkdir -p /var/www/myapp/{current,previous}
     ```

2. **Update Deployment Step in CI/CD Pipeline**:
   Modify the deployment script in the GitHub Actions workflow to create a versioned directory for rollback:
   ```bash
   ssh -o StrictHostKeyChecking=no -i private_key user@your_server_ip "
     cd /var/www/myapp &&
     mv current previous &&
     mkdir current &&
     git clone -b main <repository-url> current &&
     cd current/frontend &&
     npm install &&
     npm run build &&
     cd ../backend &&
     npm install &&
     pm2 restart all || pm2 start index.js
   "
   ```

3. **Rollback Command**:
   If a rollback is needed, you can execute the following command on the server:
   ```bash
   ssh -i private_key user@your_server_ip "
     cd /var/www/myapp &&
     rm -rf current &&
     mv previous current &&
     cd current/frontend &&
     npm install &&
     npm run build &&
     cd ../backend &&
     npm install &&
     pm2 restart all || pm2 start index.js
   "
   ```

### Step 6: Verify Deployment

1. **Access Your Application**: Open a browser and go to your domain or server IP.
2. **Check Logs**: Look for any errors in the Apache log files or PM2 logs for your Node.js application.

### Additional Considerations

- **SSL/TLS**: Consider securing your application with HTTPS using Let's Encrypt.
- **Environment Variables**: Manage sensitive information through environment variables or configuration files.
- **Database Setup**: If your application uses a database, ensure it is properly configured and secured.

Following these steps will help you deploy a full-stack React application with a CI/CD pipeline, rollback functionality, and Apache server setup. Adjust the instructions according to your specific requirements and environment.


😃😍😚😍😘🤩😘🤩😙🤩🤩😅🤩😚🤩😃😍😙😍😁😂😁😅😃😂😗😍🙂🥰😙🥰😚🥰😚🥰😚🥰😚🥰😙🥰🙂😍😚🥰😚😊😃☺️😗☺️😃😃😃😘😃😆😃😆😙😙😆😙🤩😙😗😂😃🥹😗🥹😗🥹😗🥹😗😊😃😊😙😄😙😊😚🥹😗🥹😙😆😙😂😙😂😗😂🙂😂🙂😂🙂😂🙂😂🙂😂🙂😂🙂😆🙂😆🙂🙂🥹🙂‍↕️🥹🙂‍↕️☺️🥰☺️🙂‍↕️☺️😙😂😚😂🙃😂🙃😂😚😂🙂😜🥴🙂😂🙂🥴😙😂😚🥴😚🥴😚🥴😙🥴😙
Deploying a full-stack React application with a CI/CD pipeline, rollback functionality, and using Apache for hosting requires several steps. Below is a comprehensive guide detailing each step, ensuring no part is skipped.

### Prerequisites

1. **Server Setup:**
   - A server (VPS, dedicated server, etc.) with a Linux distribution (Ubuntu is commonly used).
   - SSH access to the server.
   - Apache installed on the server.
   - Node.js and npm installed on the server for building the React app.

2. **Version Control:**
   - Your application code must be stored in a version control system like Git (e.g., GitHub, GitLab).

3. **CI/CD Tool:**
   - A CI/CD tool like Jenkins, GitHub Actions, GitLab CI/CD, or CircleCI.

4. **Rollback Mechanism:**
   - Create a system to manage previous builds, which could be via tags in Git or a separate folder for builds.

### Step 1: Prepare Your Full-Stack Application

1. **Backend (Node.js, Express, etc.):**
   - Ensure your backend is ready to be built and run. Your backend should have a `start` script in `package.json` to run your application.

   ```json
   {
     "scripts": {
       "start": "node server.js",
       "build": "your_build_command_here"
     }
   }
   ```

2. **Frontend (React):**
   - Your React application should also have a build script.

   ```json
   {
     "scripts": {
       "build": "react-scripts build"
     }
   }
   ```

### Step 2: Set Up Apache

1. **Install Apache:**

   ```bash
   sudo apt update
   sudo apt install apache2
   ```

2. **Enable Required Modules:**

   ```bash
   sudo a2enmod proxy
   sudo a2enmod proxy_http
   sudo a2enmod rewrite
   ```

3. **Configure Apache:**
   - Create a virtual host file for your application.

   ```bash
   sudo nano /etc/apache2/sites-available/myapp.conf
   ```

   - Add the following configuration:

   ```apache
   <VirtualHost *:80>
       ServerName myapp.com
       DocumentRoot /var/www/myapp/frontend/build

       <Directory /var/www/myapp/frontend/build>
           Options Indexes FollowSymLinks
           AllowOverride All
           Require all granted
       </Directory>

       ProxyPass /api http://localhost:5000/
       ProxyPassReverse /api http://localhost:5000/
   </VirtualHost
   ```

   - Enable the new site and restart Apache:

   ```bash
   sudo a2ensite myapp
   sudo systemctl restart apache2
   ```

### Step 3: Set Up SSH Key Authentication

1. **Generate SSH Key Pair on Local Machine:**

   ```bash
   ssh-keygen -t rsa -b 2048
   ```

2. **Copy the Public Key to the Server:**

   ```bash
   ssh-copy-id user@server_ip
   ```

### Step 4: Create a CI/CD Pipeline

#### For GitHub Actions:

1. **Create a `.github/workflows/deploy.yml` file:**

   ```yaml
   name: Deploy to Server

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

         - name: Set up Node.js
           uses: actions/setup-node@v2
           with:
             node-version: '14'

         - name: Install dependencies
           run: npm install

         - name: Build Frontend
           run: npm run build --prefix frontend

         - name: Build Backend
           run: npm install --prefix backend

         - name: Deploy to Server
           run: |
             ssh user@server_ip "
               cd /var/www/myapp &&
               git pull origin main &&
               rm -rf frontend/build &&
               cp -r frontend/build /var/www/myapp/frontend &&
               pm2 restart backend
             "
           env:
             SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
   ```

2. **Store the SSH Private Key in GitHub Secrets:**
   - Go to your repository settings > Secrets and variables > Actions > New repository secret, and add `SSH_PRIVATE_KEY`.

#### For GitLab CI/CD:

1. **Create a `.gitlab-ci.yml` file:**

   ```yaml
   stages:
     - build
     - deploy

   build:
     stage: build
     script:
       - npm install --prefix frontend
       - npm run build --prefix frontend
       - npm install --prefix backend

   deploy:
     stage: deploy
     script:
       - ssh user@server_ip "
           cd /var/www/myapp &&
           git pull origin main &&
           rm -rf frontend/build &&
           cp -r frontend/build /var/www/myapp/frontend &&
           pm2 restart backend
         "
     only:
       - main
   ```

### Step 5: Implement Rollback Functionality

1. **Version Your Builds:**
   - When deploying, tag each build with a version number.

   ```bash
   git tag -a v1.0 -m "Release v1.0"
   git push origin v1.0
   ```

2. **Rollback Script:**
   - Create a script that can be executed to rollback to a previous version:

   ```bash
   ssh user@server_ip "
     cd /var/www/myapp &&
     git checkout v1.0 &&
     npm install --prefix backend &&
     rm -rf frontend/build &&
     cp -r frontend/build /var/www/myapp/frontend &&
     pm2 restart backend
   "
   ```

3. **Optionally, integrate the rollback functionality into your CI/CD pipeline using environment variables or conditions based on build success/failure.**

### Step 6: Final Checks

1. **Access Your Application:**
   - Visit `http://myapp.com` to see your application in action.

2. **Monitoring:**
   - Use tools like `pm2` to manage your Node.js application processes and check for any runtime issues.

### Conclusion

By following these steps, you will have successfully deployed your full-stack React application using a CI/CD pipeline, with rollback capabilities and hosted on an Apache server. Be sure to adjust configurations based on your specific application needs and server environment.

🙂‍↕️😘😘😌🤩😌🥹😌🥹🙂‍↕️😘🙂‍↕️🤩🙂‍↕️🥹🙂‍↕️🥰🙂‍↕️🥰🙂‍↕️🥰🙂‍↕️🥰🙂‍↕️😊😄😊😄🤩😄😊😄🥹😄🥹😄🥰😄🥹😄🥲😄🥹😄🥹😄🥹😂🥹😂😊😄😊😄😗🙂‍↕️🤪🙂‍↕️🥰😂😆😂🤪😂🤪😂🤪😂🥰😂🥰🥹🙂‍↕️🙂‍↕️🤩🤩😌🙂😌😜🙂‍↕️🥭🙂‍↕️🥭🙂‍↕️😜🙂‍↕️🤪🙂‍↕️🥰🙂‍↕️🥰😛🥰😛🤪😛🤪🙂🙂‍↕️🤪😛😗😛🤪😛🥰😛🥲😛🥲😛🥲😛☺️🙂‍↕️☺️😌😊😌😊😌🥲😌🥲😌🥲😌😊😌🥹😌☺️🙂‍↕️🥲😌🤪🙂‍↕️🥰🙂‍↕️🥰🙂‍↕️🥰🙂‍↕️🥰🙂‍↕️🥰🙂‍↕️🥰🙂‍↕️🤪

     

    



To deploy a full-stack React application with a backend using GitLab CI/CD pipelines, and ensure continuous deployment, rollback functionality, SSH key authentication, and Apache as the web server for both frontend and backend, follow these detailed steps:

### Prerequisites:
1. **GitLab repository**: Store your full-stack application (React frontend and backend).
2. **SSH Access to the server**: You'll need SSH access to your server where Apache is running.
3. **Apache installed**: Ensure that Apache is installed on your server.
4. **Server configuration**: Set up SSH key authentication and ensure your server is ready for deployment.
5. **Domain or IP**: You need a domain name or server IP for your application.

---

### 1. **GitLab SSH Key Setup**
You'll need to set up SSH key authentication between your GitLab CI pipeline and your remote server.

1. **Generate SSH Key Pair** on the machine where the pipeline will run (or you can do it on your local machine):
   ```bash
   ssh-keygen -t rsa -b 4096 -C "gitlab-ci-deploy"
   ```
2. **Add Public Key to Remote Server**:
   Copy the public key to the remote server where the application will be deployed:
   ```bash
   ssh-copy-id user@your-server-ip
   ```
3. **Add Private Key to GitLab**:
   - Go to GitLab -> Your Project -> Settings -> CI/CD -> Variables.
   - Add a new variable `SSH_PRIVATE_KEY` and paste the private key (`~/.ssh/id_rsa`) here.
   - Make sure to mask this variable.

### 2. **Configure Apache for Deployment**
For the frontend and backend, you will create separate Apache configurations.

- **Frontend**: React app
- **Backend**: Node.js/Express, Flask, or other backend frameworks


1. **Initialize Git Repository**:
   - Create a new GitLab project.
   - Clone the GitLab repository to your local machine.

2. **Setup Frontend (React)**:
   - Create a React app:
     ```bash
     npx create-react-app frontend
     ```
   - Test that the app works locally:
     ```bash
     cd frontend
     npm start
     ```

3. **Setup Backend (Express)**:
   - In the project root, create a folder for the backend:
     ```bash
     mkdir backend
     cd backend
     npm init -y
     npm install express
     ```
   - Create a simple Express server (`backend/server.js`):
     ```javascript
     const express = require('express');
     const app = express();
     const PORT = process.env.PORT || 5000;

     app.get('/', (req, res) => res.send('Hello from the backend!'));

     app.listen(PORT, () => console.log(`Backend running on port ${PORT}`));
     ```
   - Add a `start` script to `backend/package.json`:
     ```json
     "scripts": {
       "start": "node server.js"
     }
     ```
   - Test the server locally:
     ```bash
     npm start
     ```

4. **Push Project to GitLab**:
   - Add both frontend and backend folders, commit, and push to GitLab:
     ```bash
     git add .
     git commit -m "Initial commit with frontend and backend"
     git push origin main
     ```


#### 2.1 Apache Configuration for React Frontend
Assume your React app will be deployed at `/var/www/frontend`.

1. Create a new Apache configuration file, e.g., `/etc/apache2/sites-available/frontend.conf`:
   ```apache
   <VirtualHost *:80>
       ServerName frontend.example.com
       DocumentRoot /var/www/frontend

       <Directory /var/www/frontend>
           Options Indexes FollowSymLinks
           AllowOverride All
           Require all granted
       </Directory>

       ErrorLog ${APACHE_LOG_DIR}/frontend-error.log
       CustomLog ${APACHE_LOG_DIR}/frontend-access.log combined
   </VirtualHost>
   ```

2. Enable the site:
   ```bash
   sudo a2ensite frontend.conf
   sudo systemctl reload apache2
   ```

#### 2.2 Apache Configuration for Backend
Assume your backend will be deployed at `/var/www/backend`.

1. Create a new Apache configuration file, e.g., `/etc/apache2/sites-available/backend.conf`:
   ```apache
   <VirtualHost *:80>
       ServerName backend.example.com
       ProxyPreserveHost On
       ProxyPass / http://localhost:5000/
       ProxyPassReverse / http://localhost:5000/

       ErrorLog ${APACHE_LOG_DIR}/backend-error.log
       CustomLog ${APACHE_LOG_DIR}/backend-access.log combined
   </VirtualHost>
   ```

2. Enable the proxy modules and the backend site:
   ```bash
   sudo a2enmod proxy
   sudo a2enmod proxy_http
   sudo a2ensite backend.conf
   sudo systemctl reload apache2
   ```

### 3. **GitLab CI/CD Pipeline Setup**

In your GitLab project, create a `.gitlab-ci.yml` file at the root of the repository. This will contain the necessary steps for deployment and rollback.

#### 3.1 Define Stages
Define the stages like `build`, `deploy`, and `rollback`.

```yaml
stages:
  - build
  - deploy
  - rollback
```

#### 3.2 Build Frontend and Backend
The build stage will compile the frontend and prepare the backend for deployment.

```yaml
build-frontend:
  stage: build
  script:
    - cd frontend
    - npm install
    - npm run build
  artifacts:
    paths:
      - frontend/build/

build-backend:
  stage: build
  script:
    - cd backend
    - npm install
    - npm run build
  artifacts:
    paths:
      - backend/
```

#### 3.3 Deploy Frontend and Backend
The deploy stage will use SSH to copy the built files to the server.

```yaml
deploy-frontend:
  stage: deploy
  script:
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - ssh -o StrictHostKeyChecking=no user@your-server-ip "rm -rf /var/www/frontend/*"
    - scp -r frontend/build/* user@your-server-ip:/var/www/frontend/

deploy-backend:
  stage: deploy
  script:
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - ssh -o StrictHostKeyChecking=no user@your-server-ip "rm -rf /var/www/backend/*"
    - scp -r backend/* user@your-server-ip:/var/www/backend/
    - ssh user@your-server-ip "systemctl restart backend.service"
```

#### 3.4 Rollback
In case of a failed deployment, use the rollback stage to revert to a previous version.

```yaml
rollback:
  stage: rollback
  script:
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - ssh user@your-server-ip "rm -rf /var/www/frontend/* && cp -r /var/www/frontend_backup/* /var/www/frontend/"
    - ssh user@your-server-ip "rm -rf /var/www/backend/* && cp -r /var/www/backend_backup/* /var/www/backend/"
    - ssh user@your-server-ip "systemctl restart backend.service"
```

You may want to adjust your rollback logic based on how you back up your files (e.g., using versioned folders).

### 4. **Add Deployment Conditions**
You can add conditions to deploy only on specific branches (e.g., `main` or `production`):

```yaml
deploy-frontend:
  stage: deploy
  only:
    - main
```

### 5. **Backup and Rollback Strategies**
Make sure to include a strategy for keeping backups of previous versions of the frontend and backend in case a rollback is needed. For example:

1. **Backup Before Deployment**:
   Add a step in the deployment process to back up the existing version before deploying the new one:
   ```bash
   ssh user@your-server-ip "cp -r /var/www/frontend /var/www/frontend_backup"
   ssh user@your-server-ip "cp -r /var/www/backend /var/www/backend_backup"
   ```

2. **Rollback in GitLab**:
   Trigger a rollback if the deployment fails. You can automatically check for deployment success and proceed to rollback on failure by using GitLab’s `allow_failure` and `when: on_failure` options.

---

### Final Thoughts:
This setup provides an end-to-end solution for deploying your full-stack React app with GitLab CI/CD pipelines, including continuous deployment, rollback functionality, and Apache server configuration. The process ensures that both the frontend and backend are properly deployed and managed on the server using SSH authentication and GitLab CI variables.
  


😗☺️😙☺️😚😁☺️😘😍😆😂😁😚😍😄😂😙😂😙😍😙😍😚😍😚😍😘😍😚🤩😙🤩😙🤩😚🤩😚🤩😚🤩😚😚🤩😚🤩😚🤩😚🤩😘🤩😘🤩😚🤩😚🤩😚🤩😚🤩😚🤩😚🤩😚🤩😁🤩😚🤩😚😚🤩😚🤩😚🤩😚🤩😁🤩😁🤩😁🤩😆🤩😆🤩😁🤩🤩😚🤩😚😁🤩😁🤩😁🤩😆🤩😚🤩😚🤩😁🤩😆🤩🤩😚🤩😁🤩😁🤩😁🤩😚🤩😚🤩😁🤩😁😍😘🤩😙🤩😗🤩😄🤩😁🤩😆🤩😚🤩😚😁🤩😁🤩😘🤩😙🤩😗🤩😉🤣🙃🤩🙂🤩🥲🤩🙂🤩😙🤩
🤪🤪🥰😆😊

To deploy a full-stack React application via GitLab CI/CD with rollback functionality, SSH key authentication, and deployment over Apache for both frontend and backend, while ensuring artifacts are included in the pipeline, follow these steps:

### Step-by-Step Guide:

1. **SSH Key Authentication Setup**
2. **React Full-Stack Application Setup**
3. **Apache Configuration for Backend and Frontend with a Domain**
4. **CI/CD Pipeline Configuration with GitLab**
5. **Adding Rollback Functionality**

---

### 1. **SSH Key Authentication Setup**

Before setting up the pipeline, make sure you can SSH into your server using SSH keys.

- **Generate SSH Key Pair**:
  On your local machine or CI runner:
  ```bash
  ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
  ```

- **Copy the Public Key to the Server**:
  ```bash
  ssh-copy-id username@server_ip_address
  ```

- **Test the Connection**:
  ```bash
  ssh username@server_ip_address
  ```

- **Store SSH Private Key in GitLab**:
  - Go to your GitLab project > Settings > CI/CD > Variables.
  - Add a new variable `SSH_PRIVATE_KEY` with the value of your private SSH key (`id_rsa`).
  - Optionally, add `DEPLOY_USER` and `DEPLOY_SERVER_IP` variables for better script management.

---

### 2. **React Full-Stack Application Setup**

#### **Frontend: React**
- Create your React app:
  ```bash
  npx create-react-app my-app
  cd my-app
  ```

- Build the production version:
  ```bash
  npm run build
  ```

#### **Backend: Node.js/Express**
- Initialize a backend directory:
  ```bash
  mkdir backend && cd backend
  npm init -y
  npm install express
  ```

- Basic Express server setup:
  ```js
  const express = require('express');
  const path = require('path');
  const app = express();

  // Serve static files from the React frontend app
  app.use(express.static(path.join(__dirname, 'frontend/build')));

  // Handle React routing, return all requests to the React app
  app.get('*', (req, res) => {
    res.sendFile(path.join(__dirname, 'frontend/build', 'index.html'));
  });

  const PORT = process.env.PORT || 5000;
  app.listen(PORT, () => {
    console.log(`Server listening on port ${PORT}`);
  });
  ```

- **Copy the React production build to the backend**:
  ```bash
  cp -r /path/to/react-app/build /path/to/backend/build
  
  ```

---

### 3. **Apache Configuration for Backend and Frontend with a Domain**

#### **Install Apache on the Server**:
On your server, install Apache:
```bash
sudo apt-get update
sudo apt-get install apache2
```

#### **Enable Apache Proxy Modules**:
You will need to enable Apache modules for proxying requests from Apache to the Node.js backend.

```bash
sudo a2enmod proxy proxy_http
sudo systemctl restart apache2
```

#### **Virtual Host Configuration for Apache**:
Configure Apache to serve both your frontend (React) and backend (Node.js) through a domain name.

1. **Create a virtual host file**:
   ```bash
   sudo nano /etc/apache2/sites-available/your-domain.com.conf
   ```

2. **Add the following configuration** to route requests:
   ```apache
   <VirtualHost *:80>
       ServerName your-domain.com
       DocumentRoot /var/www/my-app/frontend/build

       ProxyRequests Off
       ProxyPreserveHost On

       # Proxy for Backend (Node.js server running on port 5000)
       ProxyPass /api http://localhost:5000/api
       ProxyPassReverse /api http://localhost:5000/api

       # Serve React Frontend
       <Directory /var/www/my-app/frontend/build>
           Options Indexes FollowSymLinks
           AllowOverride All
           Require all granted
       </Directory>

       # Handle React Router (if using client-side routing)
       ErrorDocument 404 /index.html

       # Log files
       ErrorLog ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```

3. **Enable the new site**:
   ```bash
   sudo a2ensite your-domain.com.conf
   sudo systemctl restart apache2
   ```

4. **Update DNS**:
   Make sure your domain name (e.g., `your-domain.com`) points to your server's IP address using your DNS provider.

5. **(Optional) SSL with Let’s Encrypt**:
   You can secure your site with SSL using [Certbot](https://certbot.eff.org/):
   ```bash
   sudo apt-get install certbot python3-certbot-apache
   sudo certbot --apache -d your-domain.com
   ```

---

### 4. **GitLab CI/CD Pipeline Configuration with Artifacts**

Create a `.gitlab-ci.yml` file in the root of your project to define your GitLab CI/CD pipeline.

#### **GitLab CI/CD Pipeline:**

```yaml
stages:
  - build
  - deploy

variables:
  BACKEND_DIR: backend
  FRONTEND_DIR: my-app

# Build the frontend and backend
build:
  stage: build
  image: node:16
  script:
    # Build the frontend React app
    - cd $FRONTEND_DIR
    - npm install
    - npm run build
    - cd ..

    # Copy the built React app into the backend
    - cp -r $FRONTEND_DIR/build $BACKEND_DIR/frontend/build

    # Package backend (if needed)
    - cd $BACKEND_DIR
    - npm install

  artifacts:
    paths:
      - $BACKEND_DIR/frontend/build
    expire_in: 1 week

# Deploy to server using SSH
deploy:
  stage: deploy
  image: node:16
  before_script:
    # Install SSH Agent and configure SSH key
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan -H $DEPLOY_SERVER_IP >> ~/.ssh/known_hosts

  script:
    # Create timestamped deployment directory
    - TIMESTAMP=$(date +%Y%m%d%H%M%S)
    - ssh $DEPLOY_USER@$DEPLOY_SERVER_IP "mkdir -p /var/www/my-app-$TIMESTAMP"

    # Copy backend files (including frontend build) to the server
    - scp -r $BACKEND_DIR/* $DEPLOY_USER@$DEPLOY_SERVER_IP:/var/www/my-app-$TIMESTAMP

    # Symlink new deployment to "current" directory
    - ssh $DEPLOY_USER@$DEPLOY_SERVER_IP "ln -sfn /var/www/my-app-$TIMESTAMP /var/www/my-app"

    # Install dependencies on the server and restart the backend
    - ssh $DEPLOY_USER@$DEPLOY_SERVER_IP "cd /var/www/my-app && npm install --production && pm2 restart all"
    
  only:
    - main
```

### Key Elements of the Pipeline:

- **Build Job**:
  - Builds the React frontend.
  - Copies the built React files (`build/`) into the backend folder.
  - Bundles the backend, ready for deployment.

- **Artifacts**:
  - The frontend `build` directory is saved as an artifact, making it available for the deploy job.

- **Deploy Job**:
  - Uses SSH key authentication (configured using `SSH_PRIVATE_KEY` in GitLab CI/CD settings).
  - Deploys the app to a timestamped directory on the server.
  - Creates a symbolic link to point to the latest version (`/var/www/my-app`).
  - Uses `PM2` to restart the Node.js backend server.

### 5. **Adding Rollback Functionality**

For rollback functionality, you will deploy the app to timestamped directories and use symbolic links to point to the current version. If something goes wrong, you can point the symlink back to the previous version.

#### **Steps for Rollback**:

1. **Deploy to Timestamped Directories**:
   Each deployment creates a directory named with the current timestamp, such as `/var/www/my-app-20241022`.

2. **Symbolic Link (`ln -sfn`)**:
   The symbolic link `/var/www/my-app` always points to the latest deployment. During each deployment, the symlink is updated to point to the new directory.

3. **Manual Rollback**:
   If the new deployment fails, you can point the symbolic link back to the previous deployment by SSHing into the server and running:
   ```bash
   ln -sfn /var/www/my-app-previous-timestamp /var/www/my-app
   pm2 restart all
   ```

4. **Automated Rollback Job (Optional)**:
   You can create a separate rollback job in GitLab for manual rollback:
   ```yaml
   rollback:
     stage: deploy
     script:
       - ssh $DEPLOY_USER@$DEPLOY_SERVER_IP "ln -sfn /var/www/my-app-$ROLLBACK_TIMESTAMP /var/www/my-app"
       - ssh $DEPLOY_USER@$DEPLOY_SERVER_IP "pm2 restart all"
     when:


   😊🥲😊🥲🥰🙂🤪🥲🤩🤩🥲😊🥲🥰🥲🤪🥲🤩🥲🤪🥲🤪🥲🤩🤩🥲🤩🥲🤩🥲🤩🥲🤩🥲🤪🙂🥰🙂🤪🙂🤩🙂🤩🙂🤪🙂🥰🙂🥰🙂🤪🙂🥰🙂🥰🙂🤪🙂🤩🙂🥰🙂🤪🙂🤪🙂

   🔐😔🔐😔😉😔😉😅🥰😄🥰🥰😄🥰😄🥰😄🤩😄🤩😆🤩😆🤩😆🥰😙🤪😙🙂😙🤪😙🥰😙🤩🤩😙🤩😙😆🥰😆🤪🤪😆🤩😆😆😉😊😉😚☺️😊😉😊😉😊🥰😊🥰😊🤩🤩😆🥰


Creating and deploying a full-stack React application with a backend server via a CI/CD pipeline using GitLab CI, including rollback functionality, SSH key authentication, and deployment over Apache, involves several steps. Below is a detailed guide outlining the entire process.

### Prerequisites
1. **GitLab Account**: Ensure you have a GitLab account and a repository for your project.
2. **Server Setup**: You need access to a server (VPS, AWS, etc.) where you will deploy your application. Ensure Apache is installed and configured.
3. **SSH Key Pair**: Generate an SSH key pair for secure access to your server.
4. **Node.js and npm**: Make sure Node.js and npm are installed on your local machine and the server.
5. **React Application**: Have a basic React application set up with a backend (Node.js/Express, for example).

### Step 1: Project Structure
Assume the following structure for your full-stack application:

```
my-fullstack-app/
├── client/          # React front-end
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── ...
├── server/          # Node.js back-end
│   ├── src/
│   ├── package.json
│   └── ...
├── .gitlab-ci.yml   # GitLab CI/CD configuration file
└── README.md
```

### Step 2: Setup SSH Key Authentication
1. **Generate SSH Keys**: On your local machine, run:
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```
   Follow the prompts to save the key. The default location is usually fine.

2. **Copy Public Key to Server**: Copy the public key to your server using:
   ```bash
   ssh-copy-id user@your-server-ip
   ```

3. **Test SSH Access**: Ensure you can log in to the server without a password:
   ```bash
   ssh user@your-server-ip
   ```

### Step 3: Configure Apache
1. **Install Apache**: If not already installed, run:
   ```bash
   sudo apt update
   sudo apt install apache2
   ```

2. **Enable Proxy Modules**:
   ```bash
   sudo a2enmod proxy
   sudo a2enmod proxy_http
   ```

3. **Configure Virtual Host**: Create a new configuration file for your application (e.g., `/etc/apache2/sites-available/myapp.conf`):
   ```apache
   <VirtualHost *:80>
       ServerName your-domain.com

       ProxyRequests off
       ProxyPass /api http://localhost:5000/api
       ProxyPassReverse /api http://localhost:5000/api

       DocumentRoot /var/www/my-fullstack-app/client/build

       <Directory /var/www/my-fullstack-app/client/build>
           Options Indexes FollowSymLinks
           AllowOverride All
           Require all granted
       </Directory>

       ErrorLog ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost
   ```
4. **Enable the site and restart Apache**:
   ```bash
   sudo a2ensite myapp.conf
   sudo systemctl restart apache2
   ```

### Step 4: Create the GitLab CI/CD Pipeline
Create a `.gitlab-ci.yml` file in the root of your repository. Here’s a basic example:

```yaml
image: node:14

stages:
  - build
  - deploy

variables:
  SSH_PRIVATE_KEY: $SSH_PRIVATE_KEY # Store your private key in GitLab CI/CD variables

before_script:
  - echo "$SSH_PRIVATE_KEY" > private_key
  - chmod 600 private_key
  - apt-get update && apt-get install -y sshpass

build_client:
  stage: build
  script:
    - cd client
    - npm install
    - npm run build
  artifacts:
    paths:
      - client/build

build_server:
  stage: build
  script:
    - cd server
    - npm install

deploy:
  stage: deploy
  script:
    - ssh -o StrictHostKeyChecking=no -i private_key user@your-server-ip "rm -rf /var/www/my-fullstack-app/*"
    - scp -o StrictHostKeyChecking=no -i private_key -r client/build/* user@your-server-ip:/var/www/my-fullstack-app/client/build/
    - scp -o StrictHostKeyChecking=no -i private_key -r server/* user@your-server-ip:/var/www/my-fullstack-app/server/
    - ssh -o StrictHostKeyChecking=no -i private_key user@your-server-ip "cd /var/www/my-fullstack-app/server && npm install && pm2 restart all" # Using pm2 to manage the server

rollback:
  stage: deploy
  script:
    - echo "Rolling back to previous version"
    - ssh -o StrictHostKeyChecking=no -i private_key user@your-server-ip "cd /var/www/my-fullstack-app && git checkout previous_commit_hash"
```

### Step 5: Environment Variables
1. **Set SSH Private Key**: Go to your GitLab project settings, then to CI/CD, and add a new variable `SSH_PRIVATE_KEY`. Paste the contents of your private key here.

2. **Add Other Environment Variables**: If your application requires any environment variables, add them similarly in the CI/CD settings.

### Step 6: Test Your Pipeline
1. Commit and push your code to GitLab.
2. Navigate to the CI/CD section in your GitLab project to monitor the pipeline.
3. Check for any errors and ensure all stages complete successfully.

### Step 7: Validate Deployment
1. Open your browser and navigate to your server's domain or IP.
2. Ensure the React app loads and the backend API works.

### Step 8: Rollback Functionality
To rollback, you can either run the rollback job from the GitLab interface or manually revert to a previous commit on your server.

### Step 9: Set Up Continuous Deployment
You can also set up webhooks in GitLab to trigger the pipeline on every push to the main branch for continuous deployment.

### Conclusion
This guide provides a comprehensive approach to creating and deploying a full-stack React application using GitLab CI/CD, complete with SSH authentication, rollback functionality, and Apache server setup. Adjust configurations as needed based on your specific requirements and environments.



😀😄😁☺️😆😆🤣🤣😄😆🤣😙🤩😙🤩😘🤩😗🤩😁😌😀😀😌😀😌😀😌😚😚😌😀😌🙂🤩🥹🤩😚😌😚😌😘😌😙😌🥹😌🥹😚😌🥰😌😚🙂‍↕️🥰🙂‍↕️🥰🥴😚🙂‍↕️😊☺️🥴😘😄🤩🥹🤩🤩🥲🤩😙🤩🥹😌😙😚😚😚☺️😙☺️😚☺️😙☺️😚☺️😚😙☺️😙☺️😚☺️😙☺️😙😙☺️😙☺️😙☺️😙☺️😙☺️😙😙☺️😌🥹😌🥲
Creating and deploying a full-stack MERN (MongoDB, Express, React, Node.js) authentication application with password reset functionality, along with setting up a CI/CD pipeline using GitLab CI, involves multiple steps. Below, I’ll break down the entire process into detailed sections, covering backend and frontend development, as well as the CI/CD pipeline setup with rollback functionality.

### Overview

1. **Set up the Project Structure**
2. **Backend Development**
   - Create a Node.js and Express backend
   - Implement MongoDB for user storage
   - Set up authentication with JWT
   - Create password reset functionality
3. **Frontend Development**
   - Create a React frontend
   - Implement registration, login, and reset password pages
4. **Set up CI/CD with GitLab**
   - Configure GitLab CI/CD pipeline
   - Set up SSH key authentication
   - Implement rollback functionality
5. **Deploy the Application**
   - Deploy the backend on Apache
   - Serve the frontend using Apache

### Step 1: Set up the Project Structure

Create a directory for your project and set up two folders: one for the backend and one for the frontend.

```bash
mkdir mern-auth-app
cd mern-auth-app
mkdir backend frontend
```

### Step 2: Backend Development

#### 2.1 Create a Node.js and Express Backend

1. **Navigate to the backend folder**:

   ```bash
   cd backend
   ```

2. **Initialize a Node.js project**:

   ```bash
   npm init -y
   ```

3. **Install necessary packages**:

   ```bash
   npm install express mongoose bcryptjs jsonwebtoken dotenv nodemailer cors
   ```

4. **Create the following file structure**:

   ```plaintext
   backend/
   ├── config/
   │   └── db.js
   ├── controllers/
   │   └── authController.js
   ├── models/
   │   └── User.js
   ├── routes/
   │   └── authRoutes.js
   ├── middleware/
   │   └── authMiddleware.js
   ├── .env
   ├── server.js
   └── package.json
   ```

#### 2.2 Implement MongoDB for User Storage

- **Connect to MongoDB** (`config/db.js`):

```javascript
const mongoose = require('mongoose');

const connectDB = async () => {
    try {
        await mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true });
        console.log('MongoDB Connected');
    } catch (error) {
        console.error(error);
        process.exit(1);
    }
};

module.exports = connectDB;
```

- **User Model** (`models/User.js`):

```javascript
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
    username: { type: String, required: true, unique: true },
    email: { type: String, required: true, unique: true },
    password: { type: String, required: true },
});

module.exports = mongoose.model('User', userSchema);
```

#### 2.3 Set up Authentication with JWT

- **Authentication Controller** (`controllers/authController.js`):

```javascript
const User = require('../models/User');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

// Register user
exports.register = async (req, res) => {
    const { username, email, password } = req.body;

    try {
        const hashedPassword = await bcrypt.hash(password, 10);
        const user = await User.create({ username, email, password: hashedPassword });
        res.status(201).json({ message: 'User registered', user });
    } catch (error) {
        res.status(400).json({ message: error.message });
    }
};

// Login user
exports.login = async (req, res) => {
    const { email, password } = req.body;

    try {
        const user = await User.findOne({ email });
        if (!user || !(await bcrypt.compare(password, user.password))) {
            return res.status(400).json({ message: 'Invalid credentials' });
        }

        const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET, { expiresIn: '1h' });
        res.json({ token });
    } catch (error) {
        res.status(400).json({ message: error.message });
    }
};

// Reset Password
exports.resetPassword = async (req, res) => {
    const { email, newPassword } = req.body;
    const hashedPassword = await bcrypt.hash(newPassword, 10);

    try {
        await User.updateOne({ email }, { password: hashedPassword });
        res.json({ message: 'Password reset successful' });
    } catch (error) {
        res.status(400).json({ message: error.message });
    }
};
```

#### 2.4 Create Routes

- **Routes** (`routes/authRoutes.js`):

```javascript
const express = require('express');
const { register, login, resetPassword } = require('../controllers/authController');

const router = express.Router();

router.post('/register', register);
router.post('/login', login);
router.post('/reset-password', resetPassword);

module.exports = router;
```

#### 2.5 Create the Server

- **Server setup** (`server.js`):

```javascript
const express = require('express');
const dotenv = require('dotenv');
const cors = require('cors');
const connectDB = require('./config/db');
const authRoutes = require('./routes/authRoutes');

dotenv.config();
connectDB();

const app = express();
app.use(cors());
app.use(express.json());
app.use('/api/auth', authRoutes);

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

#### 2.6 Environment Variables

- Create a `.env` file in the backend folder:

```plaintext
MONGO_URI=mongodb://your_mongo_uri
JWT_SECRET=your_jwt_secret
```

### Step 3: Frontend Development

#### 3.1 Create a React Frontend

1. **Navigate to the frontend folder**:

   ```bash
   cd ../frontend
   ```

2. **Create a React app**:

   ```bash
   npx create-react-app .
   ```

3. **Install Axios for API calls**:

   ```bash
   npm install axios react-router-dom
   ```

#### 3.2 Create Pages

- **Create the following file structure**:

```plaintext
frontend/
├── src/
│   ├── components/
│   │   ├── Login.js
│   │   ├── Register.js
│   │   ├── ResetPassword.js
│   ├── App.js
│   └── index.js
```

#### 3.3 Implement Authentication Pages

- **Login Component** (`components/Login.js`):

```javascript
import React, { useState } from 'react';
import axios from 'axios';

const Login = () => {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');

    const handleLogin = async (e) => {
        e.preventDefault();
        try {
            const response = await axios.post('http://localhost:5000/api/auth/login', { email, password });
            localStorage.setItem('token', response.data.token);
            alert('Login successful!');
        } catch (error) {
            alert('Login failed: ' + error.response.data.message);
        }
    };

    return (
        <form onSubmit={handleLogin}>
            <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" required />
            <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Password" required />
            <button type="submit">Login</button>
        </form>
    );
};

export default Login;
```

- **Register Component** (`components/Register.js`):

```javascript
import React, { useState } from 'react';
import axios from 'axios';

const Register = () => {
    const [username, setUsername] = useState('');
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');

    const handleRegister = async (e) => {
        e.preventDefault();
        try {
            await axios.post('http://localhost:5000/api/auth/register', { username, email, password });
            alert('Registration successful!');
        } catch (error) {
            alert('Registration failed: ' + error.response.data.message);
        }
    };

    return (
        <form onSubmit={handleRegister}>
            <input type="text" value={username} onChange={(e) => setUsername(e.target.value)} placeholder="Username" required />
            <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" required />
            <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Password" required />
            <button type="submit">Register</button>
        </form>
    );
};

export default Register;
```

- **Reset Password Component** (`components/ResetPassword.js`):

```javascript
import React, { useState } from 'react';
import axios from 'axios';

const ResetPassword = () => {
    const [email, setEmail] = useState('');
    const [newPassword, setNewPassword] = useState('');

    const handleReset =

 async (e) => {
        e.preventDefault();
        try {
            await axios.post('http://localhost:5000/api/auth/reset-password', { email, newPassword });
            alert('Password reset successful!');
        } catch (error) {
            alert('Password reset failed: ' + error.response.data.message);
        }
    };

    return (
        <form onSubmit={handleReset}>
            <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" required />
            <input type="password" value={newPassword} onChange={(e) => setNewPassword(e.target.value)} placeholder="New Password" required />
            <button type="submit">Reset Password</button>
        </form>
    );
};

export default ResetPassword;
```

#### 3.4 Configure Routing

- **Update `App.js`**:

```javascript
import React from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import Login from './components/Login';
import Register from './components/Register';
import ResetPassword from './components/ResetPassword';

const App = () => {
    return (
        <Router>
            <Switch>
                <Route path="/login" component={Login} />
                <Route path="/register" component={Register} />
                <Route path="/reset-password" component={ResetPassword} />
            </Switch>
        </Router>
    );
};

export default App;
```

### Step 4: Set up CI/CD with GitLab

#### 4.1 GitLab CI/CD Pipeline Configuration

1. **Create a `.gitlab-ci.yml` file in the root of your project**:

```yaml
stages:
  - build
  - test
  - deploy

build_backend:
  stage: build
  script:
    - cd backend
    - npm install
    - npm run build
  artifacts:
    paths:
      - backend/build/

build_frontend:
  stage: build
  script:
    - cd frontend
    - npm install
    - npm run build
  artifacts:
    paths:
      - frontend/build/

deploy_backend:
  stage: deploy
  script:
    - ssh user@your-server "cd /path/to/your/backend && git pull && npm install && pm2 restart server"
  only:
    - master

deploy_frontend:
  stage: deploy
  script:
    - ssh user@your-server "cd /path/to/your/frontend && git pull && npm install && pm2 restart frontend"
  only:
    - master
```

#### 4.2 SSH Key Authentication

1. **Generate an SSH key** on your local machine (if you don't have one):

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

2. **Copy the public key**:

```bash
cat ~/.ssh/id_rsa.pub
```

3. **Add the public key to your server's `~/.ssh/authorized_keys`**.

4. **Add the private key to GitLab CI/CD**:
   - Go to your project in GitLab.
   - Navigate to **Settings > CI/CD > Variables**.
   - Add a new variable called `SSH_PRIVATE_KEY` and paste your private key.

5. **Modify the `.gitlab-ci.yml` to use the SSH key**:

```yaml
before_script:
  - eval $(ssh-agent -s)
  - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
  - mkdir -p ~/.ssh
  - chmod 700 ~/.ssh
  - ssh-keyscan -H your-server >> ~/.ssh/known_hosts
  - chmod 644 ~/.ssh/known_hosts
```

#### 4.3 Implement Rollback Functionality

- You can create a rollback stage in the `.gitlab-ci.yml` that will allow you to revert to a previous deployment if the current one fails. Here’s how you might structure that:

```yaml
rollback:
  stage: deploy
  script:
    - ssh user@your-server "cd /path/to/your/backend && git checkout previous_commit_hash"
  when: manual
```

### Step 5: Deploy the Application

#### 5.1 Deploy the Backend on Apache

1. **Install Apache and Node.js on your server** if not already done.

2. **Set up a reverse proxy for your Node.js app**. Create an Apache configuration file (e.g., `backend.conf`):

```apache
<VirtualHost *:80>
    ServerName your_domain_or_IP

    ProxyRequests Off
    ProxyPass /api http://localhost:5000/api
    ProxyPassReverse /api http://localhost:5000/api

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

3. **Enable the proxy modules**:

```bash
sudo a2enmod proxy
sudo a2enmod proxy_http
```

4. **Restart Apache**:

```bash
sudo systemctl restart apache2
```

#### 5.2 Serve the Frontend Using Apache

1. **Build the React app**:

```bash
cd frontend
npm run build
```

2. **Copy the build files to the Apache server**:

```bash
scp -r build/* user@your-server:/var/www/html
```

3. **Create an Apache configuration file for the frontend**:

```apache
<VirtualHost *:80>
    ServerName your_domain_or_IP

    DocumentRoot /var/www/html

    <Directory /var/www/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

4. **Restart Apache**:

```bash
sudo systemctl restart apache2
```

### Conclusion

You now have a full-stack MERN authentication application with registration, login, and password reset functionality. The application is deployed using a CI/CD pipeline with GitLab CI, and you’ve implemented rollback functionality for deployments.

Make sure to secure your server, manage secrets properly, and consider further enhancements such as email verification and user roles based on your project requirements. If you have any specific areas you'd like to explore further, let me know!

😌🥹😌🥹😌🥲😌😊😌😊😌😊🥹😌😊😌🥹😌🥲😌🥲🙂‍↕️😊🤪😊😛🤪🥲🤪🥹🤪😊🤪🥲🤪🥹😝🤪🥲🤪🥹🤪🥹🤪🥹🤪🥹😌🥹😌😊🥰🤪🥰🤪😘🤪😊🤪😊🤪☺️😊🤪😊🤪😊🤪
Creating and deploying a full-stack MERN (MongoDB, Express, React, Node.js) application with authentication and a password reset functionality is a comprehensive task. To accomplish this, we'll detail the entire process step-by-step, covering everything from development to deployment using a CI/CD pipeline with GitLab. This guide will include rollback functionality, SSH key authentication, and deployment using Apache.

### **1. Project Setup**

#### **1.1. Backend Setup (Node.js + Express)**

1. **Initialize the Backend Project**
   ```bash
   mkdir mern-auth-backend
   cd mern-auth-backend
   npm init -y
   npm install express mongoose bcryptjs jsonwebtoken dotenv cors nodemailer
   ```

2. **Create Directory Structure**
   ```bash
   mkdir config controllers models routes middleware
   ```

3. **Setup Environment Variables**
   - Create a `.env` file for sensitive configurations.
   ```plaintext
   PORT=5000
   MONGO_URI=mongodb://<user>:<password>@localhost:27017/mydatabase
   JWT_SECRET=mysecretkey
   EMAIL_USER=myemail@example.com
   EMAIL_PASS=mypassword
   ```

4. **Create Basic Server Setup (`index.js`)**
   ```javascript
   const express = require('express');
   const mongoose = require('mongoose');
   const cors = require('cors');
   const authRoutes = require('./routes/auth');

   require('dotenv').config();

   const app = express();
   app.use(cors());
   app.use(express.json());

   mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true })
       .then(() => console.log("MongoDB connected"))
       .catch(err => console.error(err));

   app.use('/api/auth', authRoutes);

   const PORT = process.env.PORT || 5000;
   app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
   ```

5. **Create User Model (`models/User.js`)**
   ```javascript
   const mongoose = require('mongoose');
   const userSchema = new mongoose.Schema({
       username: { type: String, required: true, unique: true },
       password: { type: String, required: true },
       email: { type: String, required: true, unique: true },
   });
   module.exports = mongoose.model('User', userSchema);
   ```

6. **Setup Authentication Routes (`routes/auth.js`)**
   ```javascript
   const express = require('express');
   const bcrypt = require('bcryptjs');
   const jwt = require('jsonwebtoken');
   const User = require('../models/User');

   const router = express.Router();

   // Register route
   router.post('/register', async (req, res) => {
       const { username, password, email } = req.body;
       const hashedPassword = await bcrypt.hash(password, 10);
       const newUser = new User({ username, password: hashedPassword, email });
       await newUser.save();
       res.status(201).json({ message: 'User registered' });
   });

   // Login route
   router.post('/login', async (req, res) => {
       const { username, password } = req.body;
       const user = await User.findOne({ username });
       if (!user) return res.status(404).send('User not found');
       const isMatch = await bcrypt.compare(password, user.password);
       if (!isMatch) return res.status(400).send('Invalid credentials');
       const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET);
       res.json({ token });
   });

   // Password reset route
   router.post('/reset-password', async (req, res) => {
       const { email } = req.body;
       const user = await User.findOne({ email });
       if (!user) return res.status(404).send('User not found');

       // Send email logic using nodemailer...
       // Generate reset link and send via nodemailer

       res.send('Password reset link sent');
   });

   module.exports = router;
   ```

#### **1.2. Frontend Setup (React)**

1. **Create the React Application**
   ```bash
   npx create-react-app mern-auth-frontend
   cd mern-auth-frontend
   npm install axios react-router-dom
   ```

2. **Setup Directory Structure**
   ```bash
   mkdir src/components src/pages src/services
   ```

3. **Setup Axios for API Requests**
   - Create `src/services/api.js`:
   ```javascript
   import axios from 'axios';

   const api = axios.create({
       baseURL: 'http://localhost:5000/api/auth',
   });

   export default api;
   ```

4. **Create Authentication Pages and Components**
   - For example, `LoginPage`, `RegisterPage`, `ResetPasswordPage` in `src/pages`.

5. **Routing Setup (`src/App.js`)**
   ```javascript
   import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
   import LoginPage from './pages/LoginPage';
   import RegisterPage from './pages/RegisterPage';
   import ResetPasswordPage from './pages/ResetPasswordPage';

   function App() {
       return (
           <Router>
               <Switch>
                   <Route path="/login" component={LoginPage} />
                   <Route path="/register" component={RegisterPage} />
                   <Route path="/reset-password" component={ResetPasswordPage} />
               </Switch>
           </Router>
       );
   }

   export default App;
   ```

### **2. CI/CD Pipeline Setup with GitLab CI**

#### **2.1. Project Structure in GitLab**

1. **Create a GitLab Repository**
   - Push both frontend and backend projects into separate folders in the same repository.

2. **Create `.gitlab-ci.yml` File**
   - This file will define your CI/CD pipeline.
   ```yaml
   stages:
     - build
     - deploy

   build_backend:
     stage: build
     image: node:14
     script:
       - cd mern-auth-backend
       - npm install
       - npm run build
     artifacts:
       paths:
         - mern-auth-backend/build

   build_frontend:
     stage: build
     image: node:14
     script:
       - cd mern-auth-frontend
       - npm install
       - npm run build
     artifacts:
       paths:
         - mern-auth-frontend/build

   deploy_backend:
     stage: deploy
     image: alpine:latest
     script:
       - apk add --no-cache openssh-client
       - ssh -i $SSH_PRIVATE_KEY -o StrictHostKeyChecking=no user@your_server "cd /path/to/your/backend && git pull origin main && npm install && pm2 restart your-app-name"
     only:
       - main

   deploy_frontend:
     stage: deploy
     image: alpine:latest
     script:
       - apk add --no-cache openssh-client
       - ssh -i $SSH_PRIVATE_KEY -o StrictHostKeyChecking=no user@your_server "cd /path/to/your/frontend && git pull origin main && npm install && npm run build"
       - ssh -i $SSH_PRIVATE_KEY -o StrictHostKeyChecking=no user@your_server "cp -r /path/to/your/frontend/build/* /var/www/html/"
     only:
       - main
   ```

#### **2.2. Configure SSH Key Authentication**

1. **Generate SSH Key Pair**
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

2. **Add Public Key to Server**
   ```bash
   ssh-copy-id user@your_server
   ```

3. **Add Private Key to GitLab**
   - Go to your project settings in GitLab > CI/CD > Variables, and add a variable named `SSH_PRIVATE_KEY` with your private key.

### **3. Rollback Functionality**

For rollback functionality, you can keep track of the last deployment. If a deployment fails, you can revert to the last stable version.

1. **Update `.gitlab-ci.yml` to Create a Backup**
   ```yaml
   backup_backend:
     stage: deploy
     script:
       - ssh -i $SSH_PRIVATE_KEY -o StrictHostKeyChecking=no user@your_server "cp -r /path/to/your/backend /path/to/your/backup/backend_backup_$(date +%Y%m%d%H%M%S)"
     only:
       - main

   rollback_backend:
     stage: deploy
     when: manual
     script:
       - ssh -i $SSH_PRIVATE_KEY -o StrictHostKeyChecking=no user@your_server "cp -r /path/to/your/backup/backend_backup_LAST /path/to/your/backend"
     only:
       - main
   ```

### **4. Apache Configuration for Deployment**

1. **Install Apache**
   ```bash
   sudo apt update
   sudo apt install apache2
   ```

2. **Configure Apache for React App**
   - Create a new configuration file for your frontend.
   ```bash
   sudo nano /etc/apache2/sites-available/mern-auth-frontend.conf
   ```

   ```apache
   <VirtualHost *:80>
       ServerName your_domain.com
       DocumentRoot /var/www/html

       <Directory /var/www/html>
           AllowOverride All
       </Directory>

       ErrorLog ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```

3. **Enable Rewrite Module and Your Site**
   ```bash
   sudo a2

enmod rewrite
   sudo a2ensite mern-auth-frontend.conf
   sudo systemctl restart apache2
   ```

4. **Deploy Backend API**
   - Ensure the Node.js application runs using a process manager like PM2.
   ```bash
   npm install pm2 -g
   pm2 start mern-auth-backend/index.js --name your-app-name
   pm2 save
   ```

### **5. Testing the Application**

1. **Run the Application Locally**
   - Start the backend:
   ```bash
   cd mern-auth-backend
   node index.js
   ```

   - Start the frontend:
   ```bash
   cd mern-auth-frontend
   npm start
   ```

2. **Accessing the Application**
   - Open your browser and go to `http://localhost:3000` for the frontend and `http://localhost:5000/api/auth` for the backend API.

### **6. Conclusion**

You now have a full-stack MERN application with authentication, password reset functionality, and a CI/CD pipeline configured with GitLab. The application can be deployed using SSH with rollback capabilities through the CI/CD pipeline, ensuring you can quickly revert changes if needed.

Remember to test thoroughly and make adjustments based on your specific environment and needs!

   


