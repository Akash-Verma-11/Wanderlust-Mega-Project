# ------------------- Stage 1: Build Stage ------------------------------
FROM node:21 AS frontend-builder

# Set the working directory to /app
WORKDIR /app

# Copy the package.json and package-lock.json for dependency installation
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application code
COPY . .

# ------------------- Stage 2: Final Stage ------------------------------
FROM node:21-slim

# Set the working directory to /app
WORKDIR /app

# Copy built assets and change ownership to the non-root 'node' user
COPY --chown=node:node --from=frontend-builder /app .

# Copy the .env.sample file to .env.local with correct ownership
COPY --chown=node:node .env.docker .env.local

# Explicitly switch to the non-root user to satisfy Trivy DS-0002
USER node

# Expose port 5173 for the Node.js application
EXPOSE 5173

# Define the default command to run the application in development mode
CMD ["npm", "run", "dev", "--", "--host"]
