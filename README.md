# 🚀 Rocket Lab - GitOps with FluxCD Demo

A complete **GitOps** demonstration using **FluxCD v2** and **GitHub Actions** for continuous deployment of a nginx web application to Kubernetes.

[![CI-CD](https://github.com/eknathdj/devops-gitops-flux/actions/workflows/ci-cd.yml/badge.svg)](https://github.com/eknathdj/devops-gitops-flux/actions/workflows/ci-cd.yml)

## 🎯 What This Repository Demonstrates

This repository showcases a **production-ready GitOps workflow** that automatically:
- 🔄 **Builds and pushes** container images via GitHub Actions
- 🔍 **Scans images** for vulnerabilities using Grype
- 📦 **Deploys applications** to Kubernetes using FluxCD
- 🔄 **Updates deployments** automatically when new images are available
- 🎛️ **Manages infrastructure** declaratively through Git

## 🏗️ Architecture Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   GitHub Repo   │    │   Docker Hub    │    │   Kubernetes    │
│                 │    │                 │    │   Cluster       │
│  ┌───────────┐  │    │  ┌───────────┐  │    │  ┌───────────┐  │
│  │App Code & │  │    │  │Container  │  │    │  │   Flux    │  │
│  │Manifests  │◄─┼────┼──┤Images     │  │    │  │Controllers│  │
│  └───────────┘  │    │  └───────────┘  │    │  └───────────┘  │
└─────────────────┘    └─────────────────┘    │       │         │
         ▲                                    │       ▼         │
         │              ┌────────────────────┐│  ┌───────────┐  │
         └──────────────┤   GitHub Actions   ││  │Rocket Lab │  │
                        │   CI/CD Pipeline   ││  │   App     │  │
                        └────────────────────┘│  └───────────┘  │
                                              └─────────────────┘
```

## 🚀 Quick Start

**Want to get started immediately?** Follow our **[⚡ Quick Start Guide](./docs/QUICK_START.md)** for a 10-minute setup!

### TL;DR - One Command Deploy
```bash
# Clone, install Flux, and deploy
git clone https://github.com/eknathdj/devops-gitops-flux.git
cd devops-gitops-flux
curl -s https://fluxcd.io/install.sh | sudo bash
flux install
kubectl apply -f manifests/flux/
kubectl port-forward svc/devops-sample-svc 8080:80 -n rocket-lab
```

Then visit `http://localhost:8080` to see your GitOps-deployed application! 🎉

### Detailed Setup
For step-by-step instructions with explanations, see the **[Complete Quick Start Guide](./docs/QUICK_START.md)**.

## 📁 Repository Structure

```
├── 📁 .github/workflows/     # GitHub Actions CI/CD pipeline
├── 📁 charts/myapp/          # Helm chart for the application
├── 📁 docs/                  # Comprehensive documentation
├── 📁 k8s/                   # Kubernetes manifests
│   ├── 📁 base/              # Base Kustomize resources
│   └── 📁 overlays/dev/      # Environment-specific overlays
├── 📁 manifests/flux/        # FluxCD configuration
│   └── 📁 image-automation/  # Image automation setup
├── 📄 Dockerfile             # Container image definition
├── 📄 index.html             # Custom nginx welcome page
├── 📄 nginx.conf             # Nginx configuration
└── 📄 README.md              # This file
```

## 🔧 Key Features

### ✅ **GitOps Workflow**
- **Declarative Configuration**: Everything defined in Git
- **Automated Sync**: FluxCD continuously reconciles cluster state
- **Git as Source of Truth**: All changes tracked and versioned

### ✅ **CI/CD Pipeline**
- **Automated Builds**: GitHub Actions builds and pushes images
- **Security Scanning**: Grype scans for vulnerabilities
- **Automated Updates**: Image tags updated automatically

### ✅ **Production Ready**
- **Health Checks**: Readiness and liveness probes configured
- **Resource Limits**: CPU and memory limits set
- **Multi-Environment**: Kustomize overlays for different environments
- **Monitoring**: Built-in observability with FluxCD

### ✅ **Image Automation**
- **Automatic Updates**: New image versions deployed automatically
- **Policy-Based**: Configurable update policies
- **Git Integration**: Changes committed back to repository

## 🎛️ Configuration

### Environment Variables (GitHub Secrets)
```bash
DOCKERHUB_USERNAME=<your-dockerhub-username>
DOCKERHUB_TOKEN=<your-dockerhub-access-token>
```

### FluxCD Components
- **GitRepository**: Monitors this repository for changes
- **Kustomization**: Applies Kubernetes manifests
- **ImageRepository**: Monitors Docker Hub for new images
- **ImagePolicy**: Defines update policies
- **ImageUpdateAutomation**: Automatically updates image tags

## 📊 Monitoring and Observability

### Check FluxCD Status
```bash
# Overall health check
flux check

# View all Flux resources
flux get all

# Check specific components
flux get sources git
flux get kustomizations
flux get image repository
```

### Application Status
```bash
# Check pods
kubectl get pods -n rocket-lab

# View deployment status
kubectl get deployment devops-sample-app -n rocket-lab

# Check service
kubectl get svc devops-sample-svc -n rocket-lab
```

## 🔍 Troubleshooting

### Common Issues
1. **ImagePullBackOff**: Check image name and registry access
2. **Flux Not Syncing**: Verify Git repository access and credentials
3. **Pods Not Starting**: Check resource limits and node capacity

### Debug Commands
```bash
# View Flux controller logs
kubectl logs -n flux-system deployment/source-controller
kubectl logs -n flux-system deployment/kustomize-controller

# Check events
kubectl get events -n rocket-lab --sort-by='.lastTimestamp'

# Force reconciliation
flux reconcile source git devops-repo
flux reconcile kustomization rocket-lab-k8s
```

## 📚 Documentation

Comprehensive documentation is available in the **[`docs/`](./docs/)** directory:

### 🎓 **Getting Started**
- **[⚡ Quick Start Guide](./docs/QUICK_START.md)** - Get running in 10 minutes
- **[📖 Documentation Index](./docs/README.md)** - Complete documentation overview

### 📖 **Learning Resources**
- **[🎯 FluxCD Complete Guide](./docs/FLUXCD_GUIDE.md)** - Comprehensive FluxCD tutorial from basics to advanced
- **[🔧 Configuration Analysis](./docs/FLUX_ANALYSIS_REPORT.md)** - Deep dive into Flux configuration best practices

### 🛠️ **Technical Details**
- **[✅ Applied Fixes](./docs/FLUX_FIXES_APPLIED.md)** - Summary of all improvements made
- **[🧪 Test Results](./docs/FLUX_TEST_RESULTS.md)** - Validation and testing documentation

### 🚨 **Troubleshooting**
- **[🔍 ImagePull Issues](./docs/IMAGEPULL_FIX_SUMMARY.md)** - Container image troubleshooting
- **[📊 Status Updates](./docs/CURRENT_STATUS_UPDATE.md)** - Real-time issue resolution logs

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test with your own cluster
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🎯 Learning Outcomes

By exploring this repository, you'll learn:
- ✅ **GitOps Principles**: Declarative, versioned, and automated deployments
- ✅ **FluxCD Operations**: Source management, kustomizations, and image automation
- ✅ **Kubernetes Best Practices**: Health checks, resource management, and observability
- ✅ **CI/CD Integration**: Automated builds, testing, and deployments
- ✅ **Container Security**: Image scanning and vulnerability management

## 🚀 Next Steps

1. **Explore the Code**: Browse through the manifests and configurations
2. **Deploy Locally**: Set up your own cluster and deploy the application
3. **Customize**: Modify the application and see GitOps in action
4. **Scale Up**: Add more environments and applications
5. **Production**: Apply these patterns to your real-world projects

## 🌟 Project Status

**Status: FULLY OPERATIONAL** ✅

- ✅ GitOps workflow implemented and tested
- ✅ Application successfully deployed and running
- ✅ Image automation configured and ready
- ✅ CI/CD pipeline functional
- ✅ Comprehensive documentation complete

See **[Project Status](./docs/PROJECT_STATUS.md)** for detailed metrics and achievements.

## 🎯 Success Metrics

```bash
# Current deployment status
kubectl get all -n rocket-lab

# Expected output:
# NAME                                    READY   STATUS    RESTARTS   AGE
# pod/devops-sample-app-xxxxxxxxx-xxxxx   1/1     Running   0          Xm
# pod/devops-sample-app-xxxxxxxxx-xxxxx   1/1     Running   0          Xm
# 
# NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
# service/devops-sample-svc   ClusterIP   10.x.x.x       <none>        80/TCP    Xm
# 
# NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/devops-sample-app   2/2     2            2           Xm
```

---

**Happy GitOps!** 🎉 

**Ready to deploy?** Start with our **[⚡ Quick Start Guide](./docs/QUICK_START.md)**

For questions or support, please open an issue or check the **[comprehensive documentation](./docs/)**.
