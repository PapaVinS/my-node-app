# Node.js Docker GitHub Actions Workflow

This repository contains a simple Node.js application with a CI/CD pipeline that automatically builds and runs the application in a Docker container when code is pushed to the `staging` branch.

## How the GitHub Workflow Works

The workflow is defined in `.github/workflows/staging.yml` and performs the following steps:

1. **Trigger**: The workflow is triggered when code is pushed to the `staging` branch.

2. **Environment**: It runs on a self-hosted runner (which you need to set up and maintain yourself).

3. **Steps**:
   - **Checkout**: First, it checks out the repository code.
   - **Setup Docker Compose**: Verifies Docker Compose is available.
   - **Run with Docker Compose**: Builds and runs the application using Docker Compose.
   - **Display Logs**: Shows the container logs to verify the application started successfully.

4. **Environment Variables**: The workflow uses environment variables defined in the GitHub repository secrets (like API_KEY) and passes them to the Docker container.

## Workflow Configuration

```yaml
name: Node.js Docker CI/CD

on:
  push:
    branches: [ staging ]

jobs:
  build-and-run:
    runs-on: self-hosted
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
        
      - name: Set up Docker Compose
        run: |
          # Ensure docker-compose is available
          docker compose version
      
      - name: Run with Docker Compose
        run: |
          docker compose up -d
        env:
          # Secrets key here if needed
          API_KEY: ${{ secrets.API_KEY }}
      
      - name: Display logs
        run: |
          # Wait for app to start
          sleep 5
          # Show logs
          docker-compose logs app
```

## Assumptions

1. **Self-Hosted Runner**: You have a self-hosted GitHub Actions runner set up and running.

2. **Docker and Docker Compose**: Your runner has Docker and Docker Compose installed and configured.

3. **Permission Setup**: The user running the self-hosted runner has permissions to use Docker (is in the docker group).

4. **Repository Structure**: Your repository has the following files:
   - `Dockerfile.dev`: Defines how to build the Node.js application image
   - `docker-compose.yml`: Defines the services (including the Node.js app)
   - `package.json`: Defines Node.js dependencies
   - `app.js`: The main application file

5. **GitHub Secrets**: If your application requires any sensitive information (like API keys), they are set up in GitHub repository secrets.

## Troubleshooting

### Permission Denied

If you see an error like:
```
unable to get image 'node-app:latest': permission denied while trying to connect to the Docker daemon socket
```

Fix it by adding the runner user to the docker group:
```bash
sudo usermod -aG docker $USER
newgrp docker   # Apply without logout
# Then restart the runner
sudo ./svc.sh stop
sudo ./svc.sh start
```

### Docker Compose Not Found

If Docker Compose isn't available, install it:
```bash
sudo apt-get update
sudo apt-get install docker-compose-plugin
```

## Local Testing

You can test the Docker setup locally before pushing to GitHub:

1. Make sure Docker and Docker Compose are installed
2. Run `docker-compose up -d` in the repository root
3. Check logs with `docker-compose logs`
4. Access the application at http://localhost:3000
