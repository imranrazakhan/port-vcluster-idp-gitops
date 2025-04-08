# port-vcluster-idp-gitops

An Internal Developer Platform (IDP) proof-of-concept for self-service **vCluster** provisioning using [Port](https://www.getport.io/), GitHub Actions, and ArgoCD powered by GitOps.

## 🚀 Overview

This project demonstrates how platform teams can enable developers to provision virtual Kubernetes clusters (`vClusters`) through a self-service portal (Port) while using GitHub as the single source of truth, automated through GitHub Actions and deployed via ArgoCD and ApplicationSet.

### 🔁 Flow Diagram

1. Developer triggers a **Port action** to request a new vCluster.
2. The action initiates a **GitHub Action** workflow which:
   - Generates a vCluster manifest.
   - Commits it to the GitHub repo under `clusters/dev/`.
3. **ArgoCD ApplicationSet** watches this directory and automatically:
   - Detects the new manifest.
   - Deploys the vCluster to the target Kubernetes cluster.
4. Optionally, developers can manage their workloads inside vClusters.

---

## 📁 Folder Structure


---

## ⚙️ Tech Stack

| Component     | Purpose                                |
|---------------|----------------------------------------|
| **Port**      | IDP portal to trigger provisioning     |
| **GitHub**    | Source of truth for manifests          |
| **GitHub Actions** | Automation pipeline for manifest creation |
| **ArgoCD**    | GitOps-based deployment controller     |
| **ApplicationSet** | Dynamic app discovery for ArgoCD   |
| **vCluster**  | Lightweight virtual Kubernetes clusters |
| **Kubernetes**| Hosting the ArgoCD and vClusters       |

---

## ✅ Prerequisites

- A Kubernetes cluster (e.g., AKS, GKE, EKS, or local dev)
- ArgoCD installed and accessible
- [ApplicationSet](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/) controller installed
- [vCluster](https://www.vcluster.com/) CLI (optional for testing)
- GitHub repository with workflows enabled
- Port environment with GitHub integration

---

## 🚦 GitHub Workflow Example

```yaml
name: Port vCluster Deployment
on:
  workflow_dispatch:
    inputs:
      clusterName:
        description: 'Cluster name'
        required: true
      namespace:
        description: 'Target namespace'
        default: 'vcluster-ns'

jobs:
  generate-manifest:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Create vCluster manifest
        run: |
          mkdir -p clusters/dev/
          cat <<EOF > clusters/dev/${{ github.event.inputs.clusterName }}.yaml
          apiVersion: cluster.x-k8s.io/v1beta1
          kind: Cluster
          metadata:
            name: ${{ github.event.inputs.clusterName }}
            namespace: ${{ github.event.inputs.namespace }}
          spec:
            infrastructureRef:
              apiVersion: infrastructure.cluster.x-k8s.io/v1alpha1
              kind: VCluster
              name: ${{ github.event.inputs.clusterName }}
          EOF

      - name: Commit changes
        uses: stefanzweifel/git-auto-commit-action@v5
        with:
          commit_message: "Deploy vCluster ${{ github.event.inputs.clusterName }}"
          branch: main
