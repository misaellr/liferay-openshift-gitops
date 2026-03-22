# Liferay OpenShift GitOps

ArgoCD values repository for deploying Liferay DXP on OpenShift. This repo is watched by an ArgoCD ApplicationSet that auto-discovers environments and syncs them to the cluster.

## How It Works

```
Push liferay.yaml → GitHub → ArgoCD detects change → Syncs to OpenShift cluster
```

ArgoCD combines the Liferay Helm chart (from [OCI registry](https://us-central1-docker.pkg.dev/liferay-artifact-registry/liferay-helm-chart/liferay-default)) with the values files in this repo using a multi-source Application pattern.

## Repository Structure

```
liferay/projects/default/
├── base/
│   └── liferay.yaml          # Shared config: probes, init containers, clustering
└── environments/
    ├── dev/liferay.yaml       # Dev: low resources, single replica
    ├── uat/liferay.yaml       # UAT: medium resources
    └── prd/liferay.yaml       # PRD: production resources, 2 replicas
```

- **base/liferay.yaml**: Configuration shared across all environments (probes, init containers, OpenSearch module download)
- **environments/\*/liferay.yaml**: Environment-specific overrides (image tag, resources, SCC config, credentials, OpenSearch endpoint)

## Values Layering

ArgoCD applies values in order:
1. Chart defaults (from OCI artifact)
2. `base/liferay.yaml` (shared)
3. `environments/<env>/liferay.yaml` (environment-specific, wins on conflicts)

All values use **flat top-level keys** (not nested under `liferay-default:`). The chart is deployed directly from OCI, not as a subchart.

## Adding an Environment

1. Create `liferay/projects/default/environments/<new-env>/liferay.yaml`
2. ArgoCD auto-discovers it via the Git generator
3. A new ArgoCD Application appears and syncs

## Prerequisites on the Cluster

Before ArgoCD can deploy Liferay to a namespace:

```bash
# Create namespace
oc new-project liferay-<env>

# Grant SCC
oc adm policy add-scc-to-user nonroot-v2 -z liferay-default -n liferay-<env>

# Label for ArgoCD access
oc label namespace liferay-<env> argocd.argoproj.io/managed-by=openshift-gitops

# Deploy data layer (PostgreSQL + OpenSearch) before Liferay
```

## Related

- [liferay-on-openshift](https://github.com/misaellr/liferay-on-openshift): Reference configs, operators, scripts, docs
- [Liferay Cloud Native Experience](https://learn.liferay.com/w/dxp/self-hosted-installation-and-upgrades/cloud-native-experience): Official documentation
