# Multi-stage build for Mattermost Enterprise Edition on ARM64 with Yarn
# Addresses peer dependency issues and uses Yarn for package management

# First stage - Build environment
FROM ubuntu:noble-20250415.1 AS builder

# Setting bash as shell with pipefail option
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Build Arguments
ARG PUID=2000
ARG PGID=2000
ARG MATTERMOST_VERSION=latest
ARG BUILD_ENTERPRISE=1
ARG TARGETARCH=arm64

# Environment variables for cross-compilation
ENV GOOS=linux
ENV GOARCH=arm64
ENV CGO_ENABLED=0

# Install system dependencies and build tools
RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    wget \
    git \
    build-essential \
    python3 \
    python3-pip \
    media-types \
    mailcap \
    unrtf \
    wv \
    poppler-utils \
    tidy \
    tzdata \
    && rm -rf /var/lib/apt/lists/*

# Install Node.js for ARM64
RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash - \
    && apt-get install -y nodejs

# Install Yarn globally
RUN npm install -g yarn

# Install Go for ARM64
RUN wget -O go.tar.gz "https://golang.org/dl/go1.21.5.linux-arm64.tar.gz" \
    && tar -C /usr/local -xzf go.tar.gz \
    && rm go.tar.gz

ENV PATH="/usr/local/go/bin:${PATH}"
ENV GOROOT="/usr/local/go"

# Create mattermost user and directories
RUN groupadd --gid ${PGID} mattermost \
    && useradd --uid ${PUID} --gid ${PGID} --comment "" --home-dir /mattermost mattermost \
    && mkdir -p /mattermost/data /mattermost/plugins /mattermost/client/plugins /build

# Clone Mattermost source code
WORKDIR /build
RUN git clone --depth 1 --branch master https://github.com/mattermost/mattermost.git

# Build Mattermost webapp with Yarn and peer dependency handling
WORKDIR /build/mattermost/webapp

# Create .yarnrc.yml for Yarn modern or use legacy approach
RUN echo 'nodeLinker: node-modules' > .yarnrc.yml

# Clear any existing node_modules and package-lock files
RUN rm -rf node_modules package-lock.json yarn.lock || true

# Install dependencies with Yarn, handling peer dependency conflicts
# RUN yarn cache clean \
#     && yarn install --ignore-engines \
#     && yarn build

# Alternative approach with legacy peer deps handling if above fails
RUN yarn install --legacy-peer-deps || yarn install --ignore-engines \
    && yarn build

# Build Mattermost server for ARM64
WORKDIR /build/mattermost/server
RUN make setup-go-work \
    && make build-linux \
    BUILD_NUMBER="docker-arm64" \
    BUILD_ENTERPRISE=${BUILD_ENTERPRISE} \
    GOARCH=arm64 \
    GOOS=linux

# Create distribution package
RUN make package \
    BUILD_NUMBER="docker-arm64" \
    BUILD_ENTERPRISE=${BUILD_ENTERPRISE} \
    GOARCH=arm64 \
    GOOS=linux

# Extract built package
RUN if [ "${BUILD_ENTERPRISE}" = "1" ]; then \
        tar -xzf dist/mattermost-*-linux-arm64.tar.gz -C /; \
    else \
        tar -xzf dist/mattermost-team-*-linux-arm64.tar.gz -C /; \
    fi

# Set proper ownership
RUN chown -R mattermost:mattermost /mattermost

# Final stage using distroless for minimal attack surface
FROM gcr.io/distroless/base-debian12:latest

# Environment variables
ENV PATH="/mattermost/bin:${PATH}"
ENV MM_SERVICESETTINGS_ENABLELOCALMODE="true"

# Copy metadata files needed by runtime
COPY --from=builder /etc/mime.types /etc/

# Copy document processing utilities and support files
COPY --from=builder /usr/bin/pdftotext /usr/bin/pdftotext
COPY --from=builder /usr/bin/wvText /usr/bin/wvText
COPY --from=builder /usr/bin/wvWare /usr/bin/wvWare
COPY --from=builder /usr/bin/unrtf /usr/bin/unrtf
COPY --from=builder /usr/bin/tidy /usr/bin/tidy
COPY --from=builder /usr/share/wv /usr/share/wv

# Copy necessary libraries for document processing (ARM64 paths)
COPY --from=builder /usr/lib/aarch64-linux-gnu/libpoppler.so* /usr/lib/
COPY --from=builder /usr/lib/aarch64-linux-gnu/libfreetype.so* /usr/lib/
COPY --from=builder /usr/lib/aarch64-linux-gnu/libpng16.so* /usr/lib/
COPY --from=builder /usr/lib/aarch64-linux-gnu/libwv.so* /usr/lib/
COPY --from=builder /usr/lib/aarch64-linux-gnu/libtidy.so* /usr/lib/
COPY --from=builder /usr/lib/aarch64-linux-gnu/libfontconfig.so* /usr/lib/

# Copy built Mattermost application
COPY --from=builder --chown=2000:2000 /mattermost /mattermost

# Copy passwd file for user resolution
COPY --from=builder /etc/passwd /etc/passwd

# Switch to non-root user
USER mattermost

# Health check using mmctl for distroless compatibility
HEALTHCHECK --interval=30s --timeout=10s \
    CMD ["/mattermost/bin/mmctl", "system", "status", "--local"]

# Set working directory
WORKDIR /mattermost

# Default command
CMD ["/mattermost/bin/mattermost"]

# Expose ports
EXPOSE 8065 8067 8074 8075

# Declare volumes
VOLUME ["/mattermost/data", "/mattermost/logs", "/mattermost/config", "/mattermost/plugins", "/mattermost/client/plugins"]
