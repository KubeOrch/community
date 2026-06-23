# KubeOrch Roadmap

> **Last updated:** April 2026

This is the unified roadmap for the KubeOrch project. It covers all repositories (core, ui, cli, community, docs) and is reviewed quarterly by maintainers.

## Vision

KubeOrch is the visual workflow orchestrator for Kubernetes. Drag-and-drop container orchestration, intelligent automation, and deep ecosystem integration — making Kubernetes accessible to developers of all skill levels.

---

## Phase 1: Core Platform Completion

**Status: In Progress | Target: Q2 2026**

**Theme: Fill the gaps in K8s resource coverage, service discovery, templates, and CLI connectivity.**

### Kubernetes Resources

- [ ] Complete remaining resource handlers: CRDs, PodDisruptionBudget, LimitRange, ResourceQuota
- [ ] Improve existing handlers: StatefulSet lifecycle, DaemonSet rolling updates, Job/CronJob status tracking
- [ ] Network policy visualization on canvas (show allowed/denied traffic flows)
- [ ] Storage class management and PVC lifecycle improvements

### Service Discovery (Bidirectional)

- [ ] **Inbound discovery**: Services auto-register endpoints; consumers discover via K8s DNS with visual indicators on canvas
- [ ] **Outbound discovery**: Canvas nodes show which services they depend on, auto-resolve DNS names and ports
- [ ] Service graph visualization in UI — interactive topology map showing all service-to-service connections
- [ ] Health-aware discovery: connection lines reflect real-time health status (healthy/degraded/down)
- [ ] DNS and endpoint debugging tools in the diagnostics panel

### CLI Backend Integration

The CLI currently only orchestrates Docker Compose locally. It needs to become a first-class API client.

- [ ] Add API client module to CLI (auth via JWT token, configurable backend URL)
- [ ] `orchcli workflow list` / `orchcli workflow run <id>` / `orchcli workflow status <id>`
- [ ] `orchcli logs <pod> [-f] [--tail N]` — stream pod logs from backend (SSE)
- [ ] `orchcli cluster list` / `orchcli cluster info <id>` / `orchcli cluster switch <name>`
- [ ] `orchcli template list` / `orchcli template apply <name>`
- [ ] `orchcli resource list [--namespace] [--type]` — query cluster resources
- [ ] `orchcli status` should show both local Docker state AND remote cluster state

### Network Traffic Visualization

- [ ] Display real-time request rates on connection lines (requires metrics collection)
- [ ] Traffic flow animation on canvas edges
- [ ] Ingress traffic breakdown by path/host

### Project Quality (CNCF Readiness)

- [ ] Increase test coverage (core 40%+, UI 30%+, CLI all platforms)
- [ ] Submit CNCF Sandbox application
- [ ] OpenSSF Best Practices passing badge
- [ ] DCO enforcement on all repositories
- [ ] Generate OpenAPI/Swagger specification
- [ ] Complete API reference documentation
- [ ] Fix CORS configuration (restrict origins)
- [ ] Production deployment guide in docs

---

## Phase 2: CI Integration & Image Building

**Status: Not Started | Target: Q3 2026**

**Theme: Integrate with existing CI providers instead of building a custom CD engine. Meet users where they already are.**

### CI Plugins (GitHub Actions, GitLab CI, CircleCI)

Provide reusable CI steps — like how npm provides publish actions — so users plug KubeOrch into their existing pipelines.

- [ ] **GitHub Action** (`kubeorch/build-and-deploy`): Build image, push to registry, optionally trigger deployment via KubeOrch API
- [ ] **GitLab CI template** (`.gitlab-ci.yml` include): Same flow as GitHub Action, packaged as a reusable GitLab CI template
- [ ] **CircleCI Orb** (`kubeorch/deploy`): Reusable orb for CircleCI pipelines
- [ ] Each plugin supports: build-only mode, build+push mode, build+push+deploy mode
- [ ] Publish plugins to their respective marketplaces (GitHub Marketplace, GitLab CI template registry, CircleCI Orb Registry)
- [ ] Documentation and quickstart guides per provider

### Webhook-Based Build Triggers (Coolify-Style)

For the "just push code and it deploys" workflow without needing to configure CI.

- [ ] GitHub webhook receiver in core (`POST /v1/webhooks/github`) — on push, trigger image build
- [ ] GitLab webhook receiver (`POST /v1/webhooks/gitlab`)
- [ ] Generic webhook receiver (`POST /v1/webhooks/generic`) — for Bitbucket, Gitea, etc.
- [ ] Webhook configuration UI: connect a repo, choose branch, auto-build on push
- [ ] Build queue with concurrency limits
- [ ] Real-time build status streaming (already partially exists via SSE)

### Registry Integration

- [ ] Improve existing GHCR, ECR, Docker Hub support
- [ ] Add GCR (Google), ACR (Azure) support
- [ ] Private registry authentication management in UI
- [ ] Image tag strategy configuration (commit SHA, branch name, semver)
- [ ] Image vulnerability scanning integration (Trivy or similar)

---

## Phase 3: Templates & Marketplace

**Status: Not Started | Target: Q3-Q4 2026**

**Theme: Give users ready-made building blocks and let teams share their own.**

### Curated Template Gallery

Popular stacks available out of the box:

- [ ] **Web stacks**: MERN, MEAN, Django + Postgres, Rails + Redis + Postgres, Laravel + MySQL
- [ ] **Microservice starters**: Go gRPC service, Node.js Express API, Python FastAPI
- [ ] **Data pipelines**: Kafka + consumer + Postgres, RabbitMQ + worker
- [ ] **Monitoring stacks**: Prometheus + Grafana, ELK/EFK
- [ ] Each template includes: sensible defaults, resource limits, health checks, connection wiring

