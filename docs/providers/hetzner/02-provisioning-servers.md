# 02: Provisioning Bare Metal Servers

Now let's use the Hetzner provider to define another infrastructure product: a simple, bare-metal server.

This demonstrates the power of Crossplane to manage a heterogeneous environment. We can manage high-level PaaS services like AKS and low-level IaaS resources like bare-metal servers from the same, unified control plane.

## Defining the `CompositeBareMetalServer` XRD

First, we define the API for our new server product.

**File: `compositions/hetzner/server.yaml`**

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: CompositeResourceDefinition
metadata:
  name: compositebaremetalservers.server.example.org
spec:
  group: server.example.org
  names:
    kind: CompositeBareMetalServer
    plural: compositebaremetalservers
  claimNames:
    kind: BareMetalServer
    plural: baremetalservers
  versions:
  - name: v1alpha1
    served: true
    referenceable: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              location:
                type: string
                description: "The Hetzner location (e.g., nbg1, fsn1)."
              serverType:
                type: string
                description: "The server type (e.g., cx11, cpx21)."
              image:
                type: string
                description: "The OS image to install."
                default: "ubuntu-22.04"
            required:
              - location
              - serverType
```

This XRD allows developers to claim a `BareMetalServer` by specifying a `location`, `serverType`, and `image`.

## Composing the Hetzner Server

Next, the `Composition`. For this simple product, we only need to create a single Hetzner `Server` resource.

**File: `compositions/hetzner/server.yaml` (continued)**

```yaml
---
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: hetzner-bare-metal.v1alpha1.server.example.org
  labels:
    provider: hetzner
spec:
  compositeTypeRef:
    apiVersion: server.example.org/v1alpha1
    kind: CompositeBareMetalServer
  resources:
    - name: hetznerServer
      base:
        apiVersion: hcloud.crossplane.io/v1alpha1
        kind: Server
        spec:
          forProvider:
            location: "from-field-path(spec.location)"
            serverType: "from-field-path(spec.serverType)"
            image: "from-field-path(spec.image)"
      patches:
        - fromFieldPath: "status.atProvider.ipv4Address"
          toFieldPath: "status.connectionDetails.ipAddress"
        - fromFieldPath: "status.atProvider.rootPassword"
          toFieldPath: "status.connectionDetails.password"
```

### Key Concepts

-   **Simplicity:** This `Composition` is much simpler than the AKS one. It only creates a single resource.
-   **Connection Details:** We are patching the `ipv4Address` and `rootPassword` from the Hetzner `Server` resource into the `BareMetalServer` claim's connection details. This gives the developer who claimed the server immediate access to its IP address and root password.

## Exercise: Multi-Cloud Composition

This exercise will demonstrate the true power of Crossplane by creating a `Composition` that uses resources from multiple providers.

**Objective:** Create a `Composition` for a `CompositeWebApp` that provisions a Hetzner server for the application and an Azure Database for MySQL for its database.

**Tasks:**

1.  **Define the XRD:** Create a new `CompositeResourceDefinition` for a `CompositeWebApp` in a new file, `compositions/multi-cloud/webapp.yaml`. The claim, `WebApp`, should have fields for `hetznerLocation`, `serverType`, `azureRegion`, and `dbSku`.

2.  **Define the Composition:** Create the `Composition` in the same file.
    -   It should be composed of two resources:
        1.  A Hetzner `Server` (`hcloud.crossplane.io/v1alpha1`).
        2.  An Azure Database for MySQL `FlexibleServer` (`dbformysql.azure.upbound.io/v1beta1`).
    -   Use `from-field-path` patches to populate the resource specs from the `WebApp` claim.
    -   You will need to create a `ResourceGroup` for the Azure database as well.

3.  **Commit and Push:** Add the new `Composition` to your `platform` repository and push the changes.

4.  **Claim the WebApp:** In your `infra-dev` repository, create a `WebApp` claim.

    **Note on Database Passwords:** This `Composition` requires a secret named `mysql-password` in the `crossplane-system` namespace to exist beforehand. For this exercise, create it manually. In a production scenario, this secret would be managed by a tool like External Secrets Operator.

    ```bash
    kubectl create secret generic mysql-password -n crossplane-system --from-literal=password='MySuperSecretPa$w0rd'
    ```

5.  **Verify:** Use `crossplane trace` to see the resources being created in both Hetzner and Azure.

**Success Criteria:**

-   A new Hetzner server is provisioned.
-   A new Azure Database for MySQL is provisioned in a new Azure Resource Group.
-   The `WebApp` claim becomes `Ready`.

This exercise proves that Crossplane can be used to create a seamless, multi-cloud abstraction layer, allowing you to build platform products that leverage the best services from any provider.

**➡️ [Next Section: GKE Provider](../gke/01-setup.md)**
