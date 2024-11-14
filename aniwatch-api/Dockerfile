# Build stage for building .ts files
FROM node:20-alpine as build

WORKDIR /home/app

# Copy package files for installing dependencies
COPY package.json package-lock.json ./
RUN npm install --ignore-scripts

# Copy all source files
COPY . .

# Build TypeScript files
RUN npm run build

# Production stage for including only necessary files
FROM node:20-alpine as prod

LABEL org.opencontainers.image.source=https://github.com/ghoshRitesh12/aniwatch-api
LABEL org.opencontainers.image.description="Node.js API for obtaining anime information from aniwatch.to (formerly zoro.to) written in TypeScript, made with Cheerio & Axios"
LABEL org.opencontainers.image.licenses=MIT

# Create a non-privileged user
RUN addgroup -S aniwatch && adduser -S zoro -G aniwatch

# Set secure folder permissions
RUN mkdir -p /app/public /app/dist && chown -R zoro:aniwatch /app

# Set non-privileged user
USER zoro

# Set working directory
WORKDIR /app

# Copy package files again for production
COPY --chown=zoro:aniwatch package.json package-lock.json ./

# Install production dependencies
RUN npm install --omit=dev --ignore-scripts

# Copy the necessary files from the build stage
COPY --from=build --chown=zoro:aniwatch /home/app/public /app/public
COPY --from=build --chown=zoro:aniwatch /home/app/dist /app/dist

# Healthcheck to ensure the app is running
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s CMD ["npm", "run", "healthcheck"]

# Environment variables and exposed port
ENV NODE_ENV=production
ENV PORT=4000
EXPOSE 4000

# Start the app
CMD ["node", "dist/src/server.js"]
