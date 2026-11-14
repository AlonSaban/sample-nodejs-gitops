# Sample Node.js — GitOps Deployment Repository

This repository holds the declarative Kubernetes manifests, Helm chart, and ArgoCD Application configuration for the `sample-nodejs` application. It serves as the single source of truth for cluster state, decoupled from the application source repository (`AlonSaban/sample-nodejs`).

## Architectural Decisions

### 1. Separate GitOps Repository vs. Single Monorepo
- **Choice:** Separate GitOps repository (`sample-nodejs-gitops`) monitored by ArgoCD.
- **Rationale:**
  - **Prevents CI Loops:** Automated image tag updates committed to this repository trigger ArgoCD syncs without re-triggering application builds, SAST scans, or Docker image builds in the source repo.
  - **RBAC & Security Boundaries:** Isolates application source code permissions from cluster deployment state.
  - **Clean Audit Trail:** Git commit history strictly reflects environment and release state transitions (`values.yaml` image tag updates).

### 2. Deployment vs. StatefulSet
- **Choice:** Kubernetes `Deployment`.
- **Rationale:** The Node.js service is completely stateless—it requires neither persistent volume claims (`PVC`) nor stable, ordered network identities. A `Deployment` provides zero-downtime rolling updates (`maxUnavailable: 0`, `maxSurge: 1`), horizontal scalability, and automated pod replacement.

### 3. Secrets Management Note
- For local reproducibility in this challenge, `values.yaml` includes a non-sensitive placeholder value injected via a Kubernetes `Secret` (`secretKeyRef`). In a production environment, secrets would be managed externally via **External Secrets Operator (ESO)** or **Bitnami Sealed Secrets** rather than stored in Git.

## Repository Structure

```text
.
├── argocd/
│   └── application.yaml       # Declarative ArgoCD Application manifest
├── sample-nodejs/             # Helm Chart
│   ├── Chart.yaml             # Chart metadata and versioning
│   ├── values.yaml            # Environment configuration & image tag
│   └── templates/
│       ├── _helpers.tpl       # Standard Helm template helpers
│       ├── configmap.yaml     # Non-sensitive runtime config (PORT)
│       ├── deployment.yaml    # Pod spec, RollingUpdate, securityContext, probes & resource limits
│       ├── ingress.yaml       # NGINX Ingress routing rules
│       ├── pdb.yaml           # PodDisruptionBudget for voluntary eviction protection
│       ├── secret.yaml        # Sensitive runtime config injection
│       └── service.yaml       # ClusterIP internal load balancing
└── README.md