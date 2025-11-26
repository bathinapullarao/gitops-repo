# GitOps Repository Structure and Workflow

```
gitops-repo/
│
├── README.md
│
├── manifests/
│   ├── namespace.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── hpa.yaml
│
└── argocd/
    └── application.yaml
```

---

## 🚀 GitOps Flow
1. Developer commits changes to the repository
2. Argo CD automatically detects updates
3. Argo CD syncs the Kubernetes cluster to match Git
4. Any manual change made in the cluster is auto-corrected

This ensures that Kubernetes **always matches what is stored in Git**.

---

## ✅ 1. Prerequisites
You must have:
- ✔ A Kubernetes Cluster (AKS / GKE / EKS / Minikube / K3s)
- ✔ A Git Repository (GitHub / GitLab / Bitbucket)

Your repo will contain:
```
/manifests
   deployment.yaml
   service.yaml
   ingress.yaml
   configmap.yaml
   hpa.yaml
```

Choose one GitOps tool:
- **Argo CD** 🔥 (recommended)
- Flux CD

Below is the **Argo CD GitOps setup**, widely used across the industry.

---

## 🚀 2. Install Argo CD in Your Cluster
### Install Argo CD
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Expose Argo CD UI
NodePort for local, LoadBalancer for cloud:
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Now open:
```
https://localhost:8080
```

---

## 🔐 3. Get Argo CD Admin Password
```
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
```

---

## 🏗 4. Push Your Kubernetes Manifests to Git
Your project repo structure:
```
gitops-demo/
 ├── deployment.yaml
 ├── service.yaml
 ├── ingress.yaml
 ├── hpa.yaml
 ├── configmap.yaml
 └── namespace.yaml
```

---

## 🧠 5. Create Argo CD Application (GitOps)
Argo CD will watch your Git repo and auto-sync changes to the cluster.

### Create `argo-application.yaml`:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-gitops
  namespace: argocd
spec:
  project: default

  source:
    repoURL: "https://github.com/YOUR-USER/YOUR-REPO.git"
    targetRevision: main
    path: manifests

  destination:
    server: https://kubernetes.default.svc
    namespace: prod

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

### Apply it:
```
kubectl apply -f argo-application.yaml
```

---

## 🎉 6. Argo CD Starts Deploying Automatically
Once committed to Git:
- Deployment changes → **auto-applied**
- ConfigMap changes → **auto-applied**
- Image updates → **auto-synced**
- Old resources → **auto-pruned**

View deployments in the **Argo CD UI**.

---

## 🔁 7. Continuous GitOps Loop
What happens next?
1. You update YAML and push to Git
2. Argo detects changes
3. Kubernetes syncs automatically
4. Manual edits in the cluster get auto-corrected

This ensures **100% Git = Kubernetes**.

---

## ⚙ Workflow Summary
### Developer Makes Changes
```
git commit → git push
```

### Argo CD Detects Updates
Argo checks your repo every few seconds.

### Argo CD Applies YAML Automatically
Kubernetes stays in sync with Git.

---

## 🎯 Final Result
You now have a complete **GitOps-enabled Kubernetes deployment** using Argo CD.

Everything is:
- Automated
- Declarative
- Self-healing
- Version-controlled

🚀 Your cluster always matches your Git repository!