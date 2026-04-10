# KubeOrch – vorchCD & AI Assistant Roadmap

This roadmap extends the existing community roadmap with a focused track for:
- **vorchCD** – a first-class continuous delivery (CD) layer for KubeOrch
- **GitOps integration** – Git as source of truth for visual workflows
- **AI Assistant** – AI-driven design, optimization, and documentation

It is meant to live alongside `community/ROADMAP.md` and be referenced from there.

---

## 1. Overview

KubeOrch already provides:
- A **visual canvas** (`ui`) for designing Kubernetes architectures
- A **core orchestration engine** (`core`) that generates manifests and connects services
- A **developer CLI** (`cli`) for local orchestration and contribution workflows
- A **community repo** (`community`) with governance and a high-level roadmap
- **Docs** (`docs`) and a **landing site** (`landing`)

This roadmap adds a focused track to:
- Introduce **vorchCD**, a CD engine that continuously reconciles desired state to clusters
- Add **GitOps workflows**, so visual designs can be exported to Git and deployed from there
- Layer in an **AI Assistant** that:
  - Designs architectures from prompts
  - Optimizes existing setups based on signals
  - Generates documentation and explanations

---

## 2. Current State (What Exists Today)

### 2.1 Core (`core`)

- REST API for:
  - Deploying visual workflows → Kubernetes manifests (`/v1/workflows/deploy`)
  - Listing templates (`/v1/templates`)
  - Listing plugins (`/v1/plugins`)
  - Auto-wiring connections (`/v1/connections/auto`)
- JSON graph → YAML manifest transformation
- Service template system with typed ports and value templates
- Auth and basic API plumbing (JWT, middleware, basic routes)
- No dedicated CD engine; deployment is currently **request-driven**, not **state-reconciler-driven**.

### 2.2 UI (`ui`)

- Next.js + React Flow canvas for creating and editing workflows
- Component library (150+ services, from templates)
- One-click plugins
- Real-time log streaming (WebSocket)
- No first-class concept of:
  - Environments (staging/prod)
  - GitOps bindings
  - Deployment histories or rollout strategies

### 2.3 CLI (`cli`)

- Strong local-dev story:
  - `orchcli init`, `start`, `stop`, `logs`, `status`, etc.
  - Handles cloning, concurrent operations, config locking
- No commands yet for:
  - GitOps export
  - CD status
  - Environment promotion

### 2.4 Community & Docs (`community`, `docs`)

- Existing `community/ROADMAP.md` with:
  - GitOps workflow support (ArgoCD/Flux) as a **long-term** item
  - Plugin SDK and marketplace as medium-term items
- No dedicated docs yet for:
  - A native KubeOrch CD layer
  - AI-assisted workflows

---

## 3. Phase 1 – Foundations (Align with Existing Roadmap)

**Goal:** Prepare the codebase and community structure so vorchCD + AI can land cleanly and align with CNCF aspirations.

### 3.1 Architecture & RFCs

- [ ] Draft an RFC in `community/proposals/` that covers:
  - vorchCD goals and scope
  - High-level architecture (services, data flow, APIs)
  - GitOps integration model
  - AI Assistant surface areas (UI, CLI, Core)
- [ ] Align with existing `community/ROADMAP.md` by:
  - Linking this document from there
  - Ensuring GitOps support here doesn’t conflict with planned ArgoCD/Flux integration but complements it.

### 3.2 Core & API Stabilization

- [ ] Identify and list APIs in `core` that vorchCD will depend on (workflow definitions, deployment endpoints, status endpoints).
- [ ] Ensure those APIs are covered by the **API Stability Policy** in `community/API_STABILITY.md`.
- [ ] Add basic deployment status primitives to `core` so vorchCD can read/write:
  - deployment ID
  - status (pending/running/succeeded/failed)
  - timestamps

### 3.3 Observability Hooks

- [ ] Add Prometheus metrics (already on the main roadmap) with:
  - deployment counts
  - deployment failures
  - rollout durations
- [ ] Ensure logs relevant to CD are structured well for future AI analysis.

---

## 4. Phase 2 – vorchCD & GitOps

**Goal:** Introduce a first iteration of vorchCD as a native CD engine and wire it tightly to GitOps workflows.

### 4.1 vorchCD Service

**Location:**
- Either a new repo `KubeOrch/vorchcd` or a module within `core` that can later be split out.

**Responsibilities:**
- Watch **desired state** from:
  - Git repositories (GitOps mode), and/or
  - KubeOrch Core (visual workflows as source of truth)
- Compare desired state to **actual cluster state**.
- Reconcile differences using Kubernetes APIs.

**Tasks:**
- [ ] Define the vorchCD domain model:
  - Application / workflow
  - Environment (staging, prod, etc.)
  - Deployment
  - Rollout strategy
