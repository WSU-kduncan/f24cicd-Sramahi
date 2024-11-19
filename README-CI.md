# CI Project Overview

In this project, I used Docker to containerize an Angular application and configured GitHub Actions for continuous integration. After that, the application is deployed to DockerHub, where it is accessible and executable. This project's primary tools are:

- **Docker**: Used to create and manage containers for the Angular app.
- **GitHub Actions**: Used to automate the CI/CD pipeline for the project.
- **DockerHub**: Used to store and share the container image.

## Containerizing Your Application

### How to Install Docker

#### For Windows:
1. Go to Docker's website and download Docker Desktop.
2. Follow the installation instructions to set it up.
3. After installing, check Docker's version with:
    ```plaintext
    docker --version

### How to Build & Configure a Container
1. Create the Dockerfile: I created a Dockerfile in my GitHub repo with the following commands:
   ```plaintext
    # Use Node.js version 18 image as the base
    FROM node:18-bullseye
    
    # Set the working directory in the container
    WORKDIR /app
    
    # Copy package.json and package-lock.json (if present) to the container
    COPY package*.json ./
    
    # Install the Angular CLI globally and project dependencies
    RUN npm install -g @angular/cli@15.0.3 && npm install
    
    # Copy all other application files to the container
    COPY . .
    
    # Expose port 4200 to allow access outside the container
    EXPOSE 4200
    
    # Start the Angular application
    CMD ["ng", "serve", "--host", "0.0.0.0"]

  - FROM node:18-bullseye: This command sets the base image to Node 18, which includes everything needed to run an Angular app.
  - WORKDIR /app: This command sets the working directory inside the container where the app will live.
  - COPY: The COPY command copies the app code and dependencies into the container.
  - RUN npm install: This installs all the necessary dependencies to run the Angular app.
  - CMD ["ng", "serve", "--host", "0.0.0.0"]: This command runs the Angular app when the container starts.

2. How to Build the Image: To build the Docker image from the Dockerfile, I ran the following command in the terminal:
   ```plaintext
   docker build -t yourlastname-ceg3120 .
This command tells Docker to create an image with the tag yourlastname-ceg3120.

3. How to Run the Container: After building the image, I ran the container with this command:
   ```plaintext
   docker run -p 4200:4200 yourlastname-ceg3120
This starts the Angular app inside the container and makes it accessible through http://localhost:4200.

4. How to View the Application: Open a web browser and go to http://localhost:4200. You should see the Angular application running.

## Working with DockerHub

### How to Create a Public Repo in DockerHub

1. Go to DockerHub.
2. Log in with your DockerHub account (create one if you don’t have it).
3. Click on “Create Repository.”
4. Name the repository YOURLASTNAME-CEG3120.
5. Select Public so everyone can see it.

### How to Authenticate with DockerHub via CLI
- To push the image to DockerHub, I needed to log in through the command line. I ran:
   ```plaintext
   docker login -u your-dockerhub-username
This will ask for your DockerHub username and password. Once logged in, you can push the image to DockerHub.

### How to Push Container Image to DockerHub

1. Tag the Image: To push the image to DockerHub, I first tagged it with the following command:
   ```plaintext
   docker tag yourlastname-ceg3120 your-dockerhub-username/yourlastname-ceg3120:latest
