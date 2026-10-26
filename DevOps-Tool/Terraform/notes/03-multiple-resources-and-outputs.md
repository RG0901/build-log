---
title: "Multiple Resources and Outputs"
domain: "DevOps"
tool: "Terraform"
source:
  - "../../../raw/DevOps-Tool/Terraform/Terraform.txt"
status: "polished"
last_reviewed: "2026-10-25"
tags: ["terraform", "resources", "outputs", "random"]
blog: ""
---

# Multiple Resources and Outputs

## TL;DR

- You can declare many resources in one file, or split them across `.tf` files in the same directory.
- References between resources create an implicit dependency graph; Terraform orders operations for you.
- Outputs expose values after apply and can feed other tooling.
- `random_string` generates a value without creating infrastructure.
- Use `random_string.rand_str.result` for a single resource; the `[*]` splat is for `count` and `for_each`.

## Why it matters

Real configurations are graphs, not lists: a container needs an image, a DNS record needs a load balancer, an app needs a database endpoint. References express those dependencies once, and Terraform handles ordering, retries and replacements. Outputs make the result usable by humans and automation.

## Core concepts

### Multiple resources, multiple files

```text
project/
├─ main.tf
├─ random.tf
└─ outputs.tf
```

Terraform loads every `.tf` file in the working directory as one module. File names are for humans; the graph is built from references.

### Implicit dependencies

```hcl
resource "random_string" "suffix" {
  length  = 6
  special = false
}

resource "local_file" "name" {
  filename = "/tmp/${random_string.suffix.result}.txt"
  content  = "generated"
}
```

The `local_file` references `random_string.suffix.result`, so Terraform creates the random value first. No explicit ordering is required.

### The `random_string` example

```hcl
resource "random_string" "rand-str" {
  length           = 16
  special          = true
  override_special = "+={}"
}

output "rand-str" {
  value = random_string.rand-str[*].result
}
```

That works, but the splat is unnecessary here. With a single resource, read the attribute directly.

### Outputs

```hcl
output "file_path" {
  description = "Where the file was written"
  value       = local_file.pet.filename
}

output "random_value" {
  value     = random_string.rand-str.result
  sensitive = true
}
```

```bash
terraform output
terraform output -json
terraform output random_value
```

## Commands and examples

```bash
# Regenerate the plan after adding resources
terraform init
terraform validate
terraform plan
terraform apply

# Inspect a single object in state
terraform state show random_string.rand-str

# Read outputs
terraform output
terraform output -json
```

### Recommended form

```hcl
resource "random_string" "rand_str" {
  length           = 16
  special          = true
  override_special = "+={}"
}

output "rand_str" {
  value = random_string.rand_str.result
}
```

### Splat only when there are many instances

```hcl
resource "random_string" "suffix" {
  count   = 3
  length  = 6
  special = false
}

output "suffixes" {
  value = random_string.suffix[*].result
}
```

## Gotchas

- **`[*]` is not required for a single resource.** It produces a one-element list and makes the output harder to consume. Use the direct attribute unless you use `count` or `for_each`.
- **`state show` needs an address.** Use `terraform state show random_string.rand-str`.
- **Hyphens are legal in Terraform identifiers but confusing.** `rand-str` works; prefer `rand_str` to avoid ambiguity with subtraction.
- **Outputs are stored in state.** Do not put secrets in outputs unless you mark them `sensitive` and protect the state.
- **`sensitive = true` hides CLI output, not state.** The value is still in the state file.
- **Changing a resource address renames it in state.** Terraform may plan a destroy/create unless you use `moved` blocks or `terraform state mv`.
- **Outputs do not create anything.** They only expose values that already exist.

## Cheat sheet

| Task | Syntax / command |
| --- | --- |
| Declare a resource | `resource "type" "name" { ... }` |
| Reference an attribute | `type.name.attribute` |
| Declare an output | `output "name" { value = ... }` |
| Read outputs | `terraform output` |
| JSON outputs | `terraform output -json` |
| Mark sensitive | `sensitive = true` |
| List resources | `terraform state list` |
| Show a resource | `terraform state show ADDRESS` |
| Splat many instances | `type.name[*].attribute` |

## Practice

1. Add a `random_string` and a `local_file` whose filename uses the random value. Confirm the ordering in `plan`.
2. Expose the filename as an output and read it with `terraform output`.
3. Convert the single random resource into `count = 3` and update the output to a splat expression.

## Further reading

- [Resources](https://developer.hashicorp.com/terraform/language/resources)
- [Output values](https://developer.hashicorp.com/terraform/language/values/outputs)
- [Splat expressions](https://developer.hashicorp.com/terraform/language/expressions/splat)
- [`random_string` provider docs](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/string)

## Source

- Raw capture (local only): `raw/DevOps-Tool/Terraform/Terraform.txt`
- Install link from the raw notes: <https://developer.hashicorp.com/terraform/install>
