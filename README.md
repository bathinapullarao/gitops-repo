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
### Install Argo CD CLI <--not mandatory
```
#sudo curl -sSL -o /usr/local/bin/argocd \
  https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
#sudo chmod +x /usr/local/bin/argocd
#argocd version
#argocd login localhost:8080 --username admin --password <your-password> --insecure
```
### Expose Argo CD UI
NodePort for local, LoadBalancer for cloud:
```
#kubectl port-forward svc/argocd-server -n argocd 8080:443  <--when you ctl+d app will stop
nohup kubectl port-forward svc/argocd-server -n argocd 8080:443 > argocd.log 2>&1 &   <--argocd app will run in background
#ps aux | grep port-forward     <--get the processid of argocd
#kill <PID>                     <--stop the argocd application
#cat argocd.log




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
#argocd app history myapp
#argocd app rollback myapp --revision 2
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

Here is the exact, simplest, and correct way to configure ArgoCD to deploy into GKE + AKS + EKS clusters from one ArgoCD server.
This is called ArgoCD Multi-Cluster GitOps.
🟢 STEP 1 - Get kubeconfig contexts for All Clusters
List contexts:
```
kubectl config get-contexts
You will see:
* argocd-cluster
  gke-cluster
  aks-cluster
  eks-cluster
```
🟢 STEP 2 — Login to ArgoCD CLI (from your laptop)
```
argocd login <ARGOCD-SERVER-IP> --username admin --password <password>
If using port-forward:
argocd login localhost:8080
```
🟢 STEP 3 — Register All External Clusters to ArgoCD
```
1️⃣ Add GKE
argocd cluster add gke-cluster
2️⃣ Add AKS
argocd cluster add aks-cluster
3️⃣ Add EKS
argocd cluster add eks-cluster
```
This will:
✔ Upload credentials
✔ Upload CA bundle
✔ Upload token
✔ Upload API server URL

to ArgoCD.

🟢 STEP 4 — Verify All Clusters Are Added
```
argocd cluster list
Expected output:
SERVER                               NAME
https://XX.XX.gke.com                gke-cluster
https://YY.YY.aks.com                aks-cluster
https://ZZ.ZZ.eks.amazonaws.com      eks-cluster
```

🟢 STEP 5 — Create 3 ArgoCD Applications (1 for each cluster)
📌 Example for GKE
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-gke
  namespace: argocd
spec:
  source:
    repoURL: "https://github.com/bathinapullarao/gitops-repo.git"
    path: manifests
    targetRevision: main
  destination:
    server: "https://XX.XX.gke.com"
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```
📌 Example for AKS
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-aks
  namespace: argocd
spec:
  source:
    repoURL: "https://github.com/bathinapullarao/gitops-repo.git"
    path: manifests
    targetRevision: main
  destination:
    server: "https://YY.YY.aks.com"
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```
📌 Example for EKS
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-eks
  namespace: argocd
spec:
  source:
    repoURL: "https://github.com/bathinapullarao/gitops-repo.git"
    path: manifests
    targetRevision: main
  destination:
    server: "https://ZZ.ZZ.eks.amazonaws.com"
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```
🟢 STEP 6 — Push Git Changes → All 3 Kubernetes Clusters Get Updated
ArgoCD: Polls your Git repo every 3 minutes
Sees changes
Applies them to all registered clusters (GKE, AKS, EKS)
Shows each cluster status in ArgoCD UI

🟢 Final Summary
Component	What to Do
ArgoCD	Install in 1 cluster only
GKE / AKS / EKS	Register via argocd cluster add
Deploy apps	Create 3 Application manifests (1 per cluster)
Sync	ArgoCD automatically applies changes when Git repo updates
