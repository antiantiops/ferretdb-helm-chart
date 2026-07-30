# FerretDB Helm Chart

A Helm chart for [FerretDB](https://www.ferretdb.com/) — a truly open-source MongoDB-compatible database backed by PostgreSQL.

## Installation

```bash
helm repo add ferretdb https://antiantiops.github.io/ferretdb-helm-chart/
helm repo update
helm install ferretdb ferretdb/ferretdb \
  --set ferretdb.postgresqlUrl="postgres://user:pass@host:5432/db"
```

## Prerequisites

- Kubernetes 1.21+
- Helm 3.x
- A running PostgreSQL instance (or [Crunchy PGO](https://access.crunchydata.com/documentation/postgres-operator/latest/), etc.)
- *(Optional)* [External Secrets Operator](https://external-secrets.io) v0.17+ if using `externalSecret`

## Configuration

### PostgreSQL Connection

There are three ways to provide the PostgreSQL URL, in order of priority:

#### 1. External Secrets Operator (recommended for production)

Pull the connection URL from an external secret store (AWS SSM, Vault, GCP Secret Manager, Azure Key Vault, etc.):

```yaml
externalSecret:
  enabled: true
  refreshInterval: "1h"
  secretStoreRef:
    name: my-secret-store
    kind: ClusterSecretStore
  remoteRef:
    key: /production/ferretdb/postgresql-url
```

#### 2. Existing Kubernetes Secret

Reference a pre-existing Secret in the same namespace:

```yaml
existingSecret:
  enabled: true
  name: my-pg-secret
  key: uri
```

#### 3. Inline URL (simplest, not recommended for production)

```yaml
ferretdb:
  postgresqlUrl: "postgres://user:pass@host:5432/ferretdb?sslmode=disable"
```

> **Priority:** `externalSecret` > `existingSecret` > `ferretdb.postgresqlUrl`

## Values Reference

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `replicaCount` | int | `1` | Number of replicas |
| `image.repository` | string | `ghcr.io/ferretdb/ferretdb` | Image repository |
| `image.pullPolicy` | string | `IfNotPresent` | Image pull policy |
| `image.tag` | string | `""` (appVersion) | Image tag override |
| `imagePullSecrets` | list | `[]` | Image pull secrets |
| `nameOverride` | string | `""` | Override chart name |
| `fullnameOverride` | string | `""` | Override full release name |
| **FerretDB** | | | |
| `ferretdb.postgresqlUrl` | string | `""` | PostgreSQL connection URL |
| `ferretdb.auth` | bool | `true` | Enable MongoDB authentication |
| `ferretdb.extraEnv` | list | `[]` | Additional environment variables |
| **Existing Secret** | | | |
| `existingSecret.enabled` | bool | `false` | Use an existing K8s Secret |
| `existingSecret.name` | string | `""` | Secret name |
| `existingSecret.key` | string | `"uri"` | Key within the Secret |
| **External Secret** | | | |
| `externalSecret.enabled` | bool | `false` | Create an ExternalSecret resource |
| `externalSecret.annotations` | object | `{}` | Annotations for the ExternalSecret |
| `externalSecret.refreshInterval` | string | `"1h"` | Refresh interval from remote store |
| `externalSecret.secretStoreRef.name` | string | `""` | SecretStore or ClusterSecretStore name |
| `externalSecret.secretStoreRef.kind` | string | `"SecretStore"` | `SecretStore` or `ClusterSecretStore` |
| `externalSecret.targetSecretName` | string | `<fullname>-pg-credentials` | Name of the created K8s Secret |
| `externalSecret.creationPolicy` | string | `"Owner"` | Secret creation policy |
| `externalSecret.deletionPolicy` | string | `"Retain"` | Secret deletion policy |
| `externalSecret.secretKey` | string | `"uri"` | Key in the created K8s Secret |
| `externalSecret.remoteRef.key` | string | `""` | Path/key in the external store |
| `externalSecret.remoteRef.property` | string | `""` | JSON property (optional) |
| `externalSecret.remoteRef.version` | string | `""` | Secret version (optional) |
| **Service Account** | | | |
| `serviceAccount.create` | bool | `true` | Create a ServiceAccount |
| `serviceAccount.automount` | bool | `true` | Automount API credentials |
| `serviceAccount.annotations` | object | `{}` | ServiceAccount annotations |
| `serviceAccount.name` | string | `""` | ServiceAccount name override |
| **Pod** | | | |
| `podAnnotations` | object | `{}` | Pod annotations |
| `podLabels` | object | `{}` | Additional pod labels |
| **Service** | | | |
| `service.type` | string | `"ClusterIP"` | Service type |
| `service.port` | int | `27017` | Service port |
| **Resources** | | | |
| `resources` | object | `{}` | CPU/memory resource requests and limits |
| `nodeSelector` | object | `{}` | Node selector |
| `tolerations` | list | `[]` | Tolerations |
| `affinity` | object | `{}` | Affinity rules |

## Connecting

Once deployed, connect using `mongosh` or any MongoDB-compatible client:

```bash
mongosh "mongodb://ferretdb.ferretdb.svc:27017/"
```

If authentication is enabled (`ferretdb.auth: true`), use your PostgreSQL-backed credentials:

```bash
mongosh "mongodb://user:pass@ferretdb.ferretdb.svc:27017/ferretdb?authMechanism=PLAIN"
```

## Upgrading

### From 0.1.x to 0.2.x

- Added External Secrets Operator support (`externalSecret.*`)
- All new fields default to disabled — no breaking changes
- ExternalSecret uses `external-secrets.io/v1` API (requires ESO v0.17+)
