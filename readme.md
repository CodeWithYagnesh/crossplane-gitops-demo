
# 🚀 GitOps Infrastructure Repo

![GitOps Workflow Diagram](./images/gitops-flow.svg)

This repository is the **Source of Truth** for our application infrastructure. It is monitored by **ArgoCD**, which continuously syncs these manifests to the Kubernetes cluster.

## 🔄 The GitOps Flow
1. **Developer** commits a `SecureBucket` manifest to this repo.
2. **ArgoCD** syncs the change to the cluster.
3. **Crossplane** (installed on the cluster) detects the new claim and provisions the actual AWS infrastructure defined in the Platform repo.

## 👩‍💻 Usage (For Developers)
To request a new compliant S3 bucket, simply add a file like `secureBucket.yaml`:

```yaml
apiVersion: yagnesh.cloud/v1alpha1
kind: SecureBucket
metadata:
  name: postgre-bucket
  namespace: default
spec:
  parameters:
    region: "us-east-2"
    tags:
      Project: "Portfolio-Demo"
      Owner: "DevOps-Team"

```

## 📂 Repository Contents

* **secureBucket.yaml**: A production-grade bucket request.
* **bucket.yaml**: Legacy/Test configurations.

## ✅ Verification

Once synced, the platform returns the status:

```bash
kubectl get securebucket
# NAME             READY   CONNECTION-SECRET   AGE
# postgre-bucket   True    postgre-bucket      2m
```

---

### 3. The Final Piece: ArgoCD Application
To make this a true "GitOps" demo, you need to tell ArgoCD to watch your second repo.

Create a file named `argocd-app.yaml` and apply it to your cluster.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: infrastructure-stack
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/codewithyagnesh/crossplane-gitops-demo.git'
    targetRevision: main
    path: .
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```
