# Build stage for building .ts files
FROM node:20-alpine as build

# Install git and bash (needed for husky)
RUN apk add --no-cache git bash

# Create the app directory
RUN mkdir /home/app

WORKDIR /home/app

# Copy package.json to install dependencies
COPY package.json .

# Install dependencies, including TypeScript and husky
RUN npm install --ignore-scripts

# Copy the rest of the application code
COPY . .

# Run the build command to compile TypeScript files
RUN npm run build

# Production stage for including only necessary files
FROM node:20-alpine as prod

# Install git and bash (needed for husky)
RUN apk add --no-cache git bash

LABEL org.opencontainers.image.source=https://github.com/ghoshRitesh12/aniwatch-api
LABEL org.opencontainers.image.description="Node.js API for obtaining anime information from aniwatch.to (formerly zoro.to) written in TypeScript, made with Cheerio & Axios"
LABEL org.opencontainers.image.licenses=MIT

# Create a non-privileged user
RUN addgroup -S aniwatch && adduser -S zoro -G aniwatch

# Set secure folder permissions
RUN mkdir -p /app/public /app/dist && chown -R zoro:aniwatch /app

# Switch to non-privileged user
USER zoro

# Set working directory
WORKDIR /app

# Copy package.json for dependency installation in production
COPY --chown=zoro:aniwatch package.json .

# Install only production dependencies (excluding dev dependencies like TypeScript)
RUN npm install --omit=dev --ignore-scripts

# Copy the build artifacts from the build stage
COPY --from=build --chown=zoro:aniwatch /home/app/public /app/public
COPY --from=build --chown=zoro:aniwatch /home/app/dist /app/dist

# Health check command
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s CMD [ "npm", "run", "healthcheck" ]

# Set environment variables
ENV NODE_ENV=production
ENV PORT=4000

# Expose the application port
EXPOSE 4000

# Command to start the app
CMD [ "node", "dist/src/server.js" ]
