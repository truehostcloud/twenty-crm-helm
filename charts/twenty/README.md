# Twenty Helm Chart

Deploy Twenty CRM on Kubernetes with server, worker, PostgreSQL, and Redis components.

## Features

- Server and worker deployments with full env exposure via `values.yaml`
- Internal PostgreSQL (Spilo) and Redis deployments included
- External PostgreSQL and Redis support when you want to reuse managed services
- PVC-based persistence using dynamic storage classes
- Ingress with configurable annotations, hosts, and TLS
- Database readiness and migrations handled by init containers instead of separate jobs

## Quick Start

See [QUICKSTART.md](QUICKSTART.md) for a minimal install.

## Installing

Prerequisites: Kubernetes 1.21+, Helm 3.8+, and a default StorageClass if you use internal persistence.

Internal DB and Redis:

```bash
helm install my-twenty ./charts/twenty \
  --namespace twentycrm --create-namespace
```

External DB and Redis:

```bash
helm install my-twenty ./charts/twenty \
  --namespace twentycrm --create-namespace \
  --set db.enabled=false \
  --set db.external.host=db.example.com \
  --set redisInternal.enabled=false \
  --set redis.external.host=redis.example.com
```

External DB credentials from an existing secret:

```bash
helm install my-twenty ./charts/twenty \
  --namespace twentycrm --create-namespace \
  --set db.enabled=false \
  --set db.external.host=db.example.com \
  --set db.external.user=twenty_app_user \
  --set db.external.database=twenty \
  --set db.external.secretName=twenty-db \
  --set db.external.passwordKey=password
```

## Key Values

See `values.yaml` for the complete list. The main prerequisite toggles are:

- `db.enabled=true`: deploy internal PostgreSQL
- `db.enabled=false`: use `db.external.*`
- `redisInternal.enabled=true`: deploy internal Redis
- `redisInternal.enabled=false`: use `redis.external.*`

## Notes

- Database URL and Redis URL are composed automatically from chart settings
- Database `twenty` and schema `core` are created automatically by the server init container when using the internal DB
- The chart does not create separate DB setup or migration jobs
- Access token is auto-generated if not provided and re-used on upgrade when the secret already exists
- TLS is enabled by default through cert-manager annotations when `server.ingress.acme=true`

## Testing

```bash
helm lint ./charts/twenty
helm template my-twenty ./charts/twenty
helm plugin install https://github.com/quintush/helm-unittest
helm unittest ./charts/twenty
```

## Storage

Local storage is the default and uses PVCs.

S3-backed storage:

```bash
# values-secrets.yaml
# storage:
#   type: s3
#   s3:
#     bucket: my-bucket
#     region: us-east-1
#     accessKeyId: AKIA...
#     secretAccessKey: ...

helm install my-twenty ./charts/twenty -f values-secrets.yaml
```

You can also reference an existing secret through `storage.s3.secretName`, `storage.s3.accessKeyIdKey`, and `storage.s3.secretAccessKeyKey`.

## Publishing

This repository includes a GitHub Actions workflow that lints, tests, packages, and publishes the chart with Chart Releaser on pushes to `main`.