2. Push the Image: After tagging the image, I pushed it to DockerHub with:
   ```plaintet
   docker push your-dockerhub-username/yourlastname-ceg3120:latest

![Docker_image_code](https://github.com/user-attachments/assets/9774996c-097c-4394-a452-9990a75e206e)
![app_running_code](https://github.com/user-attachments/assets/9e5775fa-31e3-4258-822c-af21e526ec80)
![app_running](https://github.com/user-attachments/assets/b7fcf7b0-6f07-4af2-aea5-220ced191b41)
![docker_code](https://github.com/user-attachments/assets/4314a201-e198-4408-a452-b0032611ad58)
![Docker_push_code](https://github.com/user-attachments/assets/86b3f8b2-9509-4e42-a5ff-e55edd5bc8b0)
![Docker_tag](https://github.com/user-attachments/assets/2ef1ed9e-0c98-47e2-9da0-3a9b116d589e)





## Link to DockerHub Repository
You can find the DockerHub at the following link: 
https://hub.docker.com/repository/docker/suhayb02/al-ramahi-ceg3120/general


# CI/CD with GitHub Actions and DockerHub
In this part, we automate the creation of a Docker image and push it to DockerHub in this section using GitHub Actions.

## Configuring GitHub Secrets

You must create a DockerHub Access Token in order to securely communicate with DockerHub through GitHub Actions. Pushing and pulling Docker images will be authenticated using this token.
GitHub Secrets are used to store private data, such as your access token and DockerHub username. Your GitHub Actions workflow then safely references these secrets.

1. How to Set a Secret for Use by GitHub Actions

   - Go to your DockerHub account.
   - Click on your profile picture in the top-right corner and select Account Settings.
   - Under Security, find the Access Tokens section and click Create Token.
   - Provide a name for the token
   - Click Generate, then copy the generated token and save it securely.

2. Set GitHub Secrets
   - Go to your GitHub repository.
   - Click on Settings and scroll down to Secrets and variables, then select Actions.
   - Click New repository secret to add a new secret.
       - DOCKER_USERNAME: Your DockerHub username 
       - DOCKER_TOKEN: The access token that you created in DockerHub.

## Behavior of the GitHub Workflow
### Summary of Workflow Actions
The GitHub Actions workflow is set up to:

1. Build the Docker image using the Dockerfile in the repository.
2. Tag the image with the latest commit.
3. Push the Docker image to DockerHub automatically.

This set-up provides that a new Docker image is produced and updated on DockerHub every time code is pushed to the repository.

### Link to the Workflow File
https://github.com/WSU-kduncan/f24cicd-Sramahi/tree/main/.github/workflows

### Setting Up the GitHub Actions Workflow
Here’s a breakdown of how I set up the GitHub Actions workflow. I created a YAML file docker-image.yml in .github/workflows/ with the following configuration:

     ```plaintext
    name: Build and Push Docker Image
    
    on:
      push:
        branches:
          - main
      pull_request:
        branches:
          - main
    
    jobs:
      build:
        runs-on: ubuntu-latest
    
        steps:
          # Checkout the repository code
          - name: Checkout repository
            uses: actions/checkout@v3
    
          # Cache Docker layers to speed up builds
          - name: Cache Docker layers
            uses: actions/cache@v3
            with:
              path: /tmp/.buildx-cache
              key: ${{ runner.os }}-buildx-${{ github.sha }}
              restore-keys: |
                ${{ runner.os }}-buildx-
          # Log in to DockerHub using credentials from GitHub Secrets
          - name: Log in to DockerHub
            uses: docker/login-action@v3
            with:
              username: ${{ secrets.DOCKER_USERNAME }}
              password: ${{ secrets.DOCKER_TOKEN }}
    
          # Build the Docker image from the specific Dockerfile path
          - name: Build Docker image
            run: |
              docker build -f angular-site/wsu-hw-ng-main/Dockerfile -t ${{ secrets.DOCKER_USERNAME }}/suhayb:latest angular-site/wsu-hw-ng-main
          # Push the Docker image to DockerHub
          - name: Push Docker image
            run: |
              docker push ${{ secrets.DOCKER_USERNAME }}/suhayb:latest

### Configuring to Duplicate the Workflow

If another user wants to use this workflow, they would need to:

1. Update the DockerHub repository name (replace yourlastname-ceg3120 with their repository name in the tags field).
2. Set up their own GitHub Secrets with the following names:
   - DOCKER_USERNAME: Their DockerHub username.
   - DOCKER_TOKEN: Their DockerHub Access Token with Read/Write access.

![Verify(part2)](https://github.com/user-attachments/assets/049c7bc6-dabc-4e5c-acf3-2d538fb6c2aa)
![app_running](https://github.com/user-attachments/assets/6bfc9c4d-3dd3-4e55-8474-eda675b7a59d)


## Resources
## Resources

1. [Docker Documentation: CICD with GitHub Actions](https://docs.docker.com/build/ci/github-actions/)
2. [GitHub Actions - build-push-action Documentation](https://github.com/marketplace/actions/build-and-push-docker-images)
3. [GitHub Docs - Publishing Docker Images to DockerHub](https://docs.github.com/en/actions/use-cases-and-examples/publishing-packages/publishing-docker-images#publishing-images-to-docker-hub)
4. [Mermaid - new markdown feature](https://github.blog/developer-skills/github/include-diagrams-markdown-files-mermaid/)
5. [Dockerfile Reference](https://docs.docker.com/reference/dockerfile/)
6. [Using secrets in GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions)
7. [Build and push your first image](https://docs.docker.com/get-started/introduction/build-and-push-first-image/)

# Diagram
![Diagram](https://github.com/user-attachments/assets/2ebd246e-54bd-47a3-a971-2764339cff66)

This diagram was built using Mermaid.


    

