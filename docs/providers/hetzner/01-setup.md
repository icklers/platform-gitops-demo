# 01: Setting up the Hetzner Provider

Crossplane is not limited to the major cloud hyperscalers. The community has developed providers for a wide range of services, including the popular European cloud provider, Hetzner.

We will now configure the `provider-hetzner` to allow us to provision bare metal servers alongside our AKS clusters.

## 1. Installing the Provider

Just like with Azure, the first step is to install the Hetzner provider.

**File: `gitops-bootstrap/platform/provider-hetzner.yaml`**

```yaml
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-hetzner
spec:
  package: "xpkg.upbound.io/crossplane-contrib/provider-hetzner:v0.7.0"
```

This manifest is already part of our bootstrap process. The community-contributed Hetzner provider is installed and ready to go.

## 2. Configuring Authentication

Hetzner authentication is much simpler than Azure's. It uses a simple API token.

### 1. Create a Hetzner API Token

1.  Log in to your Hetzner Cloud Console.
2.  Create a new project.
3.  Navigate to **Security** -> **API tokens**.
4.  Generate a new **Read & Write** API token.
5.  **Copy this token immediately.** You will not be able to see it again.

### 2. Store the Hetzner Token in a Secret Store
In a production environment, you should never handle secrets manually. Following the pattern we established in the **Security** section, you should store your Hetzner API token in a secure vault (like Azure Key Vault, GCP Secret Manager, or HashiCorp Vault).

For this tutorial, we will assume you have stored the Hetzner API token in your chosen vault with the secret name `hetzner-api-token`.

### 3. Create the `ProviderConfig`
Now, we create a `ProviderConfig` that tells the Hetzner provider how to authenticate. This configuration will reference a Kubernetes `Secret` that will be created by the External Secrets Operator.

**File: `platform/provider-configs/hetzner.yaml`**
```yaml
apiVersion: meta.pkg.crossplane.io/v1alpha1
kind: ProviderConfig
metadata:
  name: hetzner-default
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: hetzner-credentials # This secret will be created by ESO
      key: token
```

Next, we define the `ExternalSecret` that will create the `hetzner-credentials` secret for us. This manifest should be stored in your `platform` repository.

**File: `platform/external-secrets/hetzner.yaml`**
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: hetzner-provider-token
  namespace: crossplane-system
spec:
  refreshInterval: "1h"
  secretStoreRef:
    name: azure-key-vault # Or your chosen secret store
    kind: ClusterSecretStore
  target:
    name: hetzner-credentials
    template:
      data:
        token: "{{ .hetzner-api-token }}" # Fetches from vault
  data:
  - secretKey: "hetzner-api-token"
    remoteRef:
      key: "hetzner-api-token"
```

Once these manifests are synced by ArgoCD, the Hetzner provider will be ready to use.

Check the status:

```bash
kubectl get providerconfig.meta.pkg.crossplane.io hetzner-default -o yaml
```

Look for the `Ready` condition to be `True`.

With both Azure and Hetzner configured, our platform is now truly multi-cloud. We can create `Compositions` that provision resources across both providers, or allow developers to choose their preferred provider when they make a claim.

**➡️ [Next: Provisioning Bare Metal Servers](./02-provisioning-servers.md)**
