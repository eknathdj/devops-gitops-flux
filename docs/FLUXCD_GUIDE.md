# FluxCD Complete Guide: From Zero to GitOps Hero 🚀

## Table of Contents
1. [What is FluxCD?](#what-is-fluxcd)
2. [Core Concepts](#core-concepts)
3. [Architecture Overview](#architecture-overview)
4. [Installation & Setup](#installation--setup)
5. [Step-by-Step Implementation](#step-by-step-implementation)
6. [Advanced Features](#advanced-features)
7. [Troubleshooting](#troubleshooting)
8. [Best Practices](#best-practices)

## What is FluxCD?

FluxCD is a **GitOps operator** for Kubernetes that automatically synchronizes your cluster state with configurations stored in Git repositories. It follows the GitOps principles:

- **Declarative**: Everything is described declaratively
- **Versioned & Immutable**: Git as the single source of truth
- **Pulled Automatically**: Changes are pulled, not pushed
- **Continuously Reconciled**: Desired state is continuously applied

### Why FluxCD?
- 🔄 **Automated Deployments**: No manual kubectl commands
- 🔒 **Security**: Pull-based model, no cluster credentials in CI/CD
- 📊 **Observability**: Built-in monitoring and alerting
- 🔧 **Multi-tenancy**: Support for multiple teams and environments
- 🎯 **Progressive Delivery**: Canary deployments, A/B testing

## Core Concepts

### 1. Sources
Sources define where Flux should fetch configurations from:

- **GitRepository**: Git repositories containing Kubernetes manifests
- **HelmRepository**: Helm chart repositories
- **Bucket**: S3-compatible storage buckets

### 2. Kustomizations
Kustomizations define how to apply configurations from sources:

- **Path**: Which directory to sync from the source
- **Interval**: How often to check for changes
- **Prune**: Whether to delete resources not in Git

### 3. Helm Releases
Manages Helm chart deployments with GitOps principles.

### 4. Image Automation
Automatically updates container image tags in Git when new images are available.

## Architecture Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Git Repo      │    │   Container     │    │   Kubernetes    │
│                 │    │   Registry      │    │   Cluster       │
│  ┌───────────┐  │    │                 │    │                 │
│  │Manifests  │  │    │  ┌───────────┐  │    │  ┌───────────┐  │
│  │& Charts   │◄─┼────┼──┤New Images │  │    │  │   Flux    │  │
│  └───────────┘  │    │  └───────────┘  │    │  │Controllers│  │
└─────────────────┘    └─────────────────┘    │  └───────────┘  │
         ▲                                    │       │         │
         │                                    │       ▼         │
         │              ┌────────────────────┐│  ┌───────────┐  │
         └──────────────┤   Image Update     ││  │Application│  │
                        │   Automation       ││  │Resources  │  │
                        └────────────────────┘│  └───────────┘  │
                                              └─────────────────┘
```

## Installation & Setup

### Prerequisites
- Kubernetes cluster (1.20+)
- kubectl configured
- Git repository for storing manifests

### Step 1: Install Flux CLI

```bash
# macOS
brew install fluxcd/tap/flux

# Linux
curl -s https://fluxcd.io/install.sh | sudo bash

# Windows (using Chocolatey)
choco install flux
```

### Step 2: Verify Prerequisites

```bash
flux check --pre
```

### Step 3: Bootstrap Flux

```bash
# GitHub (recommended)
export GITHUB_TOKEN=<your-token>
flux bootstrap github \
  --owner=<github-username> \
  --repository=<repository-name> \
  --branch=main \
  --path=./clusters/my-cluster \
  --personal

# GitLab
export GITLAB_TOKEN=<your-token>
flux bootstrap gitlab \
  --owner=<gitlab-username> \
  --repository=<repository-name> \
  --branch=main \
  --path=./clusters/my-cluster
```

## Step-by-Step Implementation

### Phase 1: Basic GitOps Setup

#### 1. Create Git Repository Structure

```
your-repo/
├── apps/
│   ├── base/
│   └── overlays/
│       ├── dev/
│       ├── staging/
│       └── prod/
├── infrastructure/
│   ├── controllers/
│   └── configs/
└── clusters/
    └── my-cluster/
        └── flux-system/
```

#### 2. Define a GitRepository Source

Create `clusters/my-cluster/sources/git-repository.yaml`:

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: rocket-lab-repo
  namespace: flux-system
spec:
  interval: 1m0s
  ref:
    branch: main
  url: https://github.com/your-username/your-repo
```

#### 3. Create Kustomization for Applications

Create `clusters/my-cluster/apps/kustomization.yaml`:

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: rocket-lab-apps
  namespace: flux-system
spec:
  interval: 10m0s
  path: "./apps/overlays/dev"
  prune: true
  sourceRef:
    kind: GitRepository
    name: rocket-lab-repo
  validation: client
  healthChecks:
    - apiVersion: apps/v1
      kind: Deployment
      name: rocket-lab-app
      namespace: rocket-lab
```

### Phase 2: Application Deployment

#### 1. Create Base Application Manifests

`apps/base/deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rocket-lab-app
  namespace: rocket-lab
spec:
  replicas: 2
  selector:
    matchLabels:
      app: rocket-lab-app
  template:
    metadata:
      labels:
        app: rocket-lab-app
    spec:
      containers:
      - name: nginx
        image: docker.io/nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

#### 2. Create Environment Overlays

`apps/overlays/dev/kustomization.yaml`:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../../base
namespace: rocket-lab
images:
  - name: docker.io/nginx
    newTag: "1.21"
replicas:
  - name: rocket-lab-app
    count: 1
```

### Phase 3: Helm Integration

#### 1. Add Helm Repository

Create `infrastructure/sources/helm-repository.yaml`:

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: bitnami
  namespace: flux-system
spec:
  interval: 24h
  url: https://charts.bitnami.com/bitnami
```

#### 2. Deploy Helm Release

Create `infrastructure/controllers/nginx-ingress.yaml`:

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: nginx-ingress
  namespace: flux-system
spec:
  interval: 30m
  chart:
    spec:
      chart: nginx-ingress-controller
      version: "9.x.x"
      sourceRef:
        kind: HelmRepository
        name: bitnami
        namespace: flux-system
  values:
    service:
      type: LoadBalancer
    metrics:
      enabled: true
```

### Phase 4: Image Automation

#### 1. Create Image Repository

Create `clusters/my-cluster/image-automation/image-repository.yaml`:

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageRepository
metadata:
  name: rocket-lab-images
  namespace: flux-system
spec:
  image: docker.io/your-username/rocket-lab-app
  interval: 1m0s
```

#### 2. Define Image Policy

Create `clusters/my-cluster/image-automation/image-policy.yaml`:

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: rocket-lab-policy
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: rocket-lab-images
  policy:
    semver:
      range: ">=1.0.0"
```

#### 3. Configure Image Update Automation

Create `clusters/my-cluster/image-automation/image-update-automation.yaml`:

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata:
  name: rocket-lab-automation
  namespace: flux-system
spec:
  interval: 30m
  sourceRef:
    kind: GitRepository
    name: rocket-lab-repo
  git:
    checkout:
      ref:
        branch: main
    commit:
      author:
        name: fluxcd-bot
        email: fluxcd-bot@users.noreply.github.com
      messageTemplate: |
        Automated image update
        
        Automation name: {{ .AutomationObject }}
        
        Files:
        {{ range $filename, $_ := .Updated.Files -}}
        - {{ $filename }}
        {{ end -}}
        
        Objects:
        {{ range $resource, $_ := .Updated.Objects -}}
        - {{ $resource.Kind }} {{ $resource.Name }}
        {{ end -}}
        
        Images:
        {{ range .Updated.Images -}}
        - {{.}}
        {{ end -}}
    push:
      branch: main
  update:
    strategy: Setters
    path: "./apps"
```

#### 4. Add Image Policy Markers

Update your Kustomization files with policy markers:

```yaml
images:
  - name: docker.io/your-username/rocket-lab-app
    newTag: "1.0.0" # {"$imagepolicy": "flux-system:rocket-lab-policy"}
```

## Advanced Features

### Multi-Tenancy

#### 1. Tenant Structure

```
clusters/my-cluster/
├── tenants/
│   ├── team-a/
│   │   ├── rbac.yaml
│   │   └── kustomization.yaml
│   └── team-b/
│       ├── rbac.yaml
│       └── kustomization.yaml
```

#### 2. Tenant RBAC

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: team-a
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: team-a-admin
  namespace: team-a
subjects:
- kind: User
  name: team-a-lead
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: admin
  apiGroup: rbac.authorization.k8s.io
```

### Progressive Delivery with Flagger

#### 1. Install Flagger

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: flagger
  namespace: flux-system
spec:
  interval: 24h
  url: https://flagger.app
---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: flagger
  namespace: flux-system
spec:
  interval: 30m
  chart:
    spec:
      chart: flagger
      version: "1.x.x"
      sourceRef:
        kind: HelmRepository
        name: flagger
```

#### 2. Configure Canary Deployment

```yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: rocket-lab-app
  namespace: rocket-lab
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: rocket-lab-app
  progressDeadlineSeconds: 60
  service:
    port: 80
    targetPort: 80
  analysis:
    interval: 1m
    threshold: 5
    maxWeight: 50
    stepWeight: 10
    metrics:
    - name: request-success-rate
      thresholdRange:
        min: 99
      interval: 1m
```

### Monitoring and Alerting

#### 1. Enable Flux Monitoring

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: flux-monitoring
  namespace: flux-system
spec:
  interval: 30m
  chart:
    spec:
      chart: flux2-monitoring-example
      version: "*"
      sourceRef:
        kind: GitRepository
        name: flux-monitoring
        namespace: flux-system
  values:
    prometheus:
      enabled: true
    grafana:
      enabled: true
```

#### 2. Configure Alerts

```yaml
apiVersion: notification.toolkit.fluxcd.io/v1beta1
kind: Provider
metadata:
  name: slack
  namespace: flux-system
spec:
  type: slack
  channel: alerts
  secretRef:
    name: slack-webhook
---
apiVersion: notification.toolkit.fluxcd.io/v1beta1
kind: Alert
metadata:
  name: flux-alerts
  namespace: flux-system
spec:
  providerRef:
    name: slack
  eventSeverity: info
  eventSources:
    - kind: GitRepository
      name: '*'
    - kind: Kustomization
      name: '*'
```

## Troubleshooting

### Common Issues and Solutions

#### 1. Flux Controllers Not Starting

```bash
# Check flux system status
flux check

# View controller logs
kubectl logs -n flux-system deployment/source-controller
kubectl logs -n flux-system deployment/kustomize-controller
```

#### 2. Git Authentication Issues

```bash
# Check git repository status
flux get sources git

# Recreate git credentials
kubectl delete secret flux-system -n flux-system
flux create secret git flux-system \
  --url=https://github.com/username/repo \
  --username=username \
  --password=token
```

#### 3. Kustomization Failures

```bash
# Check kustomization status
flux get kustomizations

# View detailed events
kubectl describe kustomization app-name -n flux-system

# Force reconciliation
flux reconcile kustomization app-name
```

#### 4. Image Automation Not Working

```bash
# Check image repositories
flux get image repository

# Check image policies
flux get image policy

# View automation logs
kubectl logs -n flux-system deployment/image-automation-controller
```

### Debugging Commands

```bash
# General health check
flux check

# List all Flux resources
flux get all

# Force reconciliation
flux reconcile source git repo-name
flux reconcile kustomization app-name

# Suspend/resume resources
flux suspend kustomization app-name
flux resume kustomization app-name

# Export current configuration
flux export source git repo-name
flux export kustomization app-name
```

## Best Practices

### 1. Repository Structure

```
├── apps/
│   ├── base/           # Base configurations
│   ├── overlays/       # Environment-specific overlays
│   └── releases/       # Helm releases
├── infrastructure/
│   ├── controllers/    # Infrastructure controllers
│   ├── configs/        # Configuration management
│   └── sources/        # Helm repositories, etc.
├── clusters/
│   ├── production/     # Production cluster config
│   ├── staging/        # Staging cluster config
│   └── development/    # Development cluster config
└── scripts/           # Automation scripts
```

### 2. Security Best Practices

- **Use RBAC**: Implement proper role-based access control
- **Separate Secrets**: Store secrets in external secret management systems
- **Network Policies**: Implement network segmentation
- **Image Scanning**: Scan container images for vulnerabilities
- **Git Signing**: Sign Git commits for integrity

### 3. Operational Best Practices

- **Health Checks**: Configure health checks for all deployments
- **Resource Limits**: Set appropriate resource requests and limits
- **Monitoring**: Implement comprehensive monitoring and alerting
- **Backup**: Regular backup of cluster state and Git repositories
- **Documentation**: Maintain up-to-date documentation

### 4. Development Workflow

1. **Feature Branch**: Create feature branch for changes
2. **Local Testing**: Test changes locally with `flux diff`
3. **Pull Request**: Create PR with proper reviews
4. **Staging Deploy**: Automatic deployment to staging
5. **Production Deploy**: Controlled deployment to production

### 5. Disaster Recovery

- **Multi-cluster Setup**: Deploy across multiple clusters
- **Backup Strategy**: Regular backups of Git repositories
- **Recovery Procedures**: Documented recovery procedures
- **Testing**: Regular disaster recovery testing

## Conclusion

FluxCD provides a powerful GitOps platform that enables:

- **Automated Deployments**: Continuous deployment from Git
- **Declarative Management**: Infrastructure as Code
- **Security**: Pull-based deployment model
- **Scalability**: Multi-tenant and multi-cluster support
- **Observability**: Built-in monitoring and alerting

By following this guide, you'll have a robust GitOps workflow that scales with your organization's needs while maintaining security and operational excellence.

---

**Next Steps:**
1. Set up your first GitRepository and Kustomization
2. Implement image automation for your applications
3. Add monitoring and alerting
4. Explore advanced features like progressive delivery
5. Scale to multi-tenant and multi-cluster setups

Happy GitOps! 🚀