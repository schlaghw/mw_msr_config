# MSR GitOps Configuration Repository

This directory contains the GitOps configuration for deploying IBM webMethods Microservices Runtime (MSR) to multiple environments using ArgoCD ApplicationSets.

## Repository Structure

```
config-repo/
├── applicationset.yaml          # ArgoCD ApplicationSet manifest
├── deploy/                      # Environment-specific configurations
│   ├── dev/
│   │   ├── values-dev.yaml      # Helm values for development
│   │   └── application.properties  # MSR config for development
│   ├── test/
│   │   ├── values-test.yaml     # Helm values for test
│   │   └── application.properties  # MSR config for test
│   └── prod/
│       ├── values-prod.yaml     # Helm values for production
│       └── application.properties  # MSR config for production
└── README.md                    # This file
```

## How It Works

### ApplicationSet with Matrix Generator

The `applicationset.yaml` uses a **Matrix generator** that combines:

1. **Git Directory Generator**: Automatically discovers environment folders in `deploy/`
2. **List Generator**: Provides environment-specific metadata (namespace, ports, etc.)

This generates three ArgoCD Applications:
- `msr-dev` → deploys to `msr-dev` namespace
- `msr-test` → deploys to `msr-test` namespace  
- `msr-prod` → deploys to `msr-prod` namespace

### Environment Differences

| Environment | Replicas | Memory | CPU | NodePort HTTP | NodePort HTTPS |
|-------------|----------|--------|-----|---------------|----------------|
| **dev**     | 1        | 512Mi  | 250m| 30555         | 30543          |
| **test**    | 2        | 1Gi    | 500m| 30655         | 30643          |
| **prod**    | 3        | 2Gi    | 1000m| 30755        | 30743          |

### Configuration Files

Each environment has two configuration files:

1. **values-{env}.yaml**: Helm chart values (replicas, resources, ports, etc.)
2. **application.properties**: MSR runtime configuration (thread pools, logging, timeouts)

## Usage

### View Generated Applications

After the ApplicationSet is applied (from the ArgoCD management repo):

```bash
# List all MSR applications
argocd app list | grep msr

# Get details for a specific environment
argocd app get msr-dev
argocd app get msr-test
argocd app get msr-prod
```

### Trigger a Change

Edit any values file and commit:

```bash
# Example: Scale dev environment
vim deploy/dev/values-dev.yaml
# Change: replicaCount: 1 → replicaCount: 2

git add deploy/dev/values-dev.yaml
git commit -m "scale dev to 2 replicas"
git push

# ArgoCD will detect the change and sync automatically
```

### Access MSR Instances

```bash
# Development
curl http://localhost:30555/health

# Test
curl http://localhost:30655/health

# Production
curl http://localhost:30755/health
```

## GitOps Workflow

1. **Developer** makes changes to values or config files
2. **Git** stores the desired state
3. **ArgoCD** detects changes (polls every 3 minutes)
4. **ApplicationSet** generates/updates Applications
5. **Kubernetes** reconciles to match desired state

## Benefits

- ✅ **Single Source of Truth**: All configuration in Git
- ✅ **Environment Parity**: Same deployment process for all environments
- ✅ **Automated Sync**: Changes automatically deployed
- ✅ **Self-Healing**: Manual cluster changes are reverted
- ✅ **Audit Trail**: Git history shows who changed what and when

## Made with Bob