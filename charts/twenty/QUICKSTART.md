# Twenty Helm Chart - Quick Install

## Simple Install

Set your domain and install:

```bash
export DOMAIN=crm.example.com

helm install my-twenty ./charts/twenty \
  --namespace twentycrm --create-namespace --wait \
  --set "server.ingress.hosts[0].host=$DOMAIN" \
  --set "server.ingress.tls[0].hosts[0]=$DOMAIN"
```

That deploys:

- the Twenty server
- the Twenty worker
- internal PostgreSQL unless disabled
- internal Redis unless disabled
- a generated app secret token if one is not provided

## Access the App

Visit `https://$DOMAIN` and complete the signup flow.

## Retrieve the App Secret

```bash
kubectl get secret tokens -n twentycrm -o jsonpath='{.data.accessToken}' | base64 --decode && echo
```

## Advanced Configuration

See [README.md](README.md) for:

- external PostgreSQL and Redis
- S3 storage configuration
- resource tuning
- secret-based database credentials

## Uninstall

```bash
helm uninstall my-twenty -n twentycrm
kubectl delete namespace twentycrm
```