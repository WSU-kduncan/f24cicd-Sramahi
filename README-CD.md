# Continuous Deployment (CD) Project

## Project Overview

In this project, we're setting up a Continuous Deployment (CD) pipeline using GitHub Actions to automatically build and push Docker images to DockerHub. This process is designed to streamline the deployment of our project by automating the build and deployment steps every time changes are made.

**Why are we doing this?**
Automating the process of building and pushing Docker images ensures that our application is always up-to-date, tested, and ready to be deployed without needing to do the manual work every time changes are made. This saves time, reduces errors, and ensures consistency in our deployments.

**Tools used:**
- **GitHub Actions**: Automates the CI/CD pipeline.
- **Docker**: To package the application into containers.
- **DockerHub**: Hosts our Docker images for easy access and deployment.
- **Git**: Manages the source code and versioning.

---

## How to Generate and Push a Tag in Git

To generate and push a tag to Git, follow these steps:

1. **Create a tag**:
   - First, make sure you are on the correct branch (e.g., `main`).
   - Run the following command to create a tag:
     ```bash
     git tag v1.0.0
     ```
     You can replace `v1.0.0` with your desired version number following semantic versioning (e.g., `v1.1.0`, `v2.0.0`, etc.).

2. **Push the tag**:
   - Push the tag to the remote repository by running:
     ```bash
     git push origin v1.0.0
     ```
   - This will push the tag to GitHub, triggering the GitHub Actions workflow.

---

## Behavior of GitHub Workflow

### When does the GitHub Actions workflow run?

The workflow runs automatically when:
1. **A push is made to the `main` branch.**
2. **A new tag is pushed.** This includes version tags like `v1.0.0`, `v1.1.0`, etc.

### What does the workflow do?

Once the workflow is triggered, it goes through the following steps:
1. **Checkout the code**: It retrieves the code from the repository to build the Docker image.
2. **Cache Docker layers**: It speeds up the build process by reusing previously built layers.
3. **Log in to DockerHub**: It authenticates to DockerHub using the credentials stored in GitHub Secrets.
4. **Generate Docker image metadata**: It tags the Docker image based on the version or tag (`latest`, `v1.0.0`, etc.).
5. **Build the Docker image**: It builds the Docker image using the Dockerfile present in the project.
6. **Push the Docker image**: After the build is complete, it pushes the image to DockerHub under the configured repository and tags.

---

