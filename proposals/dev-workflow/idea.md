# KubeOrchestra Development Workflow Proposal

## Overview

This proposal outlines an optimal development workflow for the KubeOrchestra project, which consists of a visual Kubernetes orchestrator with separate UI and core repositories. The recommended approach maintains modular architecture while providing an excellent developer experience.

## Recommended Approach: Development Orchestration Repository

The proposed solution maintains UI and core repositories as separate entities (ensuring modularity and CNCF compliance) while introducing a lightweight `kubeorchestra-dev` repository that orchestrates local development workflows.

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
kubeorchestra-dev/
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
git clone https://github.com/yourorg/kubeorchestra-dev
cd kubeorchestra-dev
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
git clone https://github.com/yourorg/kubeorchestra-dev
cd kubeorchestra-dev
./scripts/contribute.sh --fork-ui=yourusername/ui --fork-core=yourusername/core
make dev

# Work in ./ui or ./core directories
# Create feature branches, commit changes, push to forks
# Open pull requests from forks to upstream repositories
```

## Makefile for Enhanced Developer Experience

```makefile
# Makefile
.PHONY: setup dev stop clean update test

setup:
	@echo "Setting up KubeOrchestra development environment..."
	@./scripts/setup.sh

dev:
	@echo "Starting development environment..."
	docker-compose up -d
	@echo "✓ Core API: http://localhost:3000"
	@echo "✓ UI: http://localhost:3001"
	@echo "✓ PostgreSQL: localhost:5432"

dev-ui-only:
	@echo "Starting UI with latest core image..."
	docker run -d -p 5432:5432 postgres:14
	docker run -d -p 3000:3000 ghcr.io/yourorg/kubeorchestra-core:latest
	cd ui && npm run dev

dev-core-only:
	@echo "Starting Core with latest UI image..."
	docker-compose up -d postgres core
	@echo "UI available at: https://kubeorchestra.dev"

stop:
	docker-compose down

clean:
	docker-compose down -v
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
```

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
kubeorchestra dev ui    # Start only UI (uses remote backend)
kubeorchestra dev core  # Start only core (uses hosted UI)
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