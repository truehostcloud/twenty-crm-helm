# Twenty CRM Helm Chart

Deploy Twenty CRM on Kubernetes with a standalone Helm chart modeled on the upstream community chart and packaged for this repository.

## Chart Layout

- `charts/twenty`: the Helm chart
- `.github/workflows/helm-release.yaml`: lint, test, package, and publish workflow

## Install

Internal PostgreSQL and Redis:

```bash
helm install my-twenty ./charts/twenty \
	--namespace twentycrm \
	--create-namespace
```

External PostgreSQL and Redis:

```bash
helm install my-twenty ./charts/twenty \
	--namespace twentycrm \
	--create-namespace \
	--set db.enabled=false \
	--set db.external.host=db.example.com \
	--set db.external.user=twenty_app_user \
	--set db.external.password=change-me \
	--set redisInternal.enabled=false \
	--set redis.external.host=redis.example.com
```

Ingress example:

```bash
export DOMAIN=crm.example.com

helm install my-twenty ./charts/twenty \
	--namespace twentycrm \
	--create-namespace \
	--wait \
	--set "server.ingress.hosts[0].host=$DOMAIN" \
	--set "server.ingress.tls[0].hosts[0]=$DOMAIN"
```

## Release Workflow

Pushes to `main` that touch `charts/twenty` run:

1. `helm lint`
2. `helm template`
3. `helm unittest`
4. `helm/chart-releaser-action` to publish packaged chart artifacts and update the repository index on `gh-pages`

Enable GitHub Pages for the repository and serve from the `gh-pages` branch so the generated chart repository index is reachable.

## Notes

- Internal or external PostgreSQL is selected with `db.enabled`.
- Internal or external Redis is selected with `redisInternal.enabled`.
- The chart auto-generates the Twenty app secret if one is not provided.
- S3-backed file storage is supported through `storage.type=s3`.

See `charts/twenty/README.md` and `charts/twenty/QUICKSTART.md` for chart-specific details.