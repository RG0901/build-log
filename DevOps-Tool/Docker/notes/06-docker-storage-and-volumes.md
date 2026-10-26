---
title: "Docker Storage and Volumes"
domain: "DevOps"
tool: "Docker"
source:
  - "../../../raw/DevOps-Tool/Docker/docker.md"
  - "../../../raw/DevOps-Tool/Docker/docker.txt"
status: "polished"
last_reviewed: "2026-10-25"
tags: ["docker", "volumes", "storage", "persistence", "bind-mounts"]
blog: ""
---

# Docker Storage and Volumes

## TL;DR

- A container's writable layer is deleted with the container. Persist data outside it.
- A **volume** is managed by Docker and stored under `/var/lib/docker/volumes/`.
- A **bind mount** maps an existing host path into the container.
- Volumes are better for data; bind mounts are better for live-editing config and source.
- `docker volume create`, not `docker create volume`, is the correct command.

## Why it matters

Databases, uploads and build caches all outlive a single container. If you store them in the container's writable layer, every `docker rm` destroys the data. Volumes also decouple storage from container lifecycle, which is required for upgrades and backups.

## Core concepts

### The writable layer

Every container gets a thin writable layer on top of its image layers. It is:

- fast for small writes,
- tied to the container,
- gone when the container is removed,
- not a good place for a database.

### Volume vs bind mount

| | Volume | Bind mount |
| --- | --- | --- |
| Managed by | Docker | You / the host |
| Host location | `/var/lib/docker/volumes/<name>/_data` (Linux default) | Any host path |
| Created with | `docker volume create` | The directory must exist |
| Portable across hosts | Yes, with tooling | No, host-specific |
| Best for | Databases, uploads, persistent state | Source code, config files, local development |
| Can be declared in a Dockerfile | Only as an anonymous `VOLUME` | No |

A third option, `tmpfs`, stores data in memory and disappears on stop. Use it for scratch data, not persistence.

### How mounting works

- `-v myvol:/data` mounts a named volume at `/data`.
- `-v /host/path:/data` mounts a host directory at `/data`.
- `:ro` makes the mount read-only.
- `--volumes-from other-container` reuses another container's volume mounts.

The `--mount` flag is more explicit and is preferred in scripts:

```bash
--mount type=volume,src=myvol,dst=/data
--mount type=bind,src=/host/path,dst=/data,readonly
```

## Commands and examples

```bash
# Create and inspect volumes
# The raw notes say "docker create volume"; the correct command is below.
docker volume create app-data
docker volume ls
docker volume inspect app-data

# Use a named volume
docker run -d --name db \
  -v app-data:/var/lib/postgresql/data \
  postgres:16-alpine

# Bind mount a host directory, read-only
docker run -d --name web \
  -v "$PWD/site:/usr/share/nginx/html:ro" \
  -p 8080:80 \
  nginx:1.27-alpine

# Equivalent explicit form
docker run -d --name web2 \
  --mount type=bind,src="$PWD/site",dst=/usr/share/nginx/html,readonly \
  -p 8081:80 \
  nginx:1.27-alpine

# Reuse another container's mounts
docker run --rm --volumes-from db alpine ls /var/lib/postgresql/data

# Back up a named volume into the current directory
docker run --rm \
  -v app-data:/data \
  -v "$PWD":/backup \
  alpine tar czf /backup/app-data.tgz -C /data .

# Remove a volume (only when no container uses it)
docker volume rm app-data

# Remove all unused volumes (destructive; double-check first)
docker volume prune
```

The Linux path for a named volume:

```text
/var/lib/docker/volumes/<volume-name>/_data/
```

## Gotchas

- **`docker create volume` is invalid.** The command is `docker volume create`.
- **`docker rm` does not remove named volumes.** Remove them explicitly with `docker volume rm`.
- **`docker compose down -v` deletes named volumes.** That is data loss by design; omit `-v` when you want to keep the database.
- **Bind-mount paths are host-specific.** The same Compose file will not work unchanged on another machine.
- **Missing host paths behave differently.** `-v` creates a missing host directory; `--mount` with `type=bind` fails if the path does not exist.
- **Permissions follow numeric UID/GID.** A container user may not be able to write to a host directory owned by another UID. Fix ownership deliberately, not with `chmod 777`.
- **SELinux hosts need a label.** Use `:z` (shared) or `:Z` (private) on bind mounts when SELinux is enforcing.
- **`docker volume prune` removes unused volumes**, which may include data you intended to keep.
- **Docker Desktop uses a VM.** The "host path" is shared into that VM, so not every host path is available.

## Cheat sheet

| Task | Command |
| --- | --- |
| Create a volume | `docker volume create NAME` |
| List volumes | `docker volume ls` |
| Inspect | `docker volume inspect NAME` |
| Use a named volume | `-v NAME:/path` |
| Bind mount | `-v /host/path:/path[:ro]` |
| Explicit mount | `--mount type=volume,src=NAME,dst=/path` |
| Reuse mounts | `--volumes-from CONTAINER` |
| Remove volume | `docker volume rm NAME` |
| Remove unused volumes | `docker volume prune` |
| Volume data path (Linux) | `/var/lib/docker/volumes/NAME/_data` |

## Practice

1. Run a Postgres container with a named volume, insert a row, remove the container, and start a new one on the same volume. Confirm the row survives.
2. Run nginx with a read-only bind mount and edit the HTML on the host. Confirm the change appears without a rebuild.
3. Back up the Postgres volume with the tar example, then restore it into a fresh volume.

## Further reading

- [Docker storage overview](https://docs.docker.com/engine/storage/)
- [Volumes](https://docs.docker.com/engine/storage/volumes/)
- [Bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)
- [`docker volume` reference](https://docs.docker.com/reference/cli/docker/volume/)

## Source

- Raw capture (local only): `raw/DevOps-Tool/Docker/docker.md`, `raw/DevOps-Tool/Docker/docker.txt`
- Video referenced in the raw notes: <https://www.youtube.com/watch?v=rjjES5IsPdg>
