# Docker notes

Structured notes built from the raw Docker captures in this folder. The raw files are kept unchanged; corrections and additions are recorded here.

## Notes

| # | Note | Focus |
| --- | --- | --- |
| 01 | [Containers vs virtual machines](01-containers-vs-virtual-machines.md) | Isolation model, kernel sharing, when to use each |
| 02 | [Docker setup on AWS EC2](02-docker-setup-on-aws-ec2.md) | IAM role, SSM, official Docker install |
| 03 | [Docker CLI and the container lifecycle](03-docker-cli-and-container-lifecycle.md) | Images, containers, run flags, cleanup |
| 04 | [Dockerfiles and image builds](04-dockerfile-and-image-builds.md) | Instructions, layers, cache, registries |
| 05 | [Docker networking](05-docker-networking.md) | Bridges, drivers, DNS, port publishing |
| 06 | [Docker storage and volumes](06-docker-storage-and-volumes.md) | Volumes, bind mounts, backup |
| 07 | [Docker Compose](07-docker-compose.md) | Multi-container YAML, commands, networks |

## Raw sources (local only)

| File | What it contains |
| --- | --- |
| `raw/DevOps-Tool/Docker/docker.md` | Long-form capture: CLI, Dockerfile, networking, volumes, Compose (local only) |
| `raw/DevOps-Tool/Docker/docker.txt` | Earlier, shorter capture of the same session (local only) |
| `raw/DevOps-Tool/Docker/name-filter.png` | Docker CLI screenshot (local only) |
| `raw/DevOps-Tool/Docker/start-stop.png` | Docker CLI screenshot (local only) |

Original reference from the raw notes: <https://www.youtube.com/watch?v=rjjES5IsPdg>

## Corrections applied

| Raw note | Correction | Where |
| --- | --- | --- |
| `docker create volume <name>` | Correct command is `docker volume create <name>` | [06](06-docker-storage-and-volumes.md) |
| `docker compose down --volume` | Correct flag is `--volumes` (or `-v`) | [07](07-docker-compose.md) |
| "Each container joins the default bridge by docker compose" | Compose creates a project-scoped network, `<project>_default` | [05](05-docker-networking.md), [07](07-docker-compose.md) |
| `chown $USER /var/run/docker.sock` | Needs `sudo`; the `docker` group is root-equivalent | [02](02-docker-setup-on-aws-ec2.md) |
| `docker build -f parent-directory/test.txt` | `-f` takes a Dockerfile path, not arbitrary text | [04](04-dockerfile-and-image-builds.md) |
| `ps a` described as "process of all users" | `ps a` shows terminal-attached processes for all users; `ps -A` is all processes | [03](03-docker-cli-and-container-lifecycle.md) |
| "userdefined" listed as a network driver | It is a network you create, usually with the `bridge` driver | [05](05-docker-networking.md) |

## Status

All seven notes are `polished` and cross-linked. Blog drafts are kept local in `content/` and are not part of this repository.

Draft content exists for:

- `docker-containers-vs-vms`
- `dockerfile-build-cache`
- `docker-networking-and-volumes`
- `docker-setup-on-aws-ec2`
- `docker-cli-and-container-lifecycle`
- `docker-compose-projects`

Back to [repository index](../../../README.md).