## Link to DockerHub Repository
[DockerHub - Suhayb's Repository](https://hub.docker.com/repository/docker/suhayb02/suhayb/general)


## Instance Information

- **Public IP**: `98.84.157.114`
- **OS**: Amazon Linux 2

## How to Install Docker on the Instance

### Step 1: Install Docker
On your EC2 instance, run the following commands to install Docker (for Amazon Linux 2):
   - sudo yum update -y
   - sudo yum install docker -y
 
### Step 2: Start Docker Service
Start Docker and enable it to start on boot:
   - sudo systemctl start docker
   - sudo systemctl enable docker

### Step 3: Verify Docker Installation
To check if Docker is installed and running, run: 
   - docker --version

## Bash Script

1. Purpose: The bash script is responsible for pulling the latest Docker image from your DockerHub repository, stopping and removing the old container, and starting a new container with the updated image.
2. Script Taskings:
     - Pull the latest Docker image from your DockerHub repository.
     - Stop and remove any running containers based on the current image.
     - Start a new container with the freshly pulled image.
3. Location on Instance Filesystem: The bash script is located in the nano /home/ec2-user/deploy.sh
4. link: https://github.com/WSU-kduncan/f24cicd-Sramahi/blob/main/deployment/bash
   
## Purpose of Installing adnanh's Webhook

The purpose of installing adnanh’s webhook is to listen for incoming requests (via HTTP POST) that will trigger the bash script (update_container.sh). This allows the EC2 instance to automatically update and restart the Docker container when a message is received from GitHub or DockerHub.

### Steps to Install and Setup Webhook on the EC2 Instance:
1. Install webhook: Download and install the webhook program on the EC2 instance:
     - sudo yum install -y golang
     - go install github.com/adnanh/webhook@latest
2. Configure webhook to run bash script: You need to create a webhook configuration file (hook definition file) to define what actions the webhook will trigger.
     - [
  {
    "id": "deploy-hook",
    "execute-command": "/home/ec2-user/deploy.sh",
    "command-working-directory": "/home/ec2-user/"
  }
]
3. Start Webhook Listener (Without using Service): To manually start the webhook listener, run the following command:
      - ~/go/bin/webhook -hooks hooks.json -verbose &
4. Test the Listener: To test that the webhook listener works and triggers the script, use curl to send a request:
      - curl -X POST http://localhost:9000/hooks/deploy-hook
5. Monitoring Webhook Logs: Monitor logs generated by the webhook using the journalctl command to see activity:
      -  sudo journalctl -u webhook -f

## Webhook / Hook Task Definition File
The hook task definition file (hooks.json) is used to define:
1. The unique hook ID to trigger the deployment process.
2. The command to execute when the webhook is triggered (e.g., running the deploy.sh script).
3. The working directory from which the script will execute.

When the webhook listener receives a POST request matching the hook ID, it automatically runs the associated command, ensuring that the deployment process (pulling the latest Docker image, restarting the container, etc.) is performed seamlessly.

Location on Instance Filesystem: located in the nano /home/ec2-user/hooks.json

link: https://github.com/WSU-kduncan/f24cicd-Sramahi/blob/main/deployment/webhook%20/hook

## Configure GitHub or DockerHub to Message the Listener
### GitHub:
1. Go to your GitHub repository.
2. Navigate to Settings > Webhooks.
3. Click Add Webhook.
      - Payload URL: http://<INSTANCE_PUBLIC_IP>:9000/hooks/deploy-hook
        Replace <INSTANCE_PUBLIC_IP> with the public IP of your instance.
      - Content Type: application/json.
      - Secret: Leave blank or set if using a secret in your hooks.json.
      - Event Type: Select Push events or customize as needed.
      - Click Add Webhook.

### DockerHub Configuration
1. Log in to your DockerHub account.
2. Go to Repositories and select your repository.
3. Navigate to Webhooks > Add Webhook.
      - Webhook URL: http://<INSTANCE_PUBLIC_IP>:9000/hooks/deploy-hook.
      - Save the webhook.
  
## How to Modify or Create a Webhook Service File

To ensure that the webhook listener starts automatically when the instance boots, modify or create a systemd service file:
1. Create the service file sudo nano /etc/systemd/system/webhook.service
2. Add the following content
   - [Unit]
Description=Webhook listener
After=network.target

[Service]
User=ec2-user
ExecStart=/home/ec2-user/go/bin/webhook -hooks /home/ec2-user/hooks.json -verbose
Restart=always
WorkingDirectory=/home/ec2-user
Environment=PATH=/home/ec2-user/go/bin:$PATH

[Install]
WantedBy=multi-user.target

3. Save the file and exit.
4. Reload the systemd configuration and start the service:
      - sudo systemctl daemon-reload
      - sudo systemctl enable webhook
      - sudo systemctl start webhook
5. link: https://github.com/WSU-kduncan/f24cicd-Sramahi/blob/main/deployment/webhook

## Debugging Steps (from Logs and Process View)
1. Logs: If you encounter issues, check the webhook logs to see any errors:
      - sudo journalctl -u webhook -f
2. Docker Process: Verify that the Docker container is running correctly:
      - docker ps
3. Port Conflicts: If you see errors like "address already in use", check if the port (9000) is already in use by another process:
      - sudo lsof -i :9000
4. Webhook Not Triggering: Ensure the webhook secret matches between the GitHub/DockerHub configuration and the hooks.json file.
5. When running the following Docker command:
      - docker run --name app -d -p 80:4200 suhayb02/suhayb:v1.0.0
   you may encounter this:
      - docker: Error response from daemon: Conflict. The container name "/app" is already in use by container "43bf756701a94e09bda6894364e63f18ca9f17944a3b67440170d245d9dc06ff". You have to remove (or rename) that container to be able to reuse that name.

To resolve this:
   - Stop the existing container: docker stop app
   - Remove the existing container: docker rm app
   - Run the docker run command again: docker run --name app -d -p 80:4200 suhayb02/suhayb:v1.0.0

## Images

![Screenshot 2024-12-08 135056](https://github.com/user-attachments/assets/f089323e-0e78-4dc9-abf6-6904039e8055)
![Screenshot 2024-12-08 135312](https://github.com/user-attachments/assets/9f1e6af5-52a4-48f7-9949-adaf16226cc1)
![Screenshot 2024-12-08 140354](https://github.com/user-attachments/assets/800fe654-826c-4bec-99fc-cc0b069ec973)
![Screenshot 2024-12-08 145951](https://github.com/user-attachments/assets/6fa3cfa6-81a1-4dfb-aa34-4b7c068faa21)
![Screenshot 2024-12-08 151735](https://github.com/user-attachments/assets/26763541-7ed5-416d-8ecf-12123f619b00)
![Screenshot 2024-12-08 151811](https://github.com/user-attachments/assets/07fc34e2-794d-4146-8117-16ea1977952f)
![Screenshot 2024-12-08 152051](https://github.com/user-attachments/assets/8e4efaf3-ec41-4706-ab9d-3ed3e9286ac6)
![Screenshot 2024-12-08 152108](https://github.com/user-attachments/assets/a6d7b900-d5e1-4986-88c1-95d528677012)
![Screenshot 2024-12-08 152144](https://github.com/user-attachments/assets/136e6ce7-d913-4815-a078-7c6b459f4429)
![Screenshot 2024-12-08 153638](https://github.com/user-attachments/assets/08a78ff3-522b-4ff2-9d6f-dac7b3bb4988)
![Screenshot 2024-12-08 185421](https://github.com/user-attachments/assets/9bdab8d8-5f95-4838-928b-e78bf4a1b050)
![Screenshot 2024-12-08 194819](https://github.com/user-attachments/assets/2ef4bc7e-dbdf-4afe-b0ed-902ca17dfeee)

## Diagrams
![Screenshot 2024-12-11 190638](https://github.com/user-attachments/assets/55f6c74f-e966-4d9c-befa-3878c9ed3a19)

![Screenshot 2024-12-11 190700](https://github.com/user-attachments/assets/84f3d8a8-ab70-446e-867c-24f22b9e0e85)

![Screenshot 2024-12-11 190740](https://github.com/user-attachments/assets/1a35b024-9ee7-4b87-846a-dbabea9bf8af)


## Resources

1. [Adnanh's Webhook Documentation](https://github.com/adnanh/webhook) - Official repository and documentation for setting up and using the webhook listener.
2. [Using GitHub Actions and Webhooks](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows) - Comprehensive guide to using GitHub webhooks to trigger workflows and actions.
3. [Using DockerHub and Webhooks](https://docs.docker.com/docker-hub/webhooks/) - Learn how to set up DockerHub webhooks to trigger updates or deployments.
4. [GitHub - docker/metadata-action](https://github.com/docker/metadata-action) - Official GitHub action for managing Docker metadata, such as tagging images.
5. [Docker - Manage Tag Labels](https://docs.docker.com/build/ci/github-actions/manage-tags-labels/) - Docker documentation on managing image tags and labels for better organization.
6. [Docker Container Best Practices](https://docs.docker.com/develop/dev-best-practices/) - Best practices for creating, managing, and deploying Docker containers.
7. [GitHub Webhooks Overview](https://docs.github.com/en/webhooks) - Understand GitHub webhooks, their payloads, and how to configure them.
8. [Automated Deployment Using Docker, GitHub Actions, and Webhooks](https://levelup.gitconnected.com/automated-deployment-using-docker-github-actions-and-webhooks-54018fc12e32) - Step-by-step tutorial on setting up automated deployments with Docker and webhooks.
9. [Building Your First CI/CD Pipeline with Docker, GitHub Actions, and Webhooks](https://blog.devgenius.io/build-your-first-ci-cd-pipeline-using-docker-github-actions-and-webhooks-while-creating-your-own-da783110e151) - A guide to creating a full CI/CD pipeline while integrating Docker and GitHub.
10. [Linux Handbook - How to Create a systemd Service](https://linuxhandbook.com/create-systemd-services/) - Step-by-step guide to creating and managing `systemd` services in Linux.
11. The diagrams were built using Mermaid.



