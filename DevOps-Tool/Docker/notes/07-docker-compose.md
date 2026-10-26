---
title: "Docker Compose"
domain: "DevOps"
tool: "Docker"
source:
  - "../../../raw/DevOps-Tool/Docker/docker.md"
  - "../../../raw/DevOps-Tool/Docker/docker.txt"
status: "polished"
last_reviewed: "2026-10-25"
tags: ["docker", "compose", "yaml", "multi-container"]
blog: ""
---

# Docker Compose

## TL;DR

- Compose defines a multi-container application in one YAML file on one host.
- The modern command is `docker compose` (v2 plugin), not `docker-compose` (v1).
- The preferred filename is `compose.yaml`; `docker-compose.yml` still works.
- Compose creates a project-scoped user-defined network, so services resolve each other by name.
- `docker compose down -v` removes volumes too. That is destructive.

## Why it matters

Running three `docker run` commands with matching networks, volumes and environment variables is error-prone and impossible to review. Compose makes the whole stack declarative, versionable and reproducible with one `up`.

## Core concepts

### File shape

```yaml
services:
  <service-name>:
    image: <image>
    # or
    build:
      context: <path>
      dockerfile: <Dockerfile name>
    container_name: <name>
    ports:
      - "host:container"
    environment:
      KEY: value
    volumes:
      - <host-path>:<container-path>
      - <volume-name>:<container-path>
    networks:
      - <network-name>
    depends_on:
      - <service-name>

volumes:
  <volume-name>:

networks:
  <network-name>:
```

Compose reads `.env` in the project directory for variable interpolation. Do not commit that file.

### Example: web + database

```yaml
services:
  web:
    image: nginx:1.27-alpine
    ports:
      - "8080:80"
    volumes:
      - ./site:/usr/share/nginx/html:ro
    depends_on:
      - db

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD:?set DB_PASSWORD in .env}
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

```bash
# .env (local only; never commit)
DB_PASSWORD=change-me
```

### What Compose creates for you

- A network named `<project>_default`, usually a user-defined bridge.
- Named volumes declared at the top level.
- Containers named `<project>-<service>-<index>` unless you set `container_name`.
- Labels that let Compose find and manage its own resources.

## Commands and examples

```bash
# Build images for services that use build:
docker compose build

# Run a one-off command in a service
docker compose run --rm web sh

# Build (if needed) and start everything in the background
docker compose up -d

# Use a non-default file
docker compose -f compose.prod.yaml up -d

# Status
docker compose ps
docker compose ps -a

# Logs
docker compose logs -f web

# Execute in a running service
docker compose exec web sh

# Stop without removing
docker compose stop

# Stop and remove containers, networks and orphans
docker compose down --remove-orphans

# Also remove named volumes (destructive)
docker compose down --volumes --remove-orphans

# Validate and render the final configuration
docker compose config
```

## Gotchas

- **`docker compose` vs `docker-compose`.** v1 is end-of-life; use the v2 plugin (`docker compose`).
- **Compose does not join `docker0`.** It creates a project-scoped network (`<project>_default`). The raw note saying every container joins the default bridge is incorrect.
- **`down --volume` is not the flag.** Use `--volumes` or `-v`.
- **`down -v` deletes named volumes.** Do not use it if the database data matters.
- **`depends_on` only controls start order.** It does not wait for the dependency to be ready. Use healthchecks and retries for readiness.
- **`container_name` blocks scaling.** With a fixed name, Compose cannot create multiple replicas of that service.
- **Relative bind-mount paths are relative to the Compose file**, not your shell's current directory.
- **The top-level `version:` key is obsolete** in Compose v2. Omit it.
- **Port collisions are common.** Change the host side of the mapping, not the container side.
- **`.env` is for interpolation, not container environment by itself.** Use `environment` or `env_file` to inject values into a service.

## Cheat sheet

| Task | Command |
| --- | --- |
| Build | `docker compose build` |
| Start detached | `docker compose up -d` |
| Stop and remove | `docker compose down` |
| Remove with volumes | `docker compose down -v` |
| Status | `docker compose ps -a` |
| Logs | `docker compose logs -f SERVICE` |
| Shell | `docker compose exec SERVICE sh` |
| One-off command | `docker compose run --rm SERVICE CMD` |
| Custom file | `docker compose -f FILE up -d` |
| Render config | `docker compose config` |

## Practice

1. Run the web + database example, connect from `web` to `db` by hostname, and confirm name resolution.
2. Change the host port, run `docker compose config`, and verify the rendered value.
3. Run `down`, then `down -v`, and observe the difference in volume persistence.

## Further reading

- [Docker Compose overview](https://docs.docker.com/compose/)
- [Compose file reference](https://docs.docker.com/reference/compose-file/)
- [Compose CLI reference](https://docs.docker.com/reference/cli/docker/compose/)
- [Environment variables in Compose](https://docs.docker.com/compose/environment-variables/)

## Source

- Raw capture (local only): `raw/DevOps-Tool/Docker/docker.md`, `raw/DevOps-Tool/Docker/docker.txt`
- Video referenced in the raw notes: <https://www.youtube.com/watch?v=rjjES5IsPdg>
