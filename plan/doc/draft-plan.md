# Draft Plan: Local Kubernetes for Family Hermes Agents

**Project:** Hermes Agents on Local Kubernetes  
**Status:** Draft  
**Date:** 2026-07-02  
**Owner:** Deployment Planner (research phase)  
**Target Executor:** Atlas (via opencode + kanban)

---

## 1. Objective

Run each Hermes agent in its own isolated Kubernetes pod on a local cluster, with durable per-agent storage for projects and skills. Agents communicate via Matrix and are assigned to family members.

## 2. Key Requirements

- Local Kubernetes cluster
- One pod per agent
- Per-agent persistent workspace (`/workspace/projects`, `/workspace/skills`, etc.)
- Strong isolation between pods (no cross-agent communication by default)
- Matrix integration for all agents
- Namespace per family member
- Simple, durable storage for initial implementation

## 3. Recommended Architecture

### 3.1 Kubernetes Distribution
- **k3s** (recommended)
  - Lightweight and suitable for homelab/family use
  - Includes `local-path` provisioner for persistent volumes

### 3.2 Isolation Model
- One namespace per family member (e.g. `hermes-shawn`, `hermes-alex`)
- Default-deny NetworkPolicy in every namespace
- Explicit allow rules only when required

### 3.3 Storage
- One PersistentVolumeClaim per agent using `local-path`
- Mounted at `/workspace` inside the pod

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
- Define PVC structure
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

## 6. Deferred Items

- Backup and data protection strategy (to be addressed after core functionality works)

---

**Note:** This is a draft. Family member names should remain as placeholders until execution time.