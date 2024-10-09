
Deploying a React full-stack web application using GitHub Actions involves several steps. Here’s a step-by-step guide to help you set up the deployment:

### Prerequisites

1. **GitHub Repository**: Ensure your full-stack application is pushed to a GitHub repository.
2. **Server Access**: Have access to a server (e.g., DigitalOcean, AWS, etc.) where you can deploy your application.
3. **Environment Setup**: Ensure your server has Node.js, npm, and any required database installed.
4. **Domain Name (optional)**: If you want a custom domain, make sure it’s set up.

### Step 1: Prepare Your Application

1. **Build Your React App**: Ensure your React application is set up to build properly. Typically, you would run:
   ```bash
   npm run build
   ```
   This command creates a `build` directory containing your production files.

2. **Server Configuration**: Ensure your backend API is set up to serve the built React app and handle requests appropriately.

### Step 2: Set Up GitHub Secrets

1. Go to your GitHub repository.
2. Navigate to **Settings** > **Secrets and variables** > **Actions**.
3. Add the following secrets:
   - `SERVER_IP`: Your server's IP address.
   - `SSH_USER`: The SSH username for your server.
   - `SSH_KEY`: Your private SSH key (ensure your server accepts this key for authentication).
   - `REMOTE_PATH`: The path on your server where the application will be deployed.

### Step 3: Create GitHub Actions Workflow

1. In your repository, create a directory called `.github/workflows`.
2. Inside this directory, create a file named `deploy.yml` (or any name you prefer).

### Sample `deploy.yml`

Here’s a sample workflow configuration:

```yaml
name: Deploy React Full Stack App

on:
  push:
    branches:
      - main  # Change to your deployment branch

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '16'  # Specify your Node.js version

    - name: Install dependencies
      run: |
        cd client  # Change to your React app directory if nested
        npm install

    - name: Build React app
      run: |
        npm run build

    - name: Copy files to server
      uses: appleboy/scp-action@master
      with:
        host: ${{ secrets.SERVER_IP }}
        username: ${{ secrets.SSH_USER }}
        key: ${{ secrets.SSH_KEY }}
        source: "client/build/*"  # Adjust path as necessary
        target: ${{ secrets.REMOTE_PATH }}

    - name: SSH commands to restart server
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.SERVER_IP }}
        username: ${{ secrets.SSH_USER }}
        key: ${{ secrets.SSH_KEY }}
        script: |
          cd ${{ secrets.REMOTE_PATH }}
          npm install # If you need to install server dependencies
          pm2 restart all # Adjust this according to your process manager
```

### Step 4: Configure Your Server

1. **Serve the React App**: Make sure your server is configured to serve the static files from the build directory.
   - Use a server like Nginx or Express.js to serve your React app.
   
2. **Start Your Backend**: Ensure your backend is running, possibly using a process manager like PM2.

### Step 5: Push Changes

1. Commit your changes and push them to the main branch:
   ```bash
   git add .
   git commit -m "Set up deployment with GitHub Actions"
   git push origin main
   ```

### Step 6: Monitor Deployment

1. Check the **Actions** tab in your GitHub repository to monitor the deployment process.
2. Verify your application is accessible on the server.

### Additional Considerations

- **Environment Variables**: Handle any environment variables necessary for your application.
- **Database Migrations**: If your backend requires database migrations, ensure you include those in your deployment script.

By following these steps, you should be able to deploy your React full-stack application to a server using GitHub Actions effectively.
