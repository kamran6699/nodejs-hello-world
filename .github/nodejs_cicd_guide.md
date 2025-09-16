## 1. Project Setup: Node.js Application

First, you need to create a directory for your project and set up the basic Node.js application files.

1.  **Create Project Directory and Navigate into it:**

    ```bash
    mkdir nodejs-hello-world
    cd nodejs-hello-world
    ```

2.  **Create `app.js`:** This file will contain your Node.js "Hello World" server.

    ```javascript
    // app.js
    const http = require('http');

    const hostname = '0.0.0.0';
    const port = 3000;

    const server = http.createServer((req, res) => {
      res.statusCode = 200;
      res.setHeader('Content-Type', 'text/plain');
      res.end('Hello World from Node.js!\n');
    });

    server.listen(port, hostname, () => {
      console.log(`Server running at http://${hostname}:${port}/`);
    });
    ```

3.  **Create `package.json`:** This file defines your project's metadata and scripts.

    ```json
    // package.json
    {
      "name": "nodejs-hello-world",
      "version": "1.0.0",
      "description": "A simple Node.js Hello World application",
      "main": "app.js",
      "scripts": {
        "start": "node app.js"
      },
      "author": "Your Name",
      "license": "ISC"
    }
    ```

## 2. Docker Configuration

Next, you'll create the necessary Docker files to containerize your Node.js application.

1.  **Create `Dockerfile`:** This file contains instructions for building your Docker image.

    ```dockerfile
    # Dockerfile
    FROM node:18-alpine

    WORKDIR /app

    COPY package*.json ./

    RUN npm install

    COPY . .

    EXPOSE 3000

    CMD [ "npm", "start" ]
    ```

2.  **Create `.dockerignore`:** This file specifies files and directories that should be excluded when building the Docker image, similar to `.gitignore`.

    ```
    # .dockerignore
    node_modules
npm-debug.log
.git
.gitignore
    ```

## 3. GitHub Repository Setup

Now, you'll initialize a Git repository, create a new repository on GitHub, and push your local code.

1.  **Initialize Git Repository:**

    ```bash
    git init
    git add .
    git commit -m "Initial commit: Node.js Hello World app with Dockerfile"
    ```

2.  **Create a New Repository on GitHub:**
    *   Go to [GitHub](https://github.com/).
    *   Click the '+' icon in the top right corner and select 'New repository'.
    *   Give your repository a name (e.g., `nodejs-hello-world`).
    *   Choose whether it's public or private.
    *   **DO NOT** initialize with a README, .gitignore, or license, as you've already created your files locally.
    *   Click 'Create repository'.

3.  **Push Local Code to GitHub:**
    After creating the repository on GitHub, you will see instructions on how to push an existing local repository. It will look something like this (replace `<YOUR_GITHUB_USERNAME>` and `<YOUR_REPO_NAME>` with your actual details):

    ```bash
    git remote add origin https://github.com/<YOUR_GITHUB_USERNAME>/<YOUR_REPO_NAME>.git
    git branch -M main
    git push -u origin main
    ```

    **Note:** You might be prompted for your GitHub username and Personal Access Token (PAT) instead of a password. If you haven't created a PAT, follow these steps:
    *   Go to your GitHub profile settings.
    *   Navigate to `Developer settings` -> `Personal access tokens` -> `Tokens (classic)`.
    *   Click `Generate new token`.
    *   Give it a descriptive name (e.g., `manus-agent-cicd`).
    *   Select the `repo` scope.
    *   Click `Generate token` and copy the token. **Treat this token like a password and do not share it publicly.**

## 4. Setup CI/CD Pipeline with GitHub Actions

This section details how to set up a GitHub Actions workflow to automatically build your Docker image and push it to Docker Hub whenever you push changes to your GitHub repository.

1.  **Create GitHub Actions Workflow File:**
    You need to create a directory `.github/workflows` in your project root and then create a YAML file inside it (e.g., `docker-publish.yml`). This file defines your CI/CD pipeline.

    ```bash
    mkdir -p .github/workflows
    ```

    Then, create the `docker-publish.yml` file with the following content:

    ```yaml
    # .github/workflows/docker-publish.yml
    name: Docker Image CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

env:
  DOCKER_HUB_USERNAME: ${{ secrets.DOCKER_HUB_USERNAME }}
  DOCKER_HUB_TOKEN: ${{ secrets.DOCKER_HUB_TOKEN }}
  IMAGE_NAME: nodejs-hello-world

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Log in to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ env.DOCKER_HUB_USERNAME }}
        password: ${{ env.DOCKER_HUB_TOKEN }}

    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: ${{ env.DOCKER_HUB_USERNAME }}/${{ env.IMAGE_NAME }}:latest
    ```

2.  **Set up Docker Hub Secrets in GitHub:**
    For your GitHub Actions workflow to log in to Docker Hub, you need to store your Docker Hub username and access token as GitHub Secrets.

    *   **Generate a Docker Hub Access Token:**
        *   Go to [Docker Hub](https://hub.docker.com/) and log in.
        *   Navigate to `Account Settings` -> `Security` -> `New Access Token`.
        *   Give it a name (e.g., `github-actions-token`).
        *   Grant it `Read & Write` permissions.
        *   Click `Generate` and copy the token. **This token will only be shown once.**

    *   **Add Secrets to your GitHub Repository:**
        *   Go to your GitHub repository.
        *   Click on `Settings` -> `Secrets and variables` -> `Actions`.
        *   Click `New repository secret`.
        *   Create a secret named `DOCKER_HUB_USERNAME` and paste your Docker Hub username as the value.
        *   Click `New repository secret` again.
        *   Create a secret named `DOCKER_HUB_TOKEN` and paste the Docker Hub access token you generated as the value.

3.  **Commit and Push the Workflow File:**

    ```bash
    git add .github/workflows/docker-publish.yml
    git commit -m "Add GitHub Actions workflow for Docker Hub CI/CD"
    git push origin main
    ```
    This push will trigger your first GitHub Actions workflow run.

## 5. Test and Verify Deployment

After pushing your code, GitHub Actions will automatically run the workflow. You can monitor its progress and verify the deployment.

1.  **Monitor GitHub Actions:**
    *   Go to your GitHub repository.
    *   Click on the `Actions` tab.
    *   You should see a workflow run in progress or completed. Click on it to view the logs and ensure all steps passed successfully.

2.  **Verify Image on Docker Hub:**
    *   Go to [Docker Hub](https://hub.docker.com/) and log in.
    *   Navigate to your repositories.
    *   You should see a new repository named `nodejs-hello-world` (or whatever you named your `IMAGE_NAME` in the workflow file) with the `latest` tag.

3.  **Run the Docker Image Locally (Optional):**
    To verify that the Docker image works correctly, you can pull it from Docker Hub and run it locally:

    ```bash
    docker pull <YOUR_DOCKER_HUB_USERNAME>/nodejs-hello-world:latest
    docker run -p 3000:3000 <YOUR_DOCKER_HUB_USERNAME>/nodejs-hello-world:latest
    ```
    Then, open your web browser and go to `http://localhost:3000`. You should see "Hello World from Node.js!".

This completes the guide for setting up your Node.js Hello World application with a CI/CD pipeline to Docker Hub.



