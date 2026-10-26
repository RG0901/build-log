---
title: "Infrastructure as Code and HCL Basics"
domain: "DevOps"
tool: "Terraform"
source:
  - "../../../raw/DevOps-Tool/Terraform/Terraform.txt"
status: "polished"
last_reviewed: "2026-10-25"
tags: ["terraform", "iac", "hcl", "providers"]
blog: ""
---

# Infrastructure as Code and HCL Basics

## TL;DR

- Terraform is a declarative infrastructure tool: you describe the end state, not the steps.
- Configuration is written in **HCL** (HashiCorp Configuration Language).
- A **provider** is a plugin that talks to an API (AWS, Docker, local files, and so on).
- A **resource** is one object Terraform manages, addressed as `<type>.<local-name>`.
- Terraform records what it created in a **state file**; that file is how it maps config to real objects.

## Why it matters

Clicking through a console is fast once and impossible to repeat, review or roll back. Terraform turns infrastructure into text: versioned in git, reviewed in pull requests, and reproducible across environments. The state file is what lets Terraform know what already exists instead of duplicating it.

## Core concepts

### Declarative vs imperative

- **Imperative:** run these steps in this order.
- **Declarative:** here is the desired end state; work out the steps.

Terraform is declarative. You describe resources; Terraform builds a dependency graph and decides the order.

### HCL block syntax

```hcl
<block_type> "<label>" "<label>" {
  argument = value
}
```

Most blocks have labels. A resource block has two:

```hcl
resource "local_file" "pet" {
  filename = "/tmp/pets.txt"
  content  = "we love pets"
}
```

| Part | Value | Meaning |
| --- | --- | --- |
| Block type | `resource` | What kind of block this is |
| First label | `local_file` | Resource type, prefixed by the provider name (`local`) |
| Second label | `pet` | Local name used inside this configuration |
| Address | `local_file.pet` | How other parts of the config refer to it |

The raw note says "local is provider, pet is name". More precisely: the resource type is `local_file`, the provider is `local`, and `pet` is the local name. The provider is inferred from the resource type prefix unless you declare it.

### Providers

Providers are distributed separately from Terraform and downloaded by `terraform init`.

Common examples:

| Provider | Use |
| --- | --- |
| `hashicorp/local` | Local files |
| `hashicorp/random` | Random values |
| `hashicorp/aws` | AWS resources |
| `kreuzwerker/docker` | Docker images and containers |

Provider source addresses live in the public registry, for example `registry.terraform.io/hashicorp/local`.

## Commands and examples

### Install and verify

```bash
# Install from the official page below, then:
terraform --version
```

Official install guide: <https://developer.hashicorp.com/terraform/install>

### First configuration

Create `main.tf`:

```hcl
terraform {
  required_providers {
    local = {
      source = "hashicorp/local"
    }
  }
}

resource "local_file" "pet" {
  filename = "/tmp/pets.txt"
  content  = "we love pets"
}
```

```bash
terraform init      # download the local provider
terraform validate  # check syntax and references
terraform plan      # show what would change
terraform apply     # create the file

cat /tmp/pets.txt   # "we love pets"
```

On Windows, use a path such as `C:/temp/pets.txt` or a relative path. `local_file` accepts both.

### What `terraform init` creates

```text
.terraform/                  # downloaded providers and modules
.terraform.lock.hcl          # provider version lock file (commit this)
```

## Gotchas

- **A provider is not a resource.** `local` is the provider; `local_file` is the resource type; `pet` is the local name.
- **`terraform init` is required** after adding or changing a provider, and on a fresh clone.
- **Paths are platform-specific.** `/root/pets.txt` assumes a Linux root shell. Use a relative path or the correct absolute path for your OS.
- **HCL strings need quotes.** Numbers and booleans do not (`length = 16`, `special = true`).
- **All `.tf` files in a directory are one module.** You do not need to import files explicitly.
- **Do not edit the state file by hand.** Use Terraform commands.
- **The provider version matters.** Behaviour and attributes can differ between major versions; pin them.

## Cheat sheet

| Task | Command / syntax |
| --- | --- |
| Check version | `terraform --version` |
| Download providers | `terraform init` |
| Validate config | `terraform validate` |
| Preview changes | `terraform plan` |
| Apply changes | `terraform apply` |
| Resource block | `resource "type" "name" { ... }` |
| Resource address | `type.name` |
| Provider registry | `registry.terraform.io/<namespace>/<provider>` |

## Practice

1. Write a `local_file` resource that writes your name into a temporary file, then apply it.
2. Add a second file resource in a separate `.tf` file and confirm `terraform plan` sees both.
3. Break the syntax on purpose and read the `terraform validate` error.

## Further reading

- [Terraform introduction](https://developer.hashicorp.com/terraform/intro)
- [HCL configuration syntax](https://developer.hashicorp.com/terraform/language/syntax/configuration)
- [Providers](https://developer.hashicorp.com/terraform/language/providers)
- [Terraform registry](https://registry.terraform.io/)
- [Install Terraform](https://developer.hashicorp.com/terraform/install)

## Source

- Raw capture (local only): `raw/DevOps-Tool/Terraform/Terraform.txt`
- Install link from the raw notes: <https://developer.hashicorp.com/terraform/install>
