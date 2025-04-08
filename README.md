# vCluster GitOps Deployment via GitHub Actions & ArgoCD

This project enables automated provisioning of vClusters using GitHub Actions triggered by Port, and deployment via ArgoCD using `ApplicationSet`.

## 📦 Project Structure



## 🚀 How It Works

### 1. Trigger via Port

Port triggers the GitHub Actions workflow `deploy-vcluster.yaml` with inputs:
- `clusterName`: Name of the vCluster
- `namespace`: (optional) Namespace for vCluster (default is `vcluster-ns`)

### 2. GitHub Action

The action:
- Creates a vCluster manifest under `clusters/dev/`
- Commits and pushes the manifest to the `main` branch

### 3. ArgoCD + ApplicationSet

The `apps/applicationset.yaml` configures ArgoCD to watch the `clusters/dev/` directory. It auto-syncs and deploys any vCluster definitions found there.

## 📂 Example Generated vCluster Manifest

```yaml
apiVersion: cluster.x-k8s.io/v1beta1
kind: Cluster
metadata:
  name: my-vcluster
spec:
  infrastructureRef:
    apiVersion: infrastructure.cluster.x-k8s.io/v1alpha1
    kind: VCluster
    name: my-vcluster

Future Ideas
Add labels in Port for tracking vCluster status.

Setup automatic clean-up of old vClusters.

Add webhooks from ArgoCD for sync status back to Port.
