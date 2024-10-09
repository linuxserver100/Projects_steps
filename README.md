
Here’s a detailed step-by-step guide to setting up a Continuous Deployment (CD) pipeline using GitLab CI/CD on Ubuntu.

### Prerequisites

1. **Ubuntu Server**: A running instance of Ubuntu.
2. **GitLab Account**: An active GitLab account and repository.
3. **Git**: Installed on your server.
4. **SSH Access**: SSH access to your deployment server.
5. **Docker (optional)**: Installed if you plan to use it for deployment.

### Step 1: Install Required Software

1. **Update the system**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Install Git**:
   ```bash
   sudo apt install git -y
   ```

3. **(Optional) Install Docker**:
   ```bash
   sudo apt install docker.io -y
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

### Step 2: Create a GitLab Repository

1. **Create a new repository** on GitLab.
2. **Clone the repository**:
   ```bash
   git clone https://gitlab.com/username/repo-name.git
   cd repo-name
   ```

### Step 3: Create `.gitlab-ci.yml` File

Create a `.gitlab-ci.yml` file in the root of your repository to define the pipeline:

```yaml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - echo "Building the application"
    - # Add your build commands here

test:
  stage: test
  script:
    - echo "Running tests"
    - # Add your test commands here

deploy:
  stage: deploy
  environment:
    name: production
    url: http://your-production-url
  script:
    - echo "Deploying to production server"
    - ssh user@your-server-ip "cd /path/to/deployment && git pull origin main"
  only:
    - main
```

### Step 4: Configure SSH Keys for GitLab

1. **Generate SSH Key** (if not already created):
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```
   Press Enter to accept defaults.

2. **Add SSH Key to GitLab**:
   - Copy the public key:
     ```bash
     cat ~/.ssh/id_rsa.pub
     ```
   - In GitLab, go to **Settings > SSH Keys** and paste the key.

3. **Copy SSH Key to Your Server**:
   ```bash
   ssh-copy-id user@your-server-ip
   ```

### Step 5: Install and Configure GitLab Runner

1. **Install GitLab Runner**:
   ```bash
   sudo apt install gitlab-runner -y
   ```

2. **Register the Runner**:
   - Obtain the registration token from your GitLab project under **Settings > CI/CD > Runners**.
   - Run:
     ```bash
     sudo gitlab-runner register
     ```
   - Follow prompts:
     - **GitLab URL**: `https://gitlab.com/`
     - **Token**: Your project’s token.
     - **Description**: Any description you prefer.
     - **Tags**: Optional tags for the runner.
     - **Executor**: Choose `shell` or `docker`.

### Step 6: Trigger the Pipeline

1. **Push Changes**:
   ```bash
   git add .
   git commit -m "Initial commit"
   git push origin main
   ```

2. **Monitor the Pipeline**:
   - Go to **CI/CD > Pipelines** in your GitLab repository to see the pipeline status.

### Step 7: Verify Deployment

- Check the production server to confirm that the application has been deployed correctly. Validate the deployment through the specified URL.

### Troubleshooting

- Check logs in GitLab CI/CD for errors.
- Ensure SSH keys are correctly configured.
- Review any firewall settings that might restrict access.

### Conclusion

You’ve successfully set up a Continuous Deployment pipeline using GitLab CI/CD on Ubuntu. Modify the `.gitlab-ci.yml` file as necessary to fit your application’s specific build, test, and deployment needs.
