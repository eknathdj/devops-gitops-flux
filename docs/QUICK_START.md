# ⚡ Quick Start Guide - Rocket Lab GitOps

Get up and running with GitOps and FluxCD in under 10 minutes!

## 🎯 What You'll Achieve

By the end of this guide, you'll have:
- ✅ FluxCD installed and configured
- ✅ Rocket Lab app deployed to Kubernetes
- ✅ GitOps workflow operational
- ✅ Image automation working

## 📋 Prerequisites

- Kubernetes cluster (minikube, kind, or cloud)
- kubectl configured and working
- Git and GitHub account
- Docker Hub account (optional, for CI/CD)

## 🚀 Step 1: Verify Your Setup

```bash
# Check kubectl connection
kubectl cluster-info

# Verify you can create resources
kubectl auth can-i create deployments --all-namespaces
```

## 🔧 Step 2: Install FluxCD

```bash
# Install Flux CLI
curl -s https://fluxcd.io/install.sh | sudo bash

# Verify installation
flux --version

# Check prerequisites
flux check --pre
```

## 📦 Step 3: Clone the Repository

```bash
# Clone the repo
git clone https://github.com/eknathdj/devops-gitops-flux.git
cd devops-gitops-flux

# Explore the structure
ls -la
```

## 🎛️ Step 4: Deploy FluxCD Components

```bash
# Install Flux controllers
flux install

# Apply the GitRepository source
kubectl apply -f manifests/flux/gitrepository.yaml

# Apply the Kustomization
kubectl apply -f manifests/flux/kustomization-k8s.yaml

# Optional: Apply image automation
kubectl apply -f manifests/flux/image-automation/
```

## ✅ Step 5: Verify Deployment

```bash
# Check Flux status
flux check

# View Flux resources
flux get all

# Check the application
kubectl get all -n rocket-lab
```

You should see:
```
NAME                                    READY   STATUS    RESTARTS   AGE
pod/devops-sample-app-xxxxxxxxx-xxxxx   1/1     Running   0          2m
pod/devops-sample-app-xxxxxxxxx-xxxxx   1/1     Running   0          2m

NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/devops-sample-svc   ClusterIP   10.x.x.x       <none>        80/TCP    2m

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/devops-sample-app   2/2     2            2           2m
```

## 🌐 Step 6: Access the Application

```bash
# Port forward to access locally
kubectl port-forward svc/devops-sample-svc 8080:80 -n rocket-lab
```

Open your browser to `http://localhost:8080` and you should see the **Rocket Lab** welcome page! 🚀

## 🔄 Step 7: Test GitOps Workflow

```bash
# Make a change to the app
echo "Updated at $(date)" >> index.html

# Commit and push
git add index.html
git commit -m "Update welcome message"
git push origin main

# Watch Flux sync the changes (takes 1-5 minutes)
flux get kustomizations --watch
```

## 🎉 Success! What's Next?

### Explore More Features
- **Image Automation**: Automatic updates when new images are available
- **Multi-Environment**: Deploy to staging and production
- **Helm Charts**: Use the included Helm chart
- **Monitoring**: Set up alerts and dashboards

### Learn More
- Read the **[Complete FluxCD Guide](./FLUXCD_GUIDE.md)**
- Explore **[Configuration Analysis](./FLUX_ANALYSIS_REPORT.md)**
- Check out **[Troubleshooting Tips](./FLUX_TEST_RESULTS.md)**

### Customize for Your Needs
- Replace nginx with your own application
- Add your own Kubernetes manifests
- Configure CI/CD with GitHub Actions
- Set up multiple environments

## 🚨 Troubleshooting

### Pods Not Starting?
```bash
# Check pod status
kubectl describe pod -n rocket-lab -l app=devops-sample-app

# Check events
kubectl get events -n rocket-lab --sort-by='.lastTimestamp'
```

### Flux Not Syncing?
```bash
# Check Flux controllers
kubectl get pods -n flux-system

# Force reconciliation
flux reconcile source git devops-repo
flux reconcile kustomization rocket-lab-k8s
```

### Need Help?
- Check the **[troubleshooting guides](./README.md)** in the docs folder
- Open an issue on GitHub
- Review the comprehensive documentation

---

**Congratulations!** 🎊 You've successfully set up a GitOps workflow with FluxCD. Your application is now automatically synchronized with your Git repository, and any changes you make will be deployed automatically.

**Time to explore:** Try making changes to the Kubernetes manifests in the `k8s/` directory and watch FluxCD automatically apply them to your cluster!