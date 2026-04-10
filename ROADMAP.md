# KubeOrch Roadmap

This document outlines the project's direction and planned milestones. The roadmap is reviewed and updated quarterly by the maintainers.

> **Last updated:** April 2026

## vorchCD, GitOps & AI Assistant Track

**Theme: Native CD + AI-powered workflows**

This track introduces:
- **vorchCD** — a native continuous delivery engine for KubeOrch
- **GitOps integration** — Git as source of truth for visual workflows
- **AI Assistant** — AI-driven design, optimization, and documentation

### Q2 2026 – Foundations
- [ ] Draft vorchCD + AI Assistant RFC in `community/proposals/` (scope, architecture, APIs, GitOps model)
- [ ] Identify and stabilize core APIs needed by vorchCD (workflow definitions, deploy, status)
- [ ] Add basic deployment status primitives in `core` (ID, status, timestamps)
- [ ] Add Prometheus metrics for deployments (counts, failures, rollout durations)

### Q3 2026 – vorchCD & GitOps
- [ ] Implement first version of vorchCD service (reconciler loop, basic rolling updates, rollback)
- [ ] Add GitOps export in `cli` (e.g. `orchcli export --git`) and CD status commands
- [ ] Add Git connection + "Export to Git" / "View CD status" flows in `ui`
- [ ] Introduce basic multi-cluster awareness (cluster registry + per-environment targeting)

### Q4 2026 – AI Assistant (Design, Optimization, Docs)
- [ ] AI Design Assistant in `ui` (prompt → draft architecture, "optimize this" workflows)
- [ ] AI Optimization endpoint using deployment history + basic metrics (surfaced in UI + `cli`)
- [ ] AI Docs / Design Copilot to generate architecture docs and PR-ready summaries

### 2027+ – Stretch Goals
- [ ] Deep GitOps integration with ArgoCD/Flux as peers to vorchCD
- [ ] Progressive delivery (canary, traffic shifting with service mesh) and policy engine
- [ ] AI cost estimation, failure forensics, and non-technical explainer modes

## Vision

KubeOrch aims to be the standard visual workflow orchestrator for Kubernetes, making container orchestration accessible to developers of all skill levels through drag-and-drop interfaces and intelligent automation.

## Short-Term (Q2 2026)

**Theme: Project quality and CNCF readiness**

- [ ] Increase test coverage across all repositories (core 40%+, UI 30%+, CLI passing on all platforms)
- [ ] Submit CNCF Sandbox application
- [ ] Achieve OpenSSF Best Practices passing badge
- [ ] Enable DCO enforcement on all repositories
- [ ] Add Prometheus metrics endpoint to core
- [ ] Complete API reference documentation with request/response examples
- [ ] Fix CORS configuration (restrict allowed origins)
- [ ] Add rate limiting middleware to core API
- [ ] Generate OpenAPI/Swagger specification
- [ ] Add production deployment guide to docs
- [ ] Submit project to Cloud Native Landscape

## Medium-Term (Q3 2026)

**Theme: Production readiness and ecosystem integration**

- [ ] Helm chart for KubeOrch deployment
- [ ] Plugin development SDK and documentation
- [ ] Distributed tracing integration (OpenTelemetry)
- [ ] Expanded OAuth/OIDC provider support
- [ ] Workflow templates marketplace
- [ ] CI/CD pipeline integration (GitHub Actions, GitLab CI)
- [ ] Backup and restore functionality
- [ ] Improved resource monitoring and alerting
- [ ] CLI update/upgrade command

## Long-Term (Q4 2026 and beyond)

**Theme: Scale, multi-tenancy, and community growth**

- [ ] Multi-tenancy support with namespace isolation
- [ ] Advanced RBAC with fine-grained permissions
- [ ] Service mesh integration
- [ ] Multi-cloud deployment support
- [ ] Cost analysis and optimization recommendations
- [ ] Audit logging
- [ ] GitOps workflow support (ArgoCD, Flux integration)
- [ ] Grow external contributor base and multi-org maintainership

## How to Influence the Roadmap

The roadmap is driven by community input. To suggest changes:

1. Open a discussion in the [community repo](https://github.com/KubeOrch/community/discussions)
2. Propose an RFC in the [proposals directory](proposals/)
3. Comment on existing roadmap items in GitHub Issues

Maintainers review roadmap priorities at the beginning of each quarter.
