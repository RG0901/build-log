---
title: "Managing Docker with the Terraform Docker Provider"
domain: "DevOps"
tool: "Terraform"
source:
  - "../../../raw/DevOps-Tool/Terraform/Terraform.txt"
status: "polished"
last_reviewed: "2026-10-25"
tags: ["terraform", "docker", "provider", "containers"]
blog: ""
---

# Managing Docker with the Terraform Docker Provider

## TL;DR

- Declare the `kreuzwerker/docker` provider, then model images and containers as resources.
- Resource references create dependencies: the container waits for the image.
- `kreuzwerker/docker` is shorthand for `registry.terraform.io/kreuzwerker/docker`.
- Terraform needs access to the Docker socket; the `docker` group is the practical option.
- The raw notes pin provider `~> 2.21.0`, where `docker_image.nginx.latest` is valid. Provider 3.x uses `.image_id`.

## Why it matters

This is the smallest useful example of Terraform managing something other than files. It shows the same pattern you will use for AWS, Kubernetes or Cloudflare: declare a provider, declare resources, reference them, and let the graph handle ordering. It is also a safe sandbox for learning state and drift.

## Core concepts

### Provider configuration

```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 2.21.0"
    }
  }
}

provider "docker" {}
```

- `required_providers` tells `terraform init` what to download.
- The short source `kreuzwerker/docker` means `registry.terraform.io/kreuzwerker/docker`.
- The empty `provider "docker" {}` block uses the default Docker host, usually `unix:///var/run/docker.sock`.

### Resources

```hcl
resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "nginx-terraform"

  ports {
    internal = 80
    external = 80
  }
}
```

`image = docker_image.nginx.latest` is a reference, so Terraform creates the image before the container.

### Version-specific attribute

| Provider version | Image reference |
| --- | --- |
| 2.x (for example `~> 2.21.0`) | `docker_image.nginx.latest` |
| 3.x | `docker_image.nginx.image_id` |

If you upgrade the provider, expect to update this reference. Pinning the version is why the raw example works as written.

## Commands and examples

### Prerequisites

```bash
# Check the daemon is reachable
docker ps

# If Docker is missing on a Debian/Ubuntu host (older distro package)
sudo apt-get update
sudo apt-get install -y docker.io

# Correct the raw note: chown needs sudo, and it is not the best fix
sudo chown "$USER" /var/run/docker.sock

# Better: add your user to the docker group, then start a new session
sudo usermod -aG docker "$USER"
```

The `docker` group is root-equivalent. Only add trusted users.

### Workflow

```bash
terraform init      # download the Docker provider
terraform validate
terraform plan      # expect one image and one container
terraform apply

docker ps           # nginx-terraform should be running
docker inspect nginx-terraform

terraform destroy   # remove the container and, with keep_locally = false, the image
```

Requires Terraform installed: <https://developer.hashicorp.com/terraform/install>

## Gotchas

- **`chown` needs `sudo`.** The raw command omits it. Group membership is the persistent fix, and it grants root-equivalent access.
- **Provider 3.x changed the image attribute.** `.latest` is a 2.x field; `.image_id` is the current one. Keep the pin or migrate deliberately.
- **`keep_locally = false` means destroy also removes the image.** Set it to `true` if you want to keep the pulled image.
- **Manual Docker changes cause drift.** If you `docker rm` a container Terraform owns, the next plan will try to recreate it.
- **Port 80 may already be in use.** Change `external` or stop the conflicting service.
- **Container names must be unique.** A leftover container with the same name makes apply fail.
- **State is local by default.** Keep `terraform.tfstate` out of git for anything beyond a lab.
- **Running Terraform itself may need socket access.** If `docker ps` works but Terraform does not, check the provider's host and your group membership.
- **The distro `docker.io` package is often older.** Prefer Docker's official repository when version behaviour matters.
- **`nginx:latest` is not reproducible.** Pin a tag such as `nginx:1.27-alpine` outside a lab.

## Cheat sheet

| Task | Command / syntax |
| --- | --- |
| Provider source | `kreuzwerker/docker` |
| Full registry address | `registry.terraform.io/kreuzwerker/docker` |
| Declare provider | `provider "docker" {}` |
| Image resource | `docker_image` |
| Container resource | `docker_container` |
| Image reference (2.x) | `docker_image.NAME.latest` |
| Image reference (3.x) | `docker_image.NAME.image_id` |
| Ports | `ports { internal = 80, external = 80 }` |
| Apply | `terraform apply` |
| Inspect container | `docker ps` / `docker inspect NAME` |
| Remove everything | `terraform destroy` |

## Practice

1. Apply the example, change the external port, and read the `~ update` plan before applying.
2. Pin `nginx:1.27-alpine` instead of `latest` and confirm the image changes.
3. Destroy the stack, then set `keep_locally = true` and confirm the image survives a second destroy.

## Further reading

- [Terraform Docker provider](https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs)
- [Terraform Docker tutorial](https://developer.hashicorp.com/terraform/tutorials/automation/automate-docker)
- [Docker provider: docker_image](https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs/resources/image)
- [Docker provider: docker_container](https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs/resources/container)
- [Install Terraform](https://developer.hashicorp.com/terraform/install)

## Source

- Raw capture (local only): `raw/DevOps-Tool/Terraform/Terraform.txt`
- Install link from the raw notes: <https://developer.hashicorp.com/terraform/install>
