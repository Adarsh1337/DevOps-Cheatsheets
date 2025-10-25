# 🫙 Podman Cheatsheet

[⬅️ Back to Root](../../README.md)

## Core Concepts
- Daemonless containers, rootless mode
- Images (OCI), containers, pods
- Compatibility with Docker CLI
- Volumes and networks

## Quick Commands
```bash
podman info
podman version

# Pull/run
podman pull alpine:3.19
podman run --rm -it alpine sh

# Build
podman build -t app:latest .

# List/inspect
podman ps -a
podman images
podman inspect app

# Exec/logs
podman logs -f app
podman exec -it app sh

# Stop/remove
podman stop app && podman rm app
```

## Volumes & Networks
```bash
podman volume create data
podman run -d -v data:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=pass --name db postgres:16

podman network create app-net
podman run -d --name api --network app-net app:latest
```

## Pods
```bash
podman pod create --name webstack -p 8080:80
podman run -d --pod webstack --name web nginx:alpine
podman run -d --pod webstack --name cache redis:7
podman pod ps
```

## Docker Compatibility
- podman-compose for docker-compose-like workflows
- alias docker=podman for CLI drop-in

## Best Practices
- Prefer rootless where possible
- Use OCI-compliant images
- Clean up regularly: podman system prune -a
