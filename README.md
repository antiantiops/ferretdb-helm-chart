# FerretDB Helm Chart

[![Lint and Test](https://github.com/antiantiops/ferretdb-helm-chart/actions/workflows/ci.yaml/badge.svg)](https://github.com/antiantiops/ferretdb-helm-chart/actions/workflows/ci.yaml)
[![Release](https://github.com/antiantiops/ferretdb-helm-chart/actions/workflows/release-chart.yaml/badge.svg)](https://github.com/antiantiops/ferretdb-helm-chart/actions/workflows/release-chart.yaml)

A Helm chart for [FerretDB](https://www.ferretdb.com/) — MongoDB-compatible database backed by PostgreSQL.

## Usage

```bash
helm repo add ferretdb https://antiantiops.github.io/ferretdb-helm-chart/
helm repo update
helm install ferretdb ferretdb/ferretdb --set ferretdb.postgresqlUrl="postgres://user:pass@host:5432/db"
```

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| replicaCount | int | `1` | Number of replicas |
| image.repository | string | `ghcr.io/ferretdb/ferretdb` | Image repository |
| image.tag | string | `""` (appVersion) | Image tag |
| ferretdb.postgresqlUrl | string | `""` | PostgreSQL connection URL |
| ferretdb.auth | bool | `true` | Enable authentication |
| ferretdb.extraEnv | list | `[]` | Additional environment variables |
| existingSecret.enabled | bool | `false` | Use existing secret for PostgreSQL URL |
| existingSecret.name | string | `""` | Secret name |
| existingSecret.key | string | `uri` | Secret key |
| externalSecret.enabled | bool | `false` | Create an ExternalSecret (requires [External Secrets Operator](https://external-secrets.io)) |
| externalSecret.annotations | object | `{}` | Annotations applied to the ExternalSecret |
| externalSecret.refreshInterval | string | `1h` | How often to refresh from the remote store |
| externalSecret.secretStoreRef.name | string | `""` | SecretStore or ClusterSecretStore name |
| externalSecret.secretStoreRef.kind | string | `SecretStore` | `SecretStore` or `ClusterSecretStore` |
| externalSecret.targetSecretName | string | `<fullname>-pg-credentials` | Name of the K8s Secret created by ExternalSecret |
| externalSecret.creationPolicy | string | `Owner` | Secret creation policy |
| externalSecret.deletionPolicy | string | `Retain` | Secret deletion policy |
| externalSecret.secretKey | string | `uri` | Key name in the created K8s Secret |
| externalSecret.remoteRef.key | string | `""` | Path/key in the external secret store |
| externalSecret.remoteRef.property | string | `""` | JSON property within the secret (optional) |
| externalSecret.remoteRef.version | string | `""` | Secret version (optional) |
| service.type | string | `ClusterIP` | Service type |
| service.port | int | `27017` | Service port |

## External Secrets Operator

If you use [External Secrets Operator](https://external-secrets.io) to manage secrets, enable `externalSecret` to automatically create a Kubernetes Secret from your external store (AWS SSM, HashiCorp Vault, GCP Secret Manager, Azure Key Vault, etc.).

```yaml
ferretdb:
  auth: true

externalSecret:
  enabled: true
  refreshInterval: "1h"
  secretStoreRef:
    name: my-secret-store
    kind: ClusterSecretStore
  remoteRef:
    key: /production/ferretdb/postgresql-url
```

This creates an `ExternalSecret` that syncs the PostgreSQL connection URL into a Kubernetes Secret, which is then referenced by the FerretDB deployment. No need to put credentials in your values file.

> **Priority order:** `externalSecret` > `existingSecret` > `ferretdb.postgresqlUrl`

## Upstream Watch

A GitHub Actions workflow checks for new FerretDB releases every 6 hours and auto-bumps the chart.
