# Rocket Lab - DevOps GitOps Sample with FluxCD + GitHub Actions (Nginx) — Flux-only

Flux-only GitOps scaffold using **Flux v2** for deployment and **GitHub Actions** for CI.
Includes nginx web server, Dockerfile, Kustomize (base + dev overlay), optional Helm chart, Flux manifests,
and DevSecOps scans (Grype).

## Key Features
- Nginx web server with custom HTML page
- Dockerfile (nginx:alpine)
- Kustomize manifests: `k8s/base`, `k8s/overlays/dev` (using `rocket-lab` namespace)
- Optional Helm chart: `charts/myapp`
- Flux manifests: `manifests/flux/*`
- CI: `.github/workflows/ci-cd.yml` (Docker build/push, Grype scan, update kustomize & helm values)
- Optional **Flux Image Automation** to bump tags (instead of CI committing tag changes)

## Setup (CI secrets)
- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`

## Bootstrap Flux
```bash
flux install
kubectl apply -n flux-system -f manifests/flux/gitrepository.yaml
kubectl apply -n flux-system -f manifests/flux/kustomization-k8s.yaml
kubectl apply -n flux-system -f manifests/flux/kustomization-helm.yaml
# optional image automation
kubectl apply -n flux-system -f manifests/flux/image-automation/
```
