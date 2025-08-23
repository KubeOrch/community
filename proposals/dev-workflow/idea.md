# KubeOrchestra Development Workflow Proposal

## Overview

This proposal outlines an optimal development workflow for the KubeOrchestra project, which consists of a visual Kubernetes orchestrator with separate UI and core repositories. The recommended approach maintains modular architecture while providing an excellent developer experience.

## Recommended Approach: Development Orchestration Repository

The proposed solution maintains UI and core repositories as separate entities (ensuring modularity and CNCF compliance) while introducing a lightweight `kubeorchestra-infra` repository that orchestrates local development workflows.

### Docker Compose Configuration

```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:14
    environment:
      POSTGRES_DB: kubeorchestra
      POSTGRES_PASSWORD: dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  core:
    build:
      context: ./core
      dockerfile: Dockerfile.dev
    volumes:
      - ./core:/app
      - /app/vendor  # Preserve Go modules
    ports:
      - "3000:3000"
    environment:
      DB_HOST: postgres
      DB_NAME: kubeorchestra
    depends_on:
      - postgres
    command: air  # Hot reload for Go

  ui:
    build:
      context: ./ui
      dockerfile: Dockerfile.dev
    volumes:
      - ./ui:/app
      - /app/node_modules  # Preserve node_modules
    ports:
      - "3001:3000"
    environment:
      NEXT_PUBLIC_API_URL: http://localhost:3000
    depends_on:
      - core
    command: npm run dev

volumes:
  postgres_data:
```

### Project Structure

```
kubeorchestra-infra/
├── docker-compose.yml
├── Makefile
├── scripts/
│   ├── setup.sh        # Clone repositories, install dependencies
│   ├── update.sh       # Pull latest changes from both repositories
│   └── contribute.sh   # Helper for fork-based contribution workflow
├── .env.example
└── README.md

# Repositories are cloned as subdirectories:
├── core/               # Cloned backend repository
└── ui/                 # Cloned frontend repository
```

## Developer Workflows

### For Organization Members

#### Initial Setup
```bash
git clone https://github.com/kubeorchestra/infra
cd infra
make setup  # Clones UI/core repositories, installs dependencies
make dev    # Starts all services with docker-compose
```

#### Daily Development Workflow
```bash
make update  # Pull latest changes from both repositories
make dev     # Start development environment
# Work in ./ui or ./core directories
# Commit and push directly to respective repositories
```

### For External Contributors

#### Fork-Based Contribution Workflow
```bash
# Fork UI and/or core repositories first, then:
git clone https://github.com/kubeorchestra/infra
cd infra
./scripts/contribute.sh --fork-ui=yourusername/ui --fork-core=yourusername/core
make dev

# Work in ./ui or ./core directories
# Create feature branches, commit changes, push to forks
# Open pull requests from forks to upstream repositories
```

## Development Scenarios

### Frontend Developer Workflow

When a frontend developer wants to contribute:

1. **Clone the infra repository**:
   ```bash
   git clone https://github.com/kubeorchestra/infra
   cd infra
   ```

2. **Setup frontend locally**:
   ```bash
   make setup-ui  # Clones UI repo, installs dependencies
   ```

3. **Run backend as published image**:
   ```bash
   make dev-ui-only  # Starts PostgreSQL + published core image
   ```

4. **Run frontend on host machine**:
   ```bash
   cd ui && npm run dev  # Frontend runs on localhost:3001
   ```

**Result**: Frontend runs locally with hot reload, backend runs as published image on port 3000, database on port 5432.

### Backend Developer Workflow

When a backend developer wants to contribute:

1. **Clone the infra repository**:
   ```bash
   git clone https://github.com/kubeorchestra/infra
   cd infra
   ```

2. **Setup backend locally**:
   ```bash
   make setup-core  # Clones core repo, installs dependencies
   ```

3. **Run frontend as published image**:
   ```bash
   make dev-core-only  # Starts PostgreSQL + published UI image
   ```

4. **Run backend on host machine**:
   ```bash
   cd core && air  # Backend runs on localhost:3000 with hot reload
   ```

**Result**: Backend runs locally with hot reload, frontend runs as published image on port 3001, database on port 5432.

### Full Development Workflow

When working on both components:

1. **Clone the infra repository**:
   ```bash
   git clone https://github.com/kubeorchestra/infra
   cd infra
   ```

2. **Setup both repositories**:
   ```bash
   make setup  # Clones both repos, installs dependencies
   ```

3. **Run everything locally**:
   ```bash
   make dev  # Starts PostgreSQL + local core + local UI
   ```

**Result**: Both frontend and backend run locally with hot reload, database on port 5432.

## Makefile for Enhanced Developer Experience

