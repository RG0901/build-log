# Terraform notes

Structured notes built from the raw Terraform capture in this folder. The raw file is kept unchanged; corrections and additions are recorded here.

## Notes

| # | Note | Focus |
| --- | --- | --- |
| 01 | [Infrastructure as Code and HCL basics](01-iac-and-hcl-basics.md) | Declarative model, HCL blocks, providers, resources |
| 02 | [Terraform workflow and state](02-workflow-and-state.md) | init, plan, apply, state, destroy |
| 03 | [Multiple resources and outputs](03-multiple-resources-and-outputs.md) | References, random_string, outputs, splat |
| 04 | [Managing Docker with the Terraform Docker provider](04-docker-provider-example.md) | Provider setup, image/container resources, versions |

## Raw source (local only)

| File | What it contains |
| --- | --- |
| `raw/DevOps-Tool/Terraform/Terraform.txt` | IaC intro, HCL, local and Docker provider examples (local only) |

Original reference from the raw notes: <https://developer.hashicorp.com/terraform/install>

## Corrections applied

| Raw note | Correction | Where |
| --- | --- | --- |
| "local is provider, pet is name" | Provider is `local`, resource type is `local_file`, local name is `pet` | [01](01-iac-and-hcl-basics.md) |
| `terraform state show` | Requires a resource address, e.g. `terraform state show local_file.pet` | [02](02-workflow-and-state.md), [03](03-multiple-resources-and-outputs.md) |
| `random_string.rand-str[*].result` for one resource | Direct attribute `.result` is correct; splat is for `count`/`for_each` | [03](03-multiple-resources-and-outputs.md) |
| `chown $USER /var/run/docker.sock` | Needs `sudo`; group membership (`usermod -aG docker`) is the persistent fix | [04](04-docker-provider-example.md) |
| `image = docker_image.nginx.latest` | Valid for provider 2.x as pinned; provider 3.x uses `.image_id` | [04](04-docker-provider-example.md) |
| `apt-get install docker.io` | Needs `sudo`, and the distro package is often older than Docker's official repo | [04](04-docker-provider-example.md) |
| `<block> <parameters> { ... }` template | HCL blocks are a block type plus labels, then arguments | [01](01-iac-and-hcl-basics.md) |

## Status

All four notes are `polished` and cross-linked. Blog drafts are kept local in `content/` and are not part of this repository.

Back to [repository index](../../../README.md).
