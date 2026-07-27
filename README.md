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
| service.type | string | `ClusterIP` | Service type |
| service.port | int | `27017` | Service port |

## Upstream Watch

A GitHub Actions workflow checks for new FerretDB releases every 6 hours and auto-bumps the chart.
