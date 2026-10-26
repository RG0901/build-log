---
title: "Dockerfiles and Image Builds"
domain: "DevOps"
tool: "Docker"
source:
  - "../../../raw/DevOps-Tool/Docker/docker.md"
  - "../../../raw/DevOps-Tool/Docker/docker.txt"
status: "polished"
last_reviewed: "2026-10-25"
tags: ["docker", "dockerfile", "images", "build", "registry"]
blog: ""
---

# Dockerfiles and Image Builds

## TL;DR

- A **Dockerfile** is a recipe; an **image** is the baked, layered result.
- Every instruction creates a layer that can be cached and reused.
- Order instructions from least to most frequently changed so the cache survives edits.
- `docker build -f` points at a **Dockerfile**, not an arbitrary text file.
- Tag images explicitly and push them with a registry-qualified name.

## Why it matters

A Dockerfile is the difference between "it works on my machine" and a reproducible artifact. Most build problems are cache problems or context problems: the right file was copied in the wrong order, the build context was too large, or a secret leaked into a layer.

## Core concepts

### Build context

The build context is the directory sent to the builder. `COPY` and `ADD` can only read from it.

```bash
# The final "." is the build context
docker build -t myapp:1.0 .
```

Use `.dockerignore` to keep the context small and to avoid copying secrets or `.git`.

### Layers and cache

- Each Dockerfile instruction creates a layer.
- Layers are read-only and content-addressed.
- If a layer's inputs and parent layers are unchanged, Docker reuses the cached layer.
- Changing an early instruction invalidates every later layer.

### Dockerfile instructions

| Instruction | Purpose | Notes |
| --- | --- | --- |
| `FROM` | Base image | Pin a tag; avoid `latest` for reproducible builds |
| `RUN` | Run a command at build time, commit a layer | Chain commands with `&&`; clean caches in the same layer |
| `CMD` | Default runtime command | One effective value; overridable at `docker run` |
| `ENTRYPOINT` | Fixed runtime executable | Harder to override; pairs with `CMD` for default args |
| `EXPOSE` | Document the listening port | Informational only; does not publish the port |
| `COPY` | Copy from build context | Cannot read outside the context; `--chown` is available |
| `WORKDIR` | Set the working directory | Creates the directory if missing |
| `ENV` | Set an environment variable | Persists at runtime; overridable with `-e` |
| `LABEL` | Add image metadata | Do not put secrets here |
| `ARG` | Build-time variable | Visible in build history; not a secret store |

### `CMD` vs `ENTRYPOINT`

| | `CMD` | `ENTRYPOINT` |
| --- | --- | --- |
| Role | Default command or arguments | Executable that always runs |
| Override at run time | Replace the command after the image name | `--entrypoint` |
| Multiple values | Last one wins | Last one wins |

## Commands and examples

### Example Dockerfile

```dockerfile
# Base image. Pin a tag for reproducibility.
FROM nginx:1.27-alpine

# Metadata
LABEL org.opencontainers.image.title="static-site"

# Runtime environment
ENV APP_ENV=production

# Working directory inside the image
WORKDIR /usr/share/nginx/html

# Copy only what the image needs, from the build context
COPY index.html .

# Documentation only; does not publish the port
EXPOSE 80

# Default process for the container
CMD ["nginx", "-g", "daemon off;"]
```

### Example `.dockerignore`

```text
.git
.gitignore
node_modules
*.md
Dockerfile
.dockerignore
```

### Build

```bash
# Build with a tag. The last argument is the build context.
docker build -t static-site:1.0 .

# Use a Dockerfile outside the context root.
# -f is the Dockerfile path; the last argument is still the context.
docker build -f docker/Dockerfile.prod -t static-site:prod .

# Inspect layers
docker history static-site:1.0
```

### Cache-friendly ordering

```dockerfile
# Copy dependency manifests first, install, then copy source.
# Source edits then do not invalidate the dependency layer.
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
```

### Chain `RUN` commands

```dockerfile
RUN apt-get update \
 && apt-get install -y --no-install-recommends curl \
 && rm -rf /var/lib/apt/lists/*
```

### Tag and push to a registry

Image names follow `registry/namespace/repository:tag`. On Docker Hub the registry is implicit.

```bash
# Local tag
docker build -t static-site:1.0 .

# Registry-qualified tag (creates an alias, does not copy)
docker tag static-site:1.0 rg0901/static-site:1.0

# Authenticate (omit the registry for Docker Hub)
docker login

# Push
docker push rg0901/static-site:1.0

# Pull it back anywhere
docker pull rg0901/static-site:1.0
```

## Gotchas

- **`-f` is the Dockerfile path.** In the raw notes the example points at a `.txt` file; Docker will try to parse it as a Dockerfile and fail unless it contains valid instructions.
- **`COPY` cannot read outside the build context.** Move the file into the context or change the context.
- **A large context is slow.** Add `.dockerignore` before you wonder why builds take minutes.
- **`EXPOSE` does not publish a port.** Use `-p` or Compose `ports`.
- **`latest` is not a version.** Pin a tag or a digest for reproducible builds.
- **Secrets in `ENV`, `ARG` or `LABEL` end up in image metadata and layer history.** Use a build secret mechanism (for example BuildKit secret mounts), not environment variables.
- **`CMD` is overridden by any command after the image name**; `ENTRYPOINT` is not.
- **Changing a file copied early invalidates later layers.** Copy manifests before source.
- **Rebuilds may reuse stale cache.** Use `--no-cache` when you need a clean build.
- **Architecture matters.** Use `--platform` when building for a different CPU architecture.

## Cheat sheet

| Task | Command |
| --- | --- |
| Build | `docker build -t name:tag .` |
| Custom Dockerfile | `docker build -f path/Dockerfile -t name:tag context/` |
| Show layers | `docker history name:tag` |
| Tag for a registry | `docker tag local:tag registry/user/repo:tag` |
| Log in | `docker login` |
| Push | `docker push registry/user/repo:tag` |
| Pull | `docker pull registry/user/repo:tag` |
| Clean build | `docker build --no-cache -t name:tag .` |

## Practice

1. Build the static-site example, run it on port 8080, and change `index.html`. Rebuild and confirm the cache behaviour.
2. Move `COPY . .` above `RUN npm ci` in a sample Node Dockerfile and compare rebuild times.
3. Tag an image for Docker Hub, push it to a test repository, then pull it on another machine or after `docker rmi`.

## Further reading

- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [Building best practices](https://docs.docker.com/build/building/best-practices/)
- [Build context](https://docs.docker.com/build/building/context/)
- [.dockerignore](https://docs.docker.com/build/building/context/#dockerignore-files)
- [Docker Hub](https://hub.docker.com/)

## Source

- Raw capture (local only): `raw/DevOps-Tool/Docker/docker.md`, `raw/DevOps-Tool/Docker/docker.txt`
- Video referenced in the raw notes: <https://www.youtube.com/watch?v=rjjES5IsPdg>