### Org-Scoped Internal Templates

- [ ] Template publishing API: `POST /v1/templates` with org scope
- [ ] Template versioning (semver)
- [ ] Template discovery within org (search, tags, categories)
- [ ] Template access control (org-only, public)
- [ ] `orchcli template publish` / `orchcli template pull <org/name>`

### Workflow Presets

- [ ] Pre-built architecture patterns: 3-tier web app, event-driven microservices, batch processing
- [ ] "Start from preset" flow in UI — choose a pattern, customize, deploy
- [ ] Preset import/export as JSON
- [ ] Community-contributed presets

### Template Composition

- [ ] Combine multiple templates into a single workflow (e.g., "MERN stack" + "monitoring" + "ingress")
- [ ] Dependency resolution between composed templates
- [ ] Override and merge configuration across templates

---

## Phase 4: CLI & Developer Experience

**Status: Not Started | Target: Q4 2026**

**Theme: Make the CLI a power-user's primary interface and a bridge for AI tools.**

### CLI as MCP Server

- [ ] Implement MCP (Model Context Protocol) server mode in CLI
- [ ] Expose backend API operations as MCP tools: deploy workflow, get logs, list resources, check status
- [ ] Any AI tool (Claude, Cursor, etc.) can connect to KubeOrch through the CLI
- [ ] `orchcli mcp serve` — starts MCP server on stdio or SSE

### CLI Improvements

- [ ] Shell completions: bash, zsh, fish (via Cobra's built-in generators)
- [ ] `orchcli update` — self-update command
- [ ] `orchcli config set/get` — persistent configuration (backend URL, default cluster, auth token)
- [ ] Interactive mode for complex operations (cluster setup, template configuration)
- [ ] Output formatting: `--output json|yaml|table`

### Distribution

- [ ] Homebrew tap for macOS/Linux
- [ ] APT/YUM repository for Linux
- [ ] Native install scripts for Windows (PowerShell + CMD)
- [ ] npm global package (`npm install -g @kubeorch/cli`)
- [ ] Docker image with CLI pre-installed

---

## Phase 5: AI Integration

**Status: Not Started | Target: Q4 2026 - Q1 2027**

**Theme: AI that understands KubeOrch natively — as a skill and through MCP.**

### Claude Skill for KubeOrch

- [ ] Encode full knowledge of KubeOrch: architecture, API surface, workflow model, template system, connection semantics
- [ ] Skill understands how components connect (e.g., "a Service needs selector labels matching a Deployment")
- [ ] Can generate valid workflow JSON from natural language descriptions
- [ ] Can explain existing workflows and suggest improvements
- [ ] Published as a reusable skill any AI tool can load

### MCP Server (Backend Bridge)

- [ ] CLI-based MCP server wraps the core API (see Phase 4)
- [ ] Tools exposed: `deploy_workflow`, `get_logs`, `list_resources`, `get_cluster_status`, `apply_template`, `build_image`
- [ ] Resources exposed: workflow definitions, template catalog, cluster info
- [ ] Enables AI agents to operate KubeOrch autonomously within guardrails

### AI-Assisted Features in UI

- [ ] "Describe what you're building" — generates workflow draft on canvas
- [ ] "Optimize this" — analyzes resource usage, suggests right-sizing, adds health checks
- [ ] "Explain this workflow" — generates human-readable architecture documentation
- [ ] Powered by core API endpoints (`/v1/ai/design`, `/v1/ai/optimize`, `/v1/ai/explain`)

---

## Phase 6: Advanced Platform

**Status: Not Started | Target: 2027+**

**Theme: Enterprise features, scale, and deep ecosystem integration.**

### Canvas & Workflow

- [ ] Module rearrangement: drag to reorder deployment topology, auto-update all connections and dependencies
- [ ] Collaborative editing (multiple users on same canvas)
- [ ] Workflow diff and version comparison improvements
- [ ] Import from existing cluster (reverse-engineer running workloads into canvas)

### Autoscaling & Traffic

- [ ] Network-traffic-based autoscaling: HPA policies driven by ingress request rates / service mesh metrics
- [ ] Traffic splitting visualization (canary, blue-green) on canvas
- [ ] Service mesh integration (Istio, Linkerd) — traffic management UI

### Multi-Tenancy & RBAC

- [ ] Namespace-isolated tenants with resource quotas
- [ ] Fine-grained RBAC: per-workflow, per-cluster, per-namespace permissions
- [ ] Team management UI
- [ ] SSO/SAML integration

### Operations

- [ ] Cost analysis: per-namespace, per-workflow cost breakdown with cloud provider pricing
- [ ] Audit logging: all API actions tracked with user, timestamp, and diff
- [ ] Backup and restore (Velero integration)
- [ ] Multi-cloud deployment support

### GitOps Compatibility

- [ ] Export workflows as Git-tracked manifests (YAML directory structure)
- [ ] ArgoCD Application generation from KubeOrch workflows
- [ ] Flux Kustomization generation from KubeOrch workflows
- [ ] Drift detection: compare canvas state vs. live cluster vs. Git

---

## How to Influence the Roadmap

1. Open a discussion in the [community repo](https://github.com/KubeOrch/community/discussions)
2. Propose an RFC in the [proposals directory](proposals/)
3. Comment on existing roadmap items in GitHub Issues

Maintainers review roadmap priorities at the beginning of each quarter.
