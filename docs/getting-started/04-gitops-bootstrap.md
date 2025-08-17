# 04: GitOps Bootstrap with ArgoCD

Now that we have a blank Kubernetes cluster, we will kick off our entire platform using a single command. This is the essence of a true GitOps workflow.

## The GitOps Bootstrap Process

We will apply a single file to our cluster: `gitops-bootstrap/argocd/install.yaml`. This manifest contains the Custom Resource Definitions for ArgoCD.

Immediately after, we will apply our `app-of-apps.yaml`. This special `Application` resource instructs ArgoCD to manage its own configuration and all other platform components from our Git repository.

### 1. Fork and Clone with the GitHub CLI

This tutorial uses the official GitHub CLI, `gh`, to manage repository operations consistently. If you have not already, ensure you have authenticated with GitHub by running `gh auth login`.

This single command will create a fork of the tutorial repository under your username and clone it to your local machine.

```bash
# Replace <source-org> with the actual source organization or user
gh repo fork <source-org>/idp-tutorial --clone=true
cd idp-tutorial
```

### 2. Apply the Bootstrap Manifests

This is the **only manual `kubectl apply`** we will perform. These commands tell the cluster to install a minimal ArgoCD and then immediately hand over all control to Git.

```bash
# Apply the ArgoCD CRDs and initial setup
kubectl apply -f gitops-bootstrap/argocd/install.yaml

# Apply the App of Apps to start the GitOps sync
kubectl apply -f gitops-bootstrap/argocd/app-of-apps.yaml
```

### 4. Observe the Magic

ArgoCD is now installing and configuring itself, Crossplane, and all the providers and compositions defined in your Git repository. To watch this happen, access the ArgoCD UI.

First, get the initial admin password:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

Then, port-forward the ArgoCD server:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Log in to `https://localhost:8080` with username `admin` and the retrieved password. You will see the `app-of-apps` and all the other applications syncing automatically.

## Congratulations!

You have successfully bootstrapped a fully functional GitOps workflow. From this point forward, we will interact with our system exclusively through Git.

**➡️ [Next Section: Repository Management](../repository-management/01-multi-repo-strategy.md)**