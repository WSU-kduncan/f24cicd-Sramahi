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
