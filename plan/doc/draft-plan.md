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

---

**Note:** This is a draft. Family member names should remain as placeholders until execution time. The system is designed to support capable personal assistant behavior with durable context and storage.