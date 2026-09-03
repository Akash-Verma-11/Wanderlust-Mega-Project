# Stage 1
FROM node:21 AS backend-builder

# setup the working dir
WORKDIR /app

# code
COPY . .

# packages install
RUN npm i

# tests
RUN npm run test

# Stage 2
FROM node:21-slim

# setup the working dir
WORKDIR /app

# Copy the above stage and change ownership to the non-root 'node' user
COPY --chown=node:node --from=backend-builder /app .
COPY --chown=node:node .env.docker .env

# Explicitly switch to the non-root user to satisfy Trivy DS-0002
USER node

# Port
EXPOSE 8080

# App
CMD ["npm", "start"]
