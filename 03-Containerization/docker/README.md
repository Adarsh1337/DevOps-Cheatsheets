# 🐳 Docker Cheatsheet

[⬅️ Back to Root](../../README.md)

## Core Concepts
- Images vs Containers
- Dockerfile and build context
- Registries (Docker Hub, GHCR)
- Volumes (named, bind, anonymous)
- Networks (bridge, host, none, overlay)
- Multi-stage builds, layers & cache

## Quick Start
```bash
# Verify installation
docker version
docker info

# Run an image
docker run --rm hello-world

# Search & pull
docker search alpine
docker pull alpine:3.19
```

## Image & Container Lifecycle
```bash
# Build
docker build -t app:latest .
docker buildx build --platform linux/amd64,linux/arm64 -t user/app:1.0 .

# List
docker images

# Tag & push
docker tag app:latest user/app:1.0
docker login
docker push user/app:1.0

# Run
docker run -d --name app -p 8080:80 app:latest

# Inspect logs/exec
docker logs -f app
docker exec -it app sh

# Stop/Start/Remove
docker stop app && docker rm app

# Prune
docker system df
docker system prune -f
```

## Volumes & Mounts
```bash
# Named volume
docker volume create data

# Use volume
docker run -d --name db \
  -v data:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=pass postgres:16

# Bind mount
docker run --rm -it -v "$PWD":/work -w /work node:20-alpine sh
```

## Networking
```bash
# Create bridge network
docker network create app-net

# Connect containers
docker run -d --name api --network app-net app:latest
docker run -d --name redis --network app-net redis:7

# Inspect networking
docker network ls
docker network inspect app-net
```

## Dockerfile Essentials
```Dockerfile
# syntax=docker/dockerfile:1.7
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine AS build
WORKDIR /app
COPY . .
RUN npm ci && npm run build

FROM node:20-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY --from=deps /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

## Common Flags
- -d: detached
- -p host:container: port mapping
- -e KEY=VAL: env var
- --rm: auto-remove on exit
- -v local:container or hostPath:containerPath
- --network: connect to network
- --restart unless-stopped|always

## Compose Compatibility Hint
- Prefer .dockerignore to minimize context
- Pin base image versions
- Use multi-stage to reduce size

## Troubleshooting
```bash
docker ps -a
docker inspect app | jq '.[0].Config.Env'
docker logs app --since 1h --tail 200

# Low disk space
docker system prune -af --volumes

# Container can’t resolve hostnames
cat /etc/resolv.conf
```

## Best Practices
- Keep images minimal (alpine, distroless)
- Use non-root users
- Cache dependencies effectively
- Use HEALTHCHECK and proper signals (STOPSIGNAL)
- Scan images (docker scout cves, trivy)
- Externalize config via env/volumes
