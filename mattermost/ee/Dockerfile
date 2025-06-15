# Dockerfile for Mattermost Enterprise Edition

# Builder Stage
FROM --platform=$BUILDPLATFORM golang:1.21-alpine AS builder

ARG MATTERMOST_VERSION=9.8.0
ARG TARGETARCH

# Install build dependencies
RUN apk add --no-cache \
    build-base \
    git \
    make

# Clone the Mattermost repository
RUN git clone --depth 1 --branch v${MATTERMOST_VERSION} https://github.com/mattermost/mattermost-server.git /src

# Build Mattermost
WORKDIR /src
RUN GOOS=linux GOARCH=${TARGETARCH} make build-enterprise

# Final Stage
FROM alpine:3.18

ARG MATTERMOST_VERSION

# Set labels
LABEL org.opencontainers.image.title="Mattermost Enterprise Edition" \
      org.opencontainers.image.description="Mattermost is an open source platform for secure collaboration across the entire software development lifecycle." \
      org.opencontainers.image.version=$MATTERMOST_VERSION \
      org.opencontainers.image.authors="Mattermost, Inc." \
      org.opencontainers.image.url="https://mattermost.com/" \
      org.opencontainers.image.documentation="https://docs.mattermost.com/" \
      org.opencontainers.image.source="https://github.com/mattermost/mattermost-server"

# Create a non-root user
RUN addgroup -g 2000 mattermost && \
    adduser -u 2000 -G mattermost -D mattermost

# Copy the compiled binary from the builder stage
COPY --from=builder /src/bin/mattermost /usr/local/bin/mattermost

# Set up the Mattermost directory structure
RUN mkdir -p /mattermost/data /mattermost/plugins /mattermost/client/plugins && \
    chown -R mattermost:mattermost /mattermost

# Set the user and working directory
USER mattermost
WORKDIR /mattermost

# Expose the Mattermost port
EXPOSE 8065

# Set the entrypoint
ENTRYPOINT ["/usr/local/bin/mattermost"]