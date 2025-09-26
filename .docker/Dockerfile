# --------------------
# Multi-stage build for TypeScript/Node.js Ghost project
# --------------------

# Build stage for TypeScript compilation
FROM node:18-alpine AS builder

# Install build dependencies
RUN apk add --no-cache python3 make g++ && \
    npm install -g yarn typescript

WORKDIR /app

# Copy package files
COPY package.json yarn.lock ./
COPY tsconfig.json ./

# Install dependencies
RUN yarn install --frozen-lockfile

# Copy source code
COPY src/ ./src/

# Build TypeScript
RUN yarn build

# Runtime stage with Ghost
FROM ghost:5-alpine AS runtime

# Update packages to fix vulnerabilities
USER root
RUN apk update && apk upgrade && \
    apk add --no-cache dumb-init curl && \
    rm -rf /var/cache/apk/*

# Set working directory
WORKDIR /var/lib/ghost

# Copy built application from builder stage
COPY --from=builder /app/dist /var/lib/ghost/content/themes/custom/
COPY --from=builder /app/package.json /var/lib/ghost/
COPY --from=builder /app/node_modules /var/lib/ghost/node_modules/

# Copy Ghost configuration
COPY config.production.json /var/lib/ghost/config.production.json

# Ensure correct permissions (use 'node' user from Ghost image)
RUN chown -R node:node /var/lib/ghost

# Switch to non-root user
USER node

# Expose Ghost's port
EXPOSE 2368

# Healthcheck to monitor Ghost's availability
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:2368 || exit 1

# Use dumb-init for proper signal handling
ENTRYPOINT ["dumb-init", "--"]

# Start Ghost
CMD ["node", "current/index.js"]
