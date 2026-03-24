# KubeOrch Roadmap

This document outlines the project's direction and planned milestones. The roadmap is reviewed and updated quarterly by the maintainers.

> **Last updated:** March 2026

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
