---
title: "Terraform Workflow and State"
domain: "DevOps"
tool: "Terraform"
source:
  - "../../../raw/DevOps-Tool/Terraform/Terraform.txt"
status: "polished"
last_reviewed: "2026-10-25"
tags: ["terraform", "state", "plan", "apply", "workflow"]
blog: ""
---

# Terraform Workflow and State

## TL;DR

- The core loop is `init -> validate -> plan -> apply`.
- `plan` is a dry run; read it before every `apply`.
- `terraform.tfstate` maps configuration to real resources and stores metadata.
- State can contain sensitive values. Do not commit local state for real projects.
- `destroy` is the reverse of `apply` and deletes real infrastructure.

## Why it matters

Terraform's safety comes from the plan step: you see creates, updates and destroys before anything happens. State is what makes that possible. Losing state does not delete resources, but Terraform forgets it owns them, which leads to duplicates or manual cleanup.

## Core concepts

### The workflow

| Step | Command | What it does |
| --- | --- | --- |
| Initialise | `terraform init` | Downloads providers/modules, creates `.terraform/` and the lock file |
| Format | `terraform fmt` | Rewrites files to canonical HCL style (optional, safe) |
| Validate | `terraform validate` | Checks syntax and internal references; no API calls |
| Plan | `terraform plan` | Compares config, state and real infrastructure |
| Apply | `terraform apply` | Executes the plan after confirmation |
| Destroy | `terraform destroy` | Plans and deletes managed resources |

### Reading a plan

| Symbol | Meaning |
| --- | --- |
| `+` | Create |
| `~` | Update in place |
| `-` | Destroy |
| `-/+` | Replace (destroy then create) |
| `<=` | Read (data source) |

### State

`terraform.tfstate` is a JSON file that records:

- which resources Terraform manages,
- the mapping between config addresses and real object IDs,
- attribute values and dependency metadata.

For a solo lab, local state is fine. For teams, use a remote backend (S3 + DynamoDB, Terraform Cloud, and so on) with locking.

```bash
# List resources in state
terraform state list

# Show one resource; an address is required
terraform state show local_file.pet
```

## Commands and examples

```bash
# Start a project
terraform init

# Optional formatting
terraform fmt

# Static checks
terraform validate

# Preview. Read this output carefully.
terraform plan

# Apply. Add -auto-approve only in automation you trust.
terraform apply

# Inspect state
terraform state list
terraform state show local_file.pet

# Show outputs
terraform output

# Destroy everything managed by this configuration (destructive)
terraform destroy
```

Typical `plan` output:

```text
Terraform will perform the following actions:

  # local_file.pet will be created
  + resource "local_file" "pet" {
      + content  = "we love pets"
      + filename = "/tmp/pets.txt"
      + id       = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

## Gotchas

- **`state show` needs an address.** The raw note omits it; `terraform state show local_file.pet` is the usable form.
- **State is not a backup.** Back it up or use a remote backend with versioning.
- **State can contain secrets.** Plaintext local state may include passwords and keys. Restrict access and never commit it for real projects.
- **Do not commit `.terraform/`.** It contains downloaded binaries.
- **Do commit `.terraform.lock.hcl`.** It pins provider versions for reproducibility.
- **A lost state file does not delete resources.** Terraform simply no longer knows about them; use `terraform import` to reattach.
- **`plan` can still surprise you.** Read replacements (`-/+`); they destroy and recreate.
- **`-auto-approve` skips the human check.** Use it only in CI with a reviewed plan.
- **Remote state needs locking.** Without it, concurrent applies can corrupt state.
- **Provider changes require `init -upgrade` or a fresh `init`.**

## Cheat sheet

| Task | Command |
| --- | --- |
| Initialise | `terraform init` |
| Format | `terraform fmt` |
| Validate | `terraform validate` |
| Preview | `terraform plan` |
| Apply | `terraform apply` |
| Destroy | `terraform destroy` |
| List state | `terraform state list` |
| Show resource | `terraform state show ADDRESS` |
| Outputs | `terraform output` |
| Upgrade providers | `terraform init -upgrade` |

## Practice

1. Apply the `local_file` example, then change the content and read the `~ update` plan before applying.
2. Run `terraform state list` and `terraform state show local_file.pet`.
3. Back up the state file, delete it, run `plan`, and observe that Terraform wants to recreate the resource. Restore the backup afterwards.

## Further reading

- [Terraform CLI: init](https://developer.hashicorp.com/terraform/cli/commands/init)
- [Terraform CLI: plan](https://developer.hashicorp.com/terraform/cli/commands/plan)
- [Terraform CLI: apply](https://developer.hashicorp.com/terraform/cli/commands/apply)
- [State](https://developer.hashicorp.com/terraform/language/state)
- [Remote state and backends](https://developer.hashicorp.com/terraform/language/backend)

## Source

- Raw capture (local only): `raw/DevOps-Tool/Terraform/Terraform.txt`
- Install link from the raw notes: <https://developer.hashicorp.com/terraform/install>
