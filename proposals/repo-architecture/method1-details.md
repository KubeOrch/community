# Method 1: Production Monolithic Dockerfile (Dockerfile.monolith)

## Overview

This method builds a single monolithic runtime image that contains both the Go backend and the Next.js frontend. A single multi-stage `Dockerfile.monolith` compiles/builds each service in its own stage and assembles a single, runnable final image. This is our **production deployment approach** for end users, providing a simple single-image experience similar to Prometheus and Thanos.

## Purpose

**Production Deployment**: This approach is designed for end users who want a simple, single-image deployment experience. It provides the same user experience as popular tools like Prometheus and Thanos, where users can simply run one container and have the entire application available.

## Files involved

- `Dockerfile.monolith` — multi-stage Dockerfile that builds frontend and backend and produces one runtime image
- `docker-entrypoint.sh` — simple supervisor/entrypoint script (expected by the monolith runtime)
- source tree: `frontend/` and `backend/` directories used as build contexts
- optional: `.dockerignore` to reduce build context size

## How it works (contract)

Inputs: repository containing `frontend/` and `backend/` source directories

Outputs: a single image `kubeorchestra/kubeorchestra:latest` which exposes both backend (8080) and frontend (3000) ports

Error modes: build failures (missing deps), port conflicts when running container, missing runtime dependencies for frontend (if not copied or installed)

Success criteria: container runs, backend health endpoint responds OK, frontend serves requests

## Basic usage commands

Run the monolith (maps both ports 8080 and 3000):

```bash
docker run -d --name kubeorchestra -p 8080:8080 -p 3000:3000 kubeorchestra/kubeorchestra:latest
```

Stop and remove:

```bash
docker rm -f kubeorchestra
```

## Measured image size (local)

- `kubeorchestra/kubeorchestra:latest` — 206MB (measured locally via `docker images` after build)

How this was measured: `docker images --format 'table {{.Repository}}\t{{.Tag}}\t{{.Size}}' | grep kubeorchestra/kubeorchestra`

## Configuration (Dockerfile.monolith)

The monolith used in this project (abridged) — full file is in project root as `Dockerfile.monolith`:

```dockerfile
# Multi-stage Dockerfile to build Go backend and Next.js frontend
# Produces a single, small runtime image that runs both services.

### Build stage: frontend
FROM node:20-alpine AS frontend-build
WORKDIR /app/frontend
COPY frontend/package*.json ./
RUN npm ci --silent
COPY frontend/ .
RUN npm run build --silent

### Build stage: backend
FROM golang:1.21-alpine AS backend-build
WORKDIR /app/backend
RUN apk add --no-cache git ca-certificates build-base
COPY backend/go.mod backend/go.sum ./
RUN go mod download
COPY backend/ .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags='-s -w' -o /usr/local/bin/backend ./

### Final runtime image
FROM alpine:3.18 AS runtime
RUN apk add --no-cache ca-certificates tini
WORKDIR /app

# Copy built backend binary
COPY --from=backend-build /usr/local/bin/backend /usr/local/bin/backend

# Copy Next.js build output and public assets
COPY --from=frontend-build /app/frontend/.next /app/frontend/.next
COPY --from=frontend-build /app/frontend/public /app/frontend/public
COPY --from=frontend-build /app/frontend/package.json /app/frontend/package.json
COPY --from=frontend-build /app/frontend/node_modules /app/frontend/node_modules

# Install Node to run the Next start in production
RUN apk add --no-cache nodejs npm

# Add a small entrypoint supervisor script
COPY docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
RUN chmod +x /usr/local/bin/docker-entrypoint.sh

ENV PORT=8080
EXPOSE 8080 3000

ENTRYPOINT ["/sbin/tini", "--", "/usr/local/bin/docker-entrypoint.sh"]
```

> Note: the final image installs `nodejs` and `npm` but the Dockerfile must ensure frontend runtime dependencies (Next, runtime node_modules or standalone output) are present in the final image — otherwise the frontend may return 500.

## Entrypoint (`docker-entrypoint.sh`)

The repository includes a small supervisor-style entrypoint script (`docker-entrypoint.sh`) used by the monolith runtime. Its behavior is:

- Starts the backend binary in background and records its PID.
- Changes to `/app/frontend` and ensures `node_modules` exist; if missing it runs `npm ci --production --silent` as a fallback.
- Starts the frontend using `npx next start -p 3000` in background and records its PID.
- Uses `wait -n` to wait until one process exits; then it attempts to gracefully terminate the other and exits with the original process' code.

### Entrypoint Script Content

```bash
#!/bin/sh
set -e

# Start backend in background
echo "Starting backend on port ${PORT:-8080}..."
PORT=${PORT:-8080}
export PORT
/usr/local/bin/backend &
BACKEND_PID=$!

cd /app/frontend || exit 0

echo "Starting frontend (next start) on port 3000..."
# Ensure node_modules exist minimally (install prod deps if needed)
if [ ! -d node_modules ]; then
  npm ci --production --silent || true
fi

# Start Next in background
npx next start -p 3000 &
FRONTEND_PID=$!

wait -n $BACKEND_PID $FRONTEND_PID

EXIT_CODE=0
if ! kill -0 $BACKEND_PID 2>/dev/null; then
  wait $BACKEND_PID || EXIT_CODE=$?
fi
if ! kill -0 $FRONTEND_PID 2>/dev/null; then
  wait $FRONTEND_PID || EXIT_CODE=$?
fi

echo "One of the processes exited, shutting down..."
kill -TERM $BACKEND_PID 2>/dev/null || true
kill -TERM $FRONTEND_PID 2>/dev/null || true
exit $EXIT_CODE
```

## Advantages

- **Single deployable artifact**: Simple to distribute or run when one image is preferred
- **User-friendly**: Similar experience to popular tools like Prometheus and Thanos
- **Easier to run in constrained environments**: Where running a single container is simpler than multiple
- **Avoids networking/compose complexity**: For basic deployments and demos
- **Production-ready**: Optimized for end-user deployment scenarios

## Disadvantages / Trade-offs

- **Larger final image**: Bundling two services increases size and surface area (monolith image measured ~206MB)
- **Less flexible deployment**: Cannot scale frontend and backend independently
- **Potential runtime dependency mistakes**: Must ensure frontend runtime deps are present in final image
- **Not suited for development**: Developers need separate images for faster iteration cycles

## Integration with CLI Tool

This monolithic approach works seamlessly with the CLI tool for:

- **Initialization**: Setting up the container with proper volumes and networking
- **Database Configuration**: Managing external database connections
- **Upgrade Operations**: Seamless version updates without data loss
- **Service Management**: Start, stop, and status operations

## Production Deployment Example

```bash
# Simple deployment with CLI
kubeorchestra init

# Manual deployment
docker run -d --name kubeorchestra \
  -p 8080:8080 -p 3000:3000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v ~/.kube:/root/.kube:ro \
  -v kubeorchestra_data:/app/data \
  kubeorchestra/kubeorchestra:latest

# Database configuration
kubeorchestra configure-db --type postgres --host localhost --port 5432

# Upgrade to new version
kubeorchestra upgrade --version v1.2.0
```
