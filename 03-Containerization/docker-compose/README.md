# 🧩 Docker Compose Cheatsheet

[⬅️ Back to Root](../../README.md)

## Core Concepts
- Compose file version and schema
- Services, networks, volumes
- Build vs image, depends_on, healthcheck
- Profiles, extensions, env_file

## YAML Skeleton
```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8080:80"
    environment:
      - NODE_ENV=production
    depends_on:
      - db
  db:
    image: postgres:16
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: pass
volumes:
  db-data:
networks:
  default:
    driver: bridge
```

## Commands
```bash
# Start/stop
docker compose up -d
docker compose down

# Rebuild
docker compose up -d --build

# Logs & exec
docker compose logs -f web
docker compose exec web sh

# Scale
docker compose up -d --scale web=3

# Validate
docker compose config
```

## Useful Fields
- restart: unless-stopped|always
- healthcheck: test, interval, timeout, retries
- env_file: list of .env files
- profiles: enable services conditionally
- deploy: resources (Swarm)

## Best Practices
- Use .dockerignore to cut context
- Pin image tags, avoid latest
- Externalize secrets via env/volumes
- Keep one concern per service
