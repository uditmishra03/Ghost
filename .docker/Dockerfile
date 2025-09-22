# --------------------
# Base Ghost Image
# --------------------
FROM ghost:5-alpine

# Set working directory
WORKDIR /var/lib/ghost

# Copy in production configuration
COPY config.production.json /var/lib/ghost/config.production.json

# If you have custom themes or content, copy them
# COPY content /var/lib/ghost/content

# Ensure correct permissions
RUN chown -R node:node /var/lib/ghost

# Run as non-root for security
USER node

# Expose Ghost's port
EXPOSE 2368

# Entrypoint from Ghost image
CMD ["node", "current/index.js"]
