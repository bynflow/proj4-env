# gitops-observability-platform-env

GitOps environment repository for Project 4: **GitOps Environment Governance & Production Hardening**.

This repository contains the Kubernetes environment layer for a multi-repository DevOps platform.

The application code, tests, CI pipeline, Docker build, GHCR publishing, Trivy scanning, dependency audit and Prometheus instrumentation are intentionally separated into the application repository:

* [`gitops-observability-platform-app`](https://github.com/bynflow/gitops-observability-platform-app)

This repository owns the **deployment state**, **environment governance**, and **Kubernetes hardening layer**.

---

## Project role

`gitops-observability-platform-env` is responsible for:

* Kubernetes manifests
* Kustomize base and overlays
* ArgoCD-driven GitOps deployment
* dev / staging / prod environment separation
* image promotion through immutable `sha-*` tags
* Ingress configuration
* TLS configuration
* RBAC hardening
* NetworkPolicy baseline
* Kubernetes Secret runtime integration
* observability-side deployment integration

It does **not** own:

* application source code
* unit tests
* Docker image build logic
* GitHub Actions application CI
* Python dependency scanning
* application-level Prometheus instrumentation

Those responsibilities belong to `gitops-observability-platform-app`.

---

## Architecture context

```text
gitops-observability-platform-app
   |
   | builds and publishes immutable image
   v
GHCR
   |
   | image tag selected in env repo
   v
gitops-observability-platform-env
   |
   | desired state stored in Git
   v
ArgoCD
   |
   | reconciles cluster state
   v
Kubernetes runtime
   |
   | Ingress + TLS + RBAC + NetworkPolicy + Secrets
   v
gitops-observability-platform-app workload
```

The application repository produces artifacts.
The environment repository governs where and how those artifacts run.

---

## GitOps model

Project 4 uses a multi-repository GitOps model:

| Repository  | Responsibility                                        |
| ----------- | ----------------------------------------------------- |
| `gitops-observability-platform-app` | application code, CI, Docker build, GHCR image        |
| `gitops-observability-platform-env` | Kubernetes desired state, overlays, ArgoCD deployment |

This separation improves:

* deployment governance
* environment isolation
* operational clarity
* security boundaries
* rollback and promotion control

---

## Environments

The repository defines three Kubernetes environments:

```text
dev
staging
prod
```

Each environment is represented through a Kustomize overlay:

```text
apps/proj4/overlays/dev
apps/proj4/overlays/staging
apps/proj4/overlays/prod
```

Promotion is performed by updating the immutable image tag in the corresponding overlay:

```yaml
images:
  - name: ghcr.io/bynflow/gitops-observability-platform-app
    newTag: sha-<commit>
```

---

## Repository structure

```text
gitops-observability-platform-env/
├── apps/
│   └── proj4/
│       ├── base/
│       │   ├── deployment.yaml
│       │   ├── service.yaml
│       │   ├── ingress.yaml
│       │   ├── serviceaccount.yaml
│       │   ├── role.yaml
│       │   ├── rolebinding.yaml
│       │   ├── networkpolicy.yaml
│       │   └── kustomization.yaml
│       └── overlays/
│           ├── dev/
│           ├── staging/
│           └── prod/
└── argocd/
```

---

## Security hardening baseline

This repository implements a first Kubernetes hardening baseline:

* dedicated ServiceAccount for the application workload
* Role and RoleBinding for explicit RBAC
* NetworkPolicy for controlled pod ingress traffic
* TLS termination on Ingress
* Kubernetes Secret injection through `secretKeyRef`
* separation between runtime secrets and Docker image artifacts

This is not a full enterprise security platform.
It is a practical baseline showing production-oriented thinking.

---

## Ingress and TLS

The application is exposed through Kubernetes Ingress.

TLS is configured at the Ingress layer using a Kubernetes TLS Secret.

Runtime validation was performed through:

```bash
curl -k https://proj4.local/health
curl -k https://proj4.local/metrics
```

---

## Observability integration

The deployed application exposes Prometheus metrics through:

```text
/metrics
```

The environment layer ensures that the workload is reachable through the Kubernetes networking path:

```text
HTTPS → Ingress → Service → Pod → /metrics
```

This validates observability through the real runtime path rather than only through local port-forwarding.

---

## GitOps deployment flow

```text
1. gitops-observability-platform-app builds an immutable image
2. image is pushed to GHCR as sha-<commit>
3. gitops-observability-platform-env overlay selects the target sha
4. ArgoCD detects the desired state change
5. ArgoCD reconciles the Kubernetes cluster
6. Kubernetes runs the selected image
7. runtime is validated through HTTPS and /metrics
```

This creates a controlled promotion model across:

```text
dev → staging → prod
```

---

## Runtime validation examples

Check deployed resources:

```bash
kubectl get all -n proj4-dev
```

Check deployed image:

```bash
kubectl describe pod -n proj4-dev | grep Image:
```

Check HTTPS health endpoint:

```bash
curl -k https://proj4.local/health
```

Check Prometheus metrics:

```bash
curl -k https://proj4.local/metrics | grep app_
```

---

## What this repository demonstrates

This repository demonstrates:

* GitOps environment governance
* multi-environment Kubernetes deployment
* separation of application and environment concerns
* ArgoCD reconciliation model
* immutable image promotion
* Kubernetes hardening baseline
* runtime secret injection
* Ingress + TLS exposure
* observability through the real cluster path
* portfolio-grade DevOps architecture design

---

## Related repository

Application repository:

* [`gitops-observability-platform-app`](https://github.com/bynflow/gitops-observability-platform-app)
