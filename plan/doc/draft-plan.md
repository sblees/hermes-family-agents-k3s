# Draft Plan: Local Kubernetes for Family Hermes Agents

**Project:** Hermes Agents on Local Kubernetes  
**Status:** Draft  
**Date:** 2026-07-02  
**Owner:** Deployment Planner  
**Target Executor:** Atlas (via opencode + kanban)

---

## 1. Objective

Deploy each Hermes agent as a capable personal assistant for a family member, running in its own isolated Kubernetes pod on a local k3s cluster. Agents maintain durable, long-term storage so they can retain context, research, briefs, and other artifacts over time. Agents communicate via Matrix and operate with strong isolation by default.

## 2. Key Requirements

- Local Kubernetes cluster (k3s)
- One pod per agent
- Per-agent persistent storage for long-term memory and artifacts
- Strong isolation between agents (namespace + NetworkPolicy)
- Matrix integration for all agents
- Namespace per family member
- Simple, durable, and extensible storage model
- General-purpose environment (not specialized for coding or research only)
- Future extensibility for tools like Obsidian (explicitly deferred)

## 3. Recommended Architecture

### 3.1 Kubernetes Distribution
- **k3s** (recommended)
  - Lightweight and suitable for homelab/family use
  - Includes `local-path` provisioner for persistent volumes

### 3.2 Isolation Model
- One namespace per family member (e.g. `hermes-shawn`, `hermes-alex`)
- Default-deny NetworkPolicy in every namespace
- Explicit allow rules only when required
- Agents cannot communicate with each other by default

### 3.3 Storage
- One PersistentVolumeClaim per agent using `local-path`
- Mounted at `/workspace` inside the pod
- Designed for long-term accumulation of research, briefs, notes, and context
- Simple filesystem-based storage in the initial phase (no advanced knowledge base tooling yet)

### 3.4 Matrix Integration
- Each agent runs as a Matrix bot
- Credentials stored in Kubernetes Secrets
- One Matrix account per agent

## 4. Phases (for Atlas Execution)

### Phase 1: Cluster Foundation
- Install k3s
- Verify `local-path` storage class
- Create base namespace structure

### Phase 2: Isolation & Security
- Implement default-deny NetworkPolicies
- Define labeling and naming conventions

### Phase 3: Agent Pod Template
- Create reusable pod/Deployment template
- Define PVC structure for durable storage
- Package using Kustomize (tentative)

### Phase 4: Matrix Bot Integration
- Configure agents as Matrix bots
- Secret management per agent

### Phase 5: Onboarding Process
- Script or procedure for adding new family members and agents

## 5. Open Decisions

- Packaging tool: Kustomize vs Helm
- Resource limits per agent
- Level of operational tooling required
- Update and rollout strategy
- How agents surface output (e.g. daily briefs) to users

## 6. Deferred Items

- Backup and data protection strategy (to be addressed after core functionality works)
- Integration with Obsidian or other knowledge base tools (much later phase)
- Advanced memory systems (e.g. TurboVec) — not required in initial deployment

## 7. MCP Tools Strategy (Simplified)

### Guiding Principle
Keep MCP usage minimal. Do not introduce a central MCP orchestration layer.

### Recommended Approach

**Shared MCP Servers**
- Only run **one** shared MCP server: Filesystem MCP
- This provides controlled access to agent workspaces

**Agent Capabilities**
- Agents are general-purpose personal assistants
- They should be able to perform research, generate customized briefs, save artifacts, and handle varied tasks over time
- No coding-specific toolchain (OpenCode, TurboVec, etc.) is required in the initial deployment

**MCP Configuration**
- Each agent declares its MCP tools at startup via configuration
- No central Hermes MCP orchestrator is used

### Rationale
- Reduces complexity and maintenance overhead
- Aligns with strong per-agent isolation
- Keeps the system flexible for future capabilities

## 8. Inference Configuration (Hermes-visible)

Hermes expects a custom provider named `local-openrouter` defined in its configuration. All model calls are made against this provider.

**Required Configuration (from current Hermes setup):**
- Provider name: `local-openrouter`
- Base URL: Points to an OpenAI-compatible endpoint (currently `http://127.0.0.1:4200/v1`)
- Default model alias: `openrouter-primary`
- Additional model aliases used internally: `openrouter-compact`, `openrouter-summary`, `openrouter-title`, `openrouter-browser-validation`
- Fallback provider support (currently configured for `xai-oauth` / Grok)

**Kubernetes Implications:**
- Each agent pod must have access to an equivalent `local-openrouter` endpoint.
- The endpoint can be provided via a Kubernetes Service, external hostname, or sidecar.
- API keys and model routing details remain encapsulated behind this endpoint (invisible to Hermes).

## 9. Foundational Configuration Standards (Initial Build)

These standards define concrete locations and patterns to ensure consistency and repeatability during the foundational deployment.

### 9.1 Hermes Configuration
- **Standard path inside container:** `/etc/hermes/config.yaml`
- **Injection method:** Kubernetes ConfigMap mounted as a file
- **Per-agent customization:** Use agent-specific ConfigMaps when overrides are needed
- **Recommendation:** Create a base `hermes-config` ConfigMap and allow per-namespace overrides

### 9.2 Secrets Management
- **Matrix credentials:** Stored in Kubernetes Secrets (one per agent). Injected as environment variables or mounted files.
- **Bitwarden integration (current approach):**
  - Continue using Hermes’ native Bitwarden Secrets support
  - Store the `BWS_ACCESS_TOKEN` in a Kubernetes Secret
  - Mount or inject `BWS_ACCESS_TOKEN` into pods so the existing `bitwarden:` configuration continues to work
  - Project ID (`3d982cf3-3ab1-436d-949d-b45b01802063`) can be included in the mounted `config.yaml`
- **Inference credentials:** Kept behind the `local-openrouter` endpoint (not exposed directly to Hermes pods)

### 9.3 Inference Endpoint
- **Recommended approach:** Deploy the inference endpoint (LiteLLM / local-openrouter) in its own namespace or as a host-accessible service.
- **Access pattern:** Agents connect via `http://inference.local-openrouter.svc.cluster.local:4200/v1` (or equivalent Service DNS).
- Since the cluster runs on the same Mac, hostPort or host networking can be used initially for simplicity.

### 9.4 Labeling & Naming Conventions
- Namespace: `hermes-<family-member>`
- Common labels:
  - `app.kubernetes.io/name: hermes-agent`
  - `hermes.family/member: <name>`
  - `hermes.agent/id: <agent-name>`

### 9.5 Resource Defaults (Initial) — 25 GB Total Budget

**Global Constraint:** Plan for a **maximum of ~25 GB RAM** total across all agents + inference endpoint + system overhead.

**Recommended Starting Allocations:**

| Component              | Memory Request | Memory Limit | CPU Request | Notes |
|------------------------|----------------|--------------|-------------|-------|
| Inference endpoint     | 8–12 Gi        | 14 Gi        | 4–6         | Local models will be the largest consumer |
| Each Hermes agent      | 1.5–2 Gi       | 3 Gi         | 500m–1      | Conservative but functional |
| **Max recommended agents** | —           | —            | —           | **Start with 2–4 agents** until usage is measured |

**Rationale:**
- Leaves comfortable headroom within the 25 GB total budget
- Prevents any single agent from starving the inference workload
- Conservative limits make it easier to add more agents later if actual usage is lower than expected

---

**Note:** This is a draft. Family member names should remain as placeholders until execution time. The system is designed to support capable personal assistant behavior with durable context and storage.