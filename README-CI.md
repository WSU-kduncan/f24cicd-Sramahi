# CI Project Overview

In this project, I used Docker to containerize an Angular application and configured GitHub Actions for continuous integration. After that, the application is deployed to DockerHub, where it is accessible and executable. This project's primary tools are:

- **Docker**: Used to create and manage containers for the Angular app.
- **GitHub Actions**: Used to automate the CI/CD pipeline for the project.
- **DockerHub**: Used to store and share the container image.

This project helps in learning how to manage apps using containers and run them in isolated contexts.

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

## Link to DockerHub Repository
You can find the Docker image for this project on DockerHub at the following link: 
https://hub.docker.com/repository/docker/suhayb02/al-ramahi-ceg3120/general


