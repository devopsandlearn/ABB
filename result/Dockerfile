# Base image
FROM node:18-slim

# Set the working directory
WORKDIR /usr/local/app

# Copy package files and install dependencies
COPY package*.json ./
RUN npm ci --production

# Copy the application code
COPY . .

# Set environment variable for the application port
ENV PORT=80
EXPOSE 80

# Define the command to start the application
CMD ["node", "server.js"]