- [ ] Implement a minimal reconciler loop:
  - Pull manifests (from Git or generated via Core)
  - Apply to cluster
  - Track status
- [ ] Add support for basic rollout strategies:
  - Rolling update
  - Immediate rollback on failed health checks
- [ ] Expose vorchCD status over an HTTP API for UI/CLI.

### 4.2 GitOps Integration

**Desired UX:**
- Visual design → exported to Git → vorchCD watches Git → cluster updated.

**Tasks in CLI (`cli`):**
- [ ] Add `orchcli export`:
  - `orchcli export --format=manifests --output=<dir>`
  - `orchcli export --git --repo=<url> --branch=<branch>` (optional auto-commit)
- [ ] Add `orchcli cd status`:
  - Show per-environment deployment status from vorchCD

**Tasks in UI (`ui`):**
- [ ] Add Git connection settings to projects:
  - Repo URL
  - Branch
  - Path prefix (e.g. `k8s/`)
- [ ] Add "Export to Git" and "View CD status" actions.

**Tasks in vorchCD:**
- [ ] Implement Git watchers:
  - Poll or webhook-based
  - Track last applied commit
- [ ] Implement environment mapping:
  - e.g. `main` → staging, `release/*` → prod (configurable)

### 4.3 Multi-Cluster Awareness (Basic)

- [ ] Define a cluster registry model:
  - Name, kubeconfig / auth, labels
- [ ] Allow vorchCD deployments to target specific clusters.
- [ ] Surface cluster selection in Core + UI as a basic dropdown or per-environment setting.

---

## 5. Phase 3 – AI Assistant (Design, Optimization, Docs)

**Goal:** Layer AI capabilities on top of the visual + CD foundation to help design, optimize, and document architectures.

### 5.1 AI Design Assistant

**Surface:** UI

**Use cases:**
- Prompt → architecture: "I want a Node.js API with Postgres, Redis, and basic monitoring".
- Refactor: "Optimize this for high availability".

**Tasks:**
- [ ] Define an internal schema for the KubeOrch workflow that’s AI-friendly (JSON representation with clear types and constraints).
- [ ] Implement a Core or sidecar service endpoint (`/ai/design`):
  - Input: natural language + optional existing workflow
  - Output: workflow JSON graph
- [ ] Add UI affordances:
  - "Describe what you’re building" prompt box that generates a draft graph.
  - "Ask AI to optimize" button that suggests modifications.

### 5.2 AI Optimization

**Surface:** UI + CLI

**Signals:**
- Deployment history and failure rates from vorchCD
- Basic metrics (once Prometheus is integrated)
- Resource requests/limits and topology from Core

**Tasks:**
- [ ] Define an `/ai/optimize` endpoint:
  - Input: workflow, deployment history, basic metrics snapshot
  - Output: ranked recommendations (e.g. tune CPU/memory, add retry policies, split services).
- [ ] Add UI panel for "Optimization suggestions" per project/environment.
- [ ] Add CLI command `orchcli ai optimize` that:
  - Pulls current workflow + basic metrics
  - Prints suggestions in markdown for PRs/docs.

### 5.3 AI Docs / Design Copilot

**Surface:** CLI + Docs + UI

**Use cases:**
- Generate human-readable documentation of the current architecture.
- Generate PR-ready descriptions when topology changes.

**Tasks:**
- [ ] Implement `/ai/docs` endpoint:
  - Input: workflow JSON
  - Output: markdown (system overview, components, dependencies, risks).
- [ ] Add `orchcli ai describe`:
  - Writes `ARCHITECTURE.md` or a chosen path from the current KubeOrch project.
- [ ] Add "Generate docs" UI action that shows an AI-generated description of the current workflow.

---

## 6. Phase 4 – Stretch Goals / Nice-to-Haves

These depend on adoption and community feedback once vorchCD + AI Assistant are in place.

### 6.1 Deep GitOps & Ecosystem Integration

- [ ] Native integration modes with ArgoCD and Flux (tying back to the existing roadmap item):
  - Export KubeOrch workflows as ArgoCD/Flux apps.
  - Optionally let vorchCD orchestrate them instead of replacing them.
- [ ] Signed manifests and supply chain metadata (SLSA, Sigstore etc.).

### 6.2 Advanced CD Features

- [ ] Progressive delivery strategies:
  - Canary with automated analysis
  - Traffic shifting with service mesh (Istio/Linkerd)
- [ ] Policy engine:
  - Validate workflows and rollouts against org policies before applying.

### 6.3 AI Assistant Enhancements

- [ ] Cost estimation:
  - Rough cost impact per deployment / topology change.
- [ ] Failure forensics:
  - Given a failed deployment and some logs/metrics, propose likely root causes.
- [ ] "Explain like I’m 5" mode:
  - Docs for non-Kubernetes stakeholders.
