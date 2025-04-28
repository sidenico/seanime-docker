# Stage 1: Node.js Builder
FROM node:latest AS node-builder

COPY ./seanime-web /tmp/build

WORKDIR /tmp/build

RUN npm ci
RUN npm run build

# Stage 2: Go Builder
FROM --platform=$BUILDPLATFORM golang:latest AS go-builder

ARG TARGETOS
ARG TARGETARCH

COPY . /tmp/build
COPY --from=node-builder /tmp/build/out /tmp/build/web

WORKDIR /tmp/build

RUN CGO_ENABLED=0 GOOS=${TARGETOS} GOARCH=${TARGETARCH} go build -o seanime -trimpath -ldflags="-s -w"

# Stage 3: Final Stage (Alpine Linux with qBittorrent, and Supervisord)
FROM alpine:latest

# Install necessary packages: qBittorrent-nox, and Supervisord for process management
RUN apk add --no-cache \
    qbittorrent-nox \
    supervisor \
    ffmpeg

# Copy the Go-built application from the previous stage
COPY --from=go-builder /tmp/build/seanime/seanime /app/seanime

# Create directories for Seanime, qBittorrent and Supervisord
RUN mkdir -p /app /data /var/log/supervisor

# Work in /app directory
WORKDIR /app

# Add QBittorrent configuration file
#
# The default credentials are:
# Username: admin
# Password: adminadmin
COPY ./.docker/config /config

# Define volumes for persistent data
VOLUME /data
VOLUME /config

# Expose ports: 8080 for Seanime, 8081 for qBittorrent Web UI
EXPOSE 8080 8081

# Start both services via Supervisord
CMD ["/usr/bin/supervisord", "-c", "/config/Supervisord/supervisord.conf"]
