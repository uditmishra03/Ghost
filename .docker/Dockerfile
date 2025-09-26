# --------------------
# Ghost CMS Production Build
# --------------------

# Use official Ghost image as base
FROM ghost:5-alpine

# Switch to root to install packages and copy files
USER root

# Update packages for security and install curl for healthcheck
RUN apk update && apk upgrade && \
    apk add --no-cache curl && \
    rm -rf /var/cache/apk/*

# Copy Ghost configuration with proper ownership
COPY --chown=node:node config.production.json /var/lib/ghost/config.production.json

# Ensure content directories exist with proper permissions
RUN mkdir -p /var/lib/ghost/content && \
    chown -R node:node /var/lib/ghost/content

# Switch back to node user for security
USER node

# Set working directory
WORKDIR /var/lib/ghost

# Expose Ghost's port
EXPOSE 2368

# Enhanced healthcheck for production readiness
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD curl -f http://localhost:2368/ghost/api/v4/admin/site/ || exit 1

# Use default Ghost entrypoint and command
