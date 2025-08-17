# Method 2: Docker Compose Deployment

<img width="1124" height="623" alt="image" src="https://github.com/user-attachments/assets/ebed4e7c-5e75-432f-9ad2-2f87e8fd9b36" />

## Overview

This deployment method uses one root `docker-compose.yml` to orchestrate multiple services. Each service (frontend, backend) has its own Dockerfile so images can be built independently and then launched together by Compose. This is convenient for local development, simple CI pipelines, and small deployments.

## Files Involved

- **`docker-compose.yml`** (root) — orchestrates services, ports, networks, healthchecks, volumes
- **`frontend/Dockerfile`** — builds the frontend image (multi-stage; build + runner)
- **`backend/Dockerfile`** — builds the Go backend image (multi-stage)

> **Note:** The compose file currently includes a deprecated `version:` field (Compose prints a warning). It still works, but you can remove it.

## How It Works (Contract)

**Inputs:** Source code in `frontend` and `backend` directories  
**Outputs:** Two images named `kubeorchestra/frontend:latest` and `kubeorchestra/backend:latest`, and two running containers  
**Error modes:** Failed image build (missing deps), runtime port conflicts, healthcheck failures  
**Success criteria:** Both containers running and health endpoints returning OK

## Basic Usage Commands

Run these from repository root (the directory with `docker-compose.yml`):

### Start the Application
```bash
docker-compose up -d
```

### Stop the Application
```bash
docker-compose down
```

### Check Application Status
```bash
docker-compose ps
```

## Measured Image Sizes (Local)

I inspected the local images built during validation. Measured sizes (human-friendly `docker images` output):

- **`kubeorchestra/frontend:latest`** — 205MB
- **`kubeorchestra/backend:latest`** — 16MB

## Configuration Files

### docker-compose.yml

```yaml
version: '3.8'

services:
  frontend:
    image: kubeorchestra/frontend:latest
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - BACKEND_URL=http://backend:8080
      - NODE_ENV=production
    depends_on:
      - backend
    restart: unless-stopped
    networks:
      - kubeorchestra
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000"]
      timeout: 10s
      interval: 30s
      retries: 3
      start_period: 40s

  backend:
    image: kubeorchestra/backend:latest
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - PORT=8080
      - GIN_MODE=release
    restart: unless-stopped
    networks:
      - kubeorchestra
    volumes:
      # Mount Docker socket for Docker operations (if needed)
      - /var/run/docker.sock:/var/run/docker.sock
      # Mount Kubernetes config for cluster access
      - ~/.kube:/root/.kube:ro
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:8080/api/health"]
      timeout: 10s
      interval: 30s
      retries: 3
      start_period: 20s

  # Optional: Add a database for production use
  # database:
  #   image: postgres:15-alpine
  #   environment:
  #     POSTGRES_DB: kubeorchestra
  #     POSTGRES_USER: kubeorchestra
  #     POSTGRES_PASSWORD: changeme
  #   volumes:
  #     - postgres_data:/var/lib/postgresql/data
  #   networks:
  #     - kubeorchestra
  #   restart: unless-stopped

networks:
  kubeorchestra:
    driver: bridge

# volumes:
#   postgres_data:
```

### frontend/Dockerfile

```dockerfile
# Build the frontend application
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Install dependencies based on the preferred package manager
COPY package.json package-lock.json* ./
# Install all dependencies (including dev) so the build step has what's required
RUN npm ci

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Build application
RUN npm run build

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# Install wget for the docker-compose healthcheck which uses wget
RUN apk add --no-cache wget

# Copy built application
COPY --from=builder /app/public ./public

# Set the correct permission for prerender cache
RUN mkdir .next
RUN chown nextjs:nodejs .next

# Automatically leverage output traces to reduce image size
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

### backend/Dockerfile

```dockerfile
# Build the Go binary
FROM golang:1.21-alpine AS builder
WORKDIR /src

# Use cached modules when possible
COPY go.mod go.sum* ./
RUN apk add --no-cache git ca-certificates && go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags "-s -w" -o /usr/local/bin/backend ./main.go

# Final image
FROM alpine:3.18
RUN adduser -D -g '' appuser && apk add --no-cache ca-certificates
# Add wget which docker-compose healthchecks use
RUN apk add --no-cache wget
COPY --from=builder /usr/local/bin/backend /usr/local/bin/backend
USER appuser
EXPOSE 8080
ENTRYPOINT ["/usr/local/bin/backend"]
```

## Advantages

- **Simple:** Single `docker-compose.yml` provides one command to bring the whole stack up/down
- **Independent builds:** Each service has its own Dockerfile allowing separate build/deploy cadence
- **Local parity:** Developers can run the entire stack locally with the same images used in CI
- **Readable and easy to adopt** for small teams and POCs

## Disadvantages / Trade-offs

- **Image size:** If Dockerfiles are not optimized (installing dev deps in final image, large base), images can be unnecessarily large (the frontend is ~205MB)
- **Not production-grade orchestration:** Compose is convenient but not a replacement for Kubernetes in larger, distributed systems
- **Security surface:** Base images need regular updates and scanning
- **Resource usage:** Running multiple full images locally consumes CPU/memory
- **Compose healthchecks rely on tools (wget) in image:** Extra package installs inflate final images — better to use minimal health endpoints or baked-in binaries

