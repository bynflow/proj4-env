# proj4-env

Environment repository for Project 4.

This repository contains the Kubernetes environment configuration for the application deployed through GitOps workflows.

The application source code is intentionally separated into a dedicated application repository:

* proj4-app

This repository owns the operational and deployment layer of the system.

---

## Responsibilities

This repository contains:

* Kubernetes manifests
* Kustomize overlays
* namespace structure
* ingress configuration
* TLS configuration
* RBAC configuration
* NetworkPolicy baseline
* deployment governance

---

## GitOps architecture

The system follows a multi-repository GitOps model:

* proj4-app → application source + CI
* proj4-env → deployment state + environment governance

This separation improves:

* operational clarity
* deployment governance
* security boundaries
* environment lifecycle management

---

## Environments

Current environments:

* dev
* staging
* prod

Managed through Kustomize overlays.

---

## Security baseline

Implemented hardening includes:

* RBAC
* NetworkPolicy
* TLS ingress
* Kubernetes Secrets baseline

---

## Observability integration

The deployed application exposes Prometheus metrics through:

```text
/metrics
```

The environment repository manages the Kubernetes-side integration required for observability workflows.

---

## Repository role

This repository owns:

* deployment manifests
* Kubernetes configuration
* environment governance
* cluster-facing configuration

This repository does NOT own:

* application source code
* unit tests
* Docker build logic
* CI application pipeline

Those responsibilities belong to:

* proj4-app

---

## Commands

```bash
nano README.md
```

Replace the entire current README content with this version.

Then execute:

```bash
git add README.md
git commit -m "docs: fix proj4-env README content"
git push origin main
```
