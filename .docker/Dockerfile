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

# Create version info file for deployment verification
ARG BUILD_DATE
ARG GIT_COMMIT  
ARG BUILD_VERSION
RUN echo "🚀 Ghost Application Version Info" > /var/lib/ghost/version.txt && \
    echo "=================================" >> /var/lib/ghost/version.txt && \
    echo "Build Date: ${BUILD_DATE:-unknown}" >> /var/lib/ghost/version.txt && \
    echo "Git Commit: ${GIT_COMMIT:-unknown}" >> /var/lib/ghost/version.txt && \
    echo "Build Version: ${BUILD_VERSION:-unknown}" >> /var/lib/ghost/version.txt && \
    echo "Image Built: $(date)" >> /var/lib/ghost/version.txt && \
    echo "=================================" >> /var/lib/ghost/version.txt && \
    # Make version.txt accessible via HTTP by copying to content directory
    mkdir -p /var/lib/ghost/content/data && \
    cp /var/lib/ghost/version.txt /var/lib/ghost/content/data/version.txt && \
    chown -R node:node /var/lib/ghost/version.txt /var/lib/ghost/content/data

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
