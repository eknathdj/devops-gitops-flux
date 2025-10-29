# DevOps GitOps Sample - FluxCD + GitHub Actions (Java) — Flux-only

Flux-only GitOps scaffold using **Flux v2** for deployment and **GitHub Actions** for CI.
Includes Java (Spring Boot) app, Dockerfile, Kustomize (base + dev overlay), optional Helm chart, Flux manifests,
and DevSecOps scans (PMD + Grype).

## Key Features
- Java Spring Boot sample (`pom.xml`, `src/...`)
- Dockerfile (Java 17)
- Kustomize manifests: `k8s/base`, `k8s/overlays/dev`
- Optional Helm chart: `charts/myapp`
- Flux manifests: `manifests/flux/*`
- CI: `.github/workflows/ci-cd.yml` (Maven build + PMD, Docker build/push, Grype scan, update kustomize & helm values)
- Optional **Flux Image Automation** to bump tags (instead of CI committing tag changes)

## Setup (CI secrets)
- `REGISTRY` (e.g., ghcr.io)
- `REGISTRY_USERNAME`
- `REGISTRY_PASSWORD`

## Bootstrap Flux
```bash
flux install
kubectl apply -n flux-system -f manifests/flux/gitrepository.yaml
kubectl apply -n flux-system -f manifests/flux/kustomization-k8s.yaml
kubectl apply -n flux-system -f manifests/flux/kustomization-helm.yaml
# optional image automation
kubectl apply -n flux-system -f manifests/flux/image-automation/
```
