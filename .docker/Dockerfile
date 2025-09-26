# --------------------
# Minimal Ghost Production Image
# --------------------

FROM ghost:5-alpine

# Switch to root for configuration
USER root

# Update packages and install curl for healthcheck  
RUN apk update && apk upgrade && \
    apk add --no-cache curl && \
    rm -rf /var/cache/apk/*

# Copy custom configuration
COPY --chown=node:node config.production.json /var/lib/ghost/config.production.json

# Ensure proper permissions on content directory
RUN chown -R node:node /var/lib/ghost/content

# Switch back to node user for security
USER node

# Set working directory
WORKDIR /var/lib/ghost

# Expose Ghost port
EXPOSE 2368

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD curl -f http://localhost:2368 || exit 1
