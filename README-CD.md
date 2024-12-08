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
