# Use Node 18 image as the base
FROM node:18-bullseye

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json to install dependencies
COPY angular-site/package*.json ./

# Install Angular CLI globally and project dependencies
RUN npm install -g @angular/cli@15.0.3 && npm install 

# Copy the Angular application code
COPY angular-site/ .

# Expose the default Angular port
EXPOSE 4200

# Command to start the Angular application
CMD ["ng", "serve", "--host", "0.0.0.0"]
