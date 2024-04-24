# Use the official Node.js image as base
FROM node:alpine

# Set the working directory in the container
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# # Install dependencies
RUN npm install

# Copy the rest of the application
COPY * .

# Set environment variables
ENV PORT=8080

# Expose port 808 to the outside world
EXPOSE 8080

# Command to run the application
CMD ["node", "server.js"]
# CMD ["sleep", "infinity"]
