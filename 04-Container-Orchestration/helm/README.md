# ⛵ Helm Cheatsheet

[⬅️ Back to Root](../../README.md)

## Core Concepts
- Charts, templates, values
- Releases and revisions
- Repositories and indexes
- Overrides via --set and -f values files

## Repos
```bash
helm version
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx
```

## Install/Upgrade/Rollback
```bash
# Install
helm install web bitnami/nginx --namespace dev --create-namespace \
  --set service.type=ClusterIP

# Upgrade with values
helm upgrade web bitnami/nginx -n dev -f values.yaml --set image.tag=1.27

# History and rollback
helm history web -n dev
helm rollback web 1 -n dev
```

## Inspect & Template
```bash
helm show chart bitnami/nginx
helm show values bitnami/nginx | less
helm template web bitnami/nginx -n dev --debug > rendered.yaml
```

## Create Chart
```bash
helm create mychart
# Structure: Chart.yaml, values.yaml, templates/*
```

## Values Patterns
- Use values.yaml for defaults
- Use environment-specific files: values-dev.yaml, values-prod.yaml
- Override order: values.yaml < -f file.yaml < --set key=val

## Best Practices
- Pin chart and app versions
- Keep templates idempotent and minimal logic
- Use NOTES.txt for usage hints
- Store chart definitions in repo, enable CI linting
