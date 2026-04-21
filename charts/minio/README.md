# MinIO Helm Chart (Mirrored)

> **Note:** This is a mirror of the archived [MinIO community Helm chart](https://github.com/minio/minio) (v5.4.0).
> The original chart is no longer maintained. Image references have been updated to use a private registry.

## Chart Details

| Field | Value |
|-------|-------|
| Chart Version | 5.4.0 |
| App Version | RELEASE.2024-12-18T13-15-44Z |
| Images | `registry-coderepo.stackforgesolutions.com/stackforgesolutions/minio` |
| | `registry-coderepo.stackforgesolutions.com/stackforgesolutions/mc` |

## Prerequisites

- Kubernetes v1.19+
- Helm 3+
- PV provisioner support in the underlying infrastructure

## Installation

### From OCI Registry

```bash
helm install minio oci://registry-coderepo.stackforgesolutions.com/stackforgesolutions/minio \
  --version 5.4.0 \
  --namespace minio \
  --create-namespace \
  --set rootUser=rootuser,rootPassword=rootpass123
```

### From Git Repository

```bash
git clone git@coderepo-ssh.stackforgesolutions.com:stackforgesolutions/minio.git
helm install minio ./minio/helm/minio \
  --namespace minio \
  --create-namespace \
  --set rootUser=rootuser,rootPassword=rootpass123
```

### Minimal Test Setup

```bash
helm install minio oci://registry-coderepo.stackforgesolutions.com/stackforgesolutions/minio \
  --version 5.4.0 \
  --set resources.requests.memory=512Mi \
  --set replicas=1 \
  --set persistence.enabled=false \
  --set mode=standalone \
  --set rootUser=rootuser,rootPassword=rootpass123
```

### ArgoCD

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: minio
  namespace: argocd
spec:
  project: default
  destination:
    server: https://kubernetes.default.svc
    namespace: minio
  source:
    chart: stackforgesolutions/minio
    repoURL: registry-coderepo.stackforgesolutions.com
    targetRevision: 5.4.0
    helm:
      valueFiles:
        - values.yaml
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

## Upgrading

```bash
helm get values my-release > old_values.yaml
# Edit old_values.yaml as needed
helm upgrade -f old_values.yaml my-release oci://registry-coderepo.stackforgesolutions.com/stackforgesolutions/minio --version 5.4.0
```

## Configuration

Refer to the [values.yaml](./values.yaml) file for all configurable parameters.

You can specify parameters using `--set key=value[,key=value]` or provide a YAML values file with `-f values.yaml`.

### Key Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.repository` | MinIO image | `registry-coderepo.stackforgesolutions.com/stackforgesolutions/minio` |
| `image.tag` | MinIO image tag | `RELEASE.2024-12-18T13-15-44Z` |
| `mcImage.repository` | MinIO Client image | `registry-coderepo.stackforgesolutions.com/stackforgesolutions/mc` |
| `mode` | MinIO mode (`standalone` or `distributed`) | `distributed` |
| `replicas` | Number of MinIO containers | `16` |
| `persistence.enabled` | Enable persistence | `true` |
| `persistence.size` | PVC size | `500Gi` |
| `rootUser` | Root user name | (generated) |
| `rootPassword` | Root password | (generated) |
| `existingSecret` | Use existing secret for credentials | `""` |

### Persistence

This chart provisions a PersistentVolumeClaim and mounts it to `/export`. To use `emptyDir` instead:

```bash
helm install --set persistence.enabled=false ...
```

To use an existing PVC:

```bash
helm install --set persistence.existingClaim=PVC_NAME ...
```

### TLS

```bash
kubectl create secret generic tls-ssl-minio --from-file=path/to/private.key --from-file=path/to/public.crt
helm install --set tls.enabled=true,tls.certSecret=tls-ssl-minio ...
```

### Existing Secret

```bash
kubectl create secret generic my-minio-secret --from-literal=rootUser=foobarbaz --from-literal=rootPassword=foobarbazqux
helm install --set existingSecret=my-minio-secret ...
```

### Buckets

Create buckets after install:

```bash
helm install --set buckets[0].name=bucket1,buckets[0].policy=none,buckets[0].purge=false ...
```

### NetworkPolicy

Enable by setting `networkPolicy.enabled=true`. When using Cilium as CNI, set `networkPolicy.flavor=cilium`.

## Uninstalling

```bash
helm uninstall my-release
```

## Mirror Images

To mirror the required Docker images to the private registry:

```bash
# Pull
docker pull quay.io/minio/minio:RELEASE.2024-12-18T13-15-44Z
docker pull quay.io/minio/mc:RELEASE.2024-11-21T17-21-54Z

# Tag
docker tag quay.io/minio/minio:RELEASE.2024-12-18T13-15-44Z registry-coderepo.stackforgesolutions.com/stackforgesolutions/minio:RELEASE.2024-12-18T13-15-44Z
docker tag quay.io/minio/mc:RELEASE.2024-11-21T17-21-54Z registry-coderepo.stackforgesolutions.com/stackforgesolutions/mc:RELEASE.2024-11-21T17-21-54Z

# Push
docker push registry-coderepo.stackforgesolutions.com/stackforgesolutions/minio:RELEASE.2024-12-18T13-15-44Z
docker push registry-coderepo.stackforgesolutions.com/stackforgesolutions/mc:RELEASE.2024-11-21T17-21-54Z
```
