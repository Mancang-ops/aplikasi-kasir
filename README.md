# aplikasi-kasir: GitOps lab repo

Desired state for the **GitOps + Argo CD** lab. Argo CD watches this repo; nothing
is deployed with `kubectl apply` except the root app (`argocd/root-app.yaml`).

```
apps/
  kasir/                  Kasir Batang Garing PWA (Kustomize)
    base/                 Deployment, Service, Ingress, PreSync + PostSync hook Jobs, sync waves
    overlays/dev/         1 replica, kasir-dev.localtest.me, SealedSecret kasir-cloud
    overlays/prod/        3 replicas, kasir-prod.localtest.me
  nginx-helm/             Small Helm chart; values-dev.yaml / values-prod.yaml
argocd/
  root-app.yaml           App-of-Apps (apply once by hand)
  projects/gitops-lab.yaml  AppProject: this repo only, 4 namespaces, only Namespace cluster-scoped
  applicationset.yaml     kasir-dev (auto-sync, prune, self-heal) + kasir-prod (manual promotion)
  apps/                   nginx-helm-dev, nginx-helm-prod Applications
```

## Bootstrap

```bash
kubectl apply -f argocd/root-app.yaml
argocd app sync kasir-prod      # prod is a manual promotion
```

## Environments

| | dev | prod |
|---|---|---|
| kasir replicas | 1 | 3 |
| kasir image | `localhost:5050/kasir:1.0.0` (overlay `images:`) | `localhost:5050/kasir:1.0.0` |
| kasir sync | automated, prune, selfHeal | manual |
| nginx-helm replicas | 1 | 2 |

## Pinned versions

| Component | Version |
|---|---|
| k3s | `rancher/k3s:v1.36.5-k3s1` (k3d v5.9.0) |
| Argo CD | Helm chart `argo/argo-cd` 10.11.1 (Argo CD v3.5.6) |
| Sealed Secrets | Helm chart 2.20.0 (controller v0.40.0) |
| kasir image base | `nginxinc/nginx-unprivileged:1.30.6-alpine` |
| nginx-helm image | `nginxinc/nginx-unprivileged:1.30.6-alpine` |
| hook jobs | `busybox:1.38.0` |

## Secrets

No plain-text secrets live in this repo. `apps/kasir/overlays/dev/sealedsecret-kasir-cloud.yaml`
is a **SealedSecret** (demo values only) that only the lab cluster's controller can decrypt.
After a cluster rebuild it must be re-sealed (see LAB-GUIDE, Bonus).

## Tags

- `lab-start`: starting point for the exercises
- `lab-final`: state after all exercises
