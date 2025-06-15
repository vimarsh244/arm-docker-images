#
# STAGE 1: Builder - Mimics the official Mattermost build environment
#
FROM --platform=$BUILDPLATFORM golang:1.24-bookworm AS builder

# Arguments for version and architecture
ARG MATTERMOST_VERSION=9.8.0
ARG TARGETARCH

# Install the correct build dependencies for a Debian-based system
RUN apt-get update && apt-get install -y \
    build-essential \
    git \
    make

# Set the working directory for the source code
WORKDIR /src

# Clone the correct Mattermost server repository
# Using --depth 1 for a faster clone
RUN git clone --depth 1 --branch v${MATTERMOST_VERSION} https://github.com/mattermost/mattermost-server.git .

# IMPORTANT: Change to the /server subdirectory where the main Makefile resides
WORKDIR /src/server

# Build the enterprise edition for the target architecture.
# This is the corrected command, run from the correct directory.
RUN export MM_EDITION=enterprise && GOOS=linux GOARCH=${TARGETARCH} make build-server

# Also build mmctl, which is needed for the healthcheck
RUN GOOS=linux GOARCH=${TARGETARCH} make build-mmctl

# Create a non-root user config file for the distroless image
ARG PUID=2000
ARG PGID=2000
RUN echo "mattermost:x:${PUID}:${PGID}:Mattermost:/mattermost:/bin/false" > /passwd


#
# STAGE 2: Final Image - Minimal and secure using distroless
#
FROM gcr.io/distroless/base-debian12

ARG PUID=2000
ARG PGID=2000

# Set the working directory
WORKDIR /mattermost

# Create necessary directories and copy the application binary from the builder stage
# The output path of the build is /src/bin/
COPY --from=builder --chown=${PUID}:${PGID} /src/bin/mattermost /mattermost/bin/mattermost
COPY --from=builder --chown=${PUID}:${PGID} /src/bin/mmctl /mattermost/bin/mmctl

# Copy the default configuration file
COPY --from=builder --chown=${PUID}:${PGID} /src/server/config/config.json /mattermost/config/config.json

# Copy the user definition we created in the builder stage
COPY --from=builder /passwd /etc/passwd

# It's good practice to create these directories so permissions are set correctly
# We create empty directories in the builder and copy them over
RUN mkdir -p /mattermost_dirs/{data,logs,plugins,client/plugins}
COPY --from=builder --chown=${PUID}:${PGID} /mattermost_dirs/ /mattermost/

# Set environment variables
ENV PATH="/mattermost/bin:${PATH}"
ENV MM_CONFIG="/mattermost/config/config.json"

# Run as the non-privileged user
USER mattermost

# Healthcheck to make sure the container is ready
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD ["/mattermost/bin/mmctl", "system", "status", "--local"]

# Expose the primary port for CapRover
EXPOSE 8065

# Declare volumes that CapRover will manage
VOLUME ["/mattermost/data", "/mattermost/logs", "/mattermost/config", "/mattermost/plugins", "/mattermost/client/plugins"]

# Entrypoint to start the server
ENTRYPOINT ["/mattermost/bin/mattermost"]