```makefile
# Makefile
.PHONY: setup setup-ui setup-core dev dev-ui-only dev-core-only stop clean update test

setup:
	@echo "Setting up KubeOrchestra development environment..."
	@./scripts/setup.sh

setup-ui:
	@echo "Setting up UI development environment..."
	git clone https://github.com/kubeorchestra/ui
	cd ui && npm install

setup-core:
	@echo "Setting up Core development environment..."
	git clone https://github.com/kubeorchestra/core
	cd core && go mod download

dev:
	@echo "Starting full development environment..."
	docker-compose up -d postgres
	@echo "✓ PostgreSQL: localhost:5432"
	@echo "Starting Core (local development)..."
	cd core && air &
	@echo "Starting UI (local development)..."
	cd ui && npm run dev &
	@echo "✓ Core API: http://localhost:3000"
	@echo "✓ UI: http://localhost:3001"

dev-ui-only:
	@echo "Starting UI development with published core image..."
	docker run -d --name kubeorchestra-postgres -p 5432:5432 postgres:14
	docker run -d --name kubeorchestra-core -p 3000:3000 ghcr.io/kubeorchestra/core:latest
	@echo "✓ Core API (published image): http://localhost:3000"
	@echo "✓ PostgreSQL: localhost:5432"
	@echo "Now run: cd ui && npm run dev"

dev-core-only:
	@echo "Starting Core development with published UI image..."
	docker run -d --name kubeorchestra-postgres -p 5432:5432 postgres:14
	docker run -d --name kubeorchestra-ui -p 3001:3000 ghcr.io/kubeorchestra/ui:latest
	@echo "✓ UI (published image): http://localhost:3001"
	@echo "✓ PostgreSQL: localhost:5432"
	@echo "Now run: cd core && air"

stop:
	docker-compose down
	docker stop kubeorchestra-core kubeorchestra-postgres kubeorchestra-ui || true
	docker rm kubeorchestra-core kubeorchestra-postgres kubeorchestra-ui || true

clean:
	docker-compose down -v
	docker stop kubeorchestra-core kubeorchestra-postgres kubeorchestra-ui || true
	docker rm kubeorchestra-core kubeorchestra-postgres kubeorchestra-ui || true
	rm -rf ui/node_modules core/vendor

update:
	@echo "Updating repositories..."
	cd ui && git pull origin main
	cd core && git pull origin main

test:
	cd ui && npm test
	cd core && go test ./...

logs:
	docker-compose logs -f

logs-core:
	docker-compose logs -f core

logs-ui:
	docker-compose logs -f ui

# Component management commands
stop-ui-only:
	docker stop kubeorchestra-core kubeorchestra-postgres || true
	docker rm kubeorchestra-core kubeorchestra-postgres || true

stop-core-only:
	docker stop kubeorchestra-postgres kubeorchestra-ui || true
	docker rm kubeorchestra-postgres kubeorchestra-ui || true

# Image management
pull-latest:
	docker pull ghcr.io/kubeorchestra/core:latest
	docker pull ghcr.io/kubeorchestra/ui:latest
	@echo "✓ Latest core and UI images pulled"

# Environment validation
validate-env:
	@echo "Validating development environment..."
	@docker ps | grep -q kubeorchestra-core || echo "⚠ Core container not running"
	@docker ps | grep -q kubeorchestra-postgres || echo "⚠ PostgreSQL container not running"
	@docker ps | grep -q kubeorchestra-ui || echo "⚠ UI container not running"
	@curl -s http://localhost:3000/health > /dev/null && echo "✓ Core API responding" || echo "⚠ Core API not responding"
	@curl -s http://localhost:3001 > /dev/null && echo "✓ UI responding" || echo "⚠ UI not responding"
```

## Component Isolation and Published Image Integration

A key feature of this development workflow is the ability to run individual components locally while using the latest published images for the other components. This ensures that:

- **Consistency**: Development matches production behavior exactly
- **Network Transparency**: Components connect via the same ports regardless of whether they're running locally or in containers
- **Isolation**: Developers can focus on one component while using stable versions of others
- **Production Parity**: Reduces "works on my machine" issues by using the same images as production

### Published Image Usage Strategy

When running individual components:
- **UI-only mode**: Frontend runs locally on host, backend uses published `ghcr.io/kubeorchestra/core:latest` image
- **Core-only mode**: Backend runs locally on host, frontend uses published `ghcr.io/kubeorchestra/ui:latest` image
- **Full development mode**: Both components run locally with hot reload

This ensures that all three components (database, core, and UI) are consistently available as published images, providing complete offline development capability and production parity.

## Alternative: Lightweight Development CLI

As an alternative approach, we propose creating a simple Go/Node CLI tool that developers can install globally:

### Installation
```bash
npm install -g @kubeorchestra/dev-cli
```

### Available Commands
```bash
kubeorchestra init      # Clone repositories, setup development environment
kubeorchestra dev       # Start all services
kubeorchestra dev ui    # Start only UI (uses published backend)
kubeorchestra dev core  # Start only core (uses published frontend)
kubeorchestra test      # Run all tests
kubeorchestra contribute # Setup for external contribution workflow
```

## Benefits of This Approach

1. **Repository Separation**: Maintains clean boundaries, independent versioning, and CNCF compliance
2. **Simplified Developer Experience**: Single command to run the entire development environment
3. **Flexible Workflows**: Developers can work on UI or core components independently
4. **Fork-Friendly**: Automated scripts handle fork setup for external contributors
5. **Docker-Based**: Ensures consistent development environment across all team members
6. **Hot Reload Support**: Both UI and core components support live reloading for rapid development

## CNCF Submission Considerations

This architecture is particularly well-suited for CNCF submission because:

- **Clear Separation of Concerns**: Distinct UI, core, and infrastructure components
- **Independent Evolution**: Each component can evolve independently without affecting others
- **Cloud-Native Patterns**: Follows established cloud-native development practices
- **Contributor-Friendly**: Easy for new contributors to understand project scope and boundaries
- **Proven Success**: Similar to successful CNCF projects (Prometheus, Flux, etc.)

The development orchestration repository serves as the "glue" that connects components without creating tight coupling between the actual codebases.