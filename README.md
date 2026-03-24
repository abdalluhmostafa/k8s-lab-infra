# k8s-lab-infra

FluxCD infrastructure repo for k8s lab cluster.

Mirrors the pattern used in production platform infra repos.

---

## How it works

FluxCD watches this repo and applies changes to the cluster automatically.

```
flux2/
  flux-system/
    base/           # Flux Kustomization objects (what to sync and from where)
      namespaces/   # Creates namespaces on the cluster
      helm-repositories/  # Registers Helm chart sources
    primary/        # Entry point — lists what base/ components are active
  kustomizations/
    base/           # The actual Kubernetes manifests being deployed
      namespaces/   # Namespace definitions
```

---

## Flow

```
GitHub repo (this)
      ↓
  FluxCD (running in cluster)
      ↓  watches flux2/flux-system/primary/
      ↓  reads Kustomization objects
      ↓  pulls manifests from flux2/kustomizations/
      ↓
  Applies to cluster
```

---

## Bootstrap (first time setup)

1. Install FluxCD CLI on your machine:
   ```bash
   brew install fluxcd/tap/flux
   ```

2. Bootstrap FluxCD pointing to this repo:
   ```bash
   flux bootstrap git \
     --url=ssh://git@github.com-personal/abdalluhmostafa/k8s-lab-infra \
     --branch=main \
     --path=flux2/flux-system/primary \
     --private-key-file=~/.ssh/id_rsa
   ```

3. FluxCD will install itself in `flux-system` namespace and start syncing.

---

## Adding something new

1. Add the Kubernetes manifest under `flux2/kustomizations/base/<name>/`
2. Add a `Kustomization` object under `flux2/flux-system/base/<name>/`
3. Reference it in `flux2/flux-system/primary/kustomization.yaml`
4. Push → FluxCD picks it up automatically

---

## Check sync status

```bash
flux get kustomizations
flux get sources git
kubectl get kustomization -n flux-system
```
