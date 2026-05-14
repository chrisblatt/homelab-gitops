# homelab-gitops

This repository demonstrates a centralized **Argo CD Multi-Cluster** managementment using the **App of Apps** pattern and a bootstrapping **ApplicationSets**.

**Note:** This README is LLM generated because I really dislike documentation and writing succinctly. However this pattern comes from real world experience and has been deployed to manage 100+ production clusters at the edge.

## Architecture Overview

1.  **Bootstrap ApplicationSet**: Located in `bootstrapping/`, this watches for Kubernetes clusters joined to Argo CD with specific labels.
2.  **Root App**: Located in `apps/root/`, this is a Helm chart that defines which applications (like `test-app`) should be deployed to a cluster.

## Getting Started

### 1. Deploy the Bootstrap AppSet
Apply the bootstrap manifest to your management cluster:

```bash
kubectl apply -f bootstrapping/infra-bootstrap-app-set.yaml
```

### 2. Register a Managed Cluster
Ensure your target cluster is added to Argo CD. To be picked up by the automation, it must have the label `env=managed`.

```bash
argocd cluster add <context-name> --label env=managed
```

Argo CD will now automatically generate a `root` application for every labeled cluster.

## Operations

### Adding a New Cluster
The directory structure follows a "Global vs. Local" override pattern. To add a cluster named `prod-west`:

1.  Create a directory: `clusters/prod-west/`.
2.  Create an `overrides.yaml` in that directory.
3.  Label the cluster in Argo CD as `env: managed`.

The ApplicationSet will automatically find these files via the `$values` reference.

### Adding a New Application
To add a new service (e.g., `monitoring`):

1.  **Create the Chart**: Add your Helm chart or manifest folder to `apps/monitoring`.
2.  **Update the Root App**: Add a template for the new Application in `apps/root/templates/`. This template should point to `apps/monitoring`.
3.  **Define Defaults**: Add default values for the new app in `clusters/global/overrides.yaml`.
4.  **Cluster Overrides**: If a specific cluster needs a different version or config for `monitoring`, add those values to `clusters/<cluster-name>/overrides.yaml`.

### Directory Structure

```text
.
├── apps
│   ├── root           # The "App of Apps" logic
│   └── test-app       # A sample application
├── bootstrapping      # ApplicationSet definitions
└── clusters
    ├── global         # Values applied to ALL clusters
    └── <cluster-name> # Values applied to specific clusters
```

## Key Concepts

*   **Separation of Concerns**: `apps/` contains "how" to deploy, `clusters/` contains "what" (configuration) to deploy.
