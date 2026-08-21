# HCP Terraform Stacks Example (dev + prod)

**Read this before using the code.** HCP Terraform Stacks' exact configuration
syntax changed between its beta and GA releases — there are reports of these
changes breaking existing demo configs at HashiConf. The structure and
concepts below are solid, but the precise block/attribute names in
`main.tfcomponent.hcl` and `deployments.tfdeploy.hcl` should be verified
against the current docs before you run this for real:

- https://developer.hashicorp.com/terraform/language/stacks
- https://developer.hashicorp.com/terraform/cloud-docs/stacks

## Layout

```
.
├── components/
│   ├── vpc/              # plain Terraform, a "component"
│   └── database/         # depends on the vpc component's outputs
├── main.tfcomponent.hcl  # wires components together, declares providers
├── variables.tfcomponent.hcl
└── deployments.tfdeploy.hcl  # dev + prod, each with its own input values
```

## Key concepts (these are stable, unlike the exact syntax)

- **Component** — a reusable unit of infrastructure (here: `vpc`, `database`),
  similar in spirit to a Terraform module but declared at the Stack level.
- **Deployment** — an instance of the whole component graph with its own
  input values — this replaces what a "workspace" was for. `dev` and `prod`
  here are deployments of the same component graph.
- **Automatic dependency ordering** — because `database` references
  `component.vpc.public_subnet_ids`, HCP Terraform knows to apply `vpc`
  first and defers `database` until it succeeds. No manual `run-all` or
  ordering needed.

## How this differs from what we built earlier

- The plain-Terraform monorepo example used one workspace per environment
  with a `cloud` block — that's the classic **Workspace** model.
- This example uses one **Stack** with two **Deployments** (`dev`, `prod`)
  instead of two separate workspaces — the dependency between `vpc` and
  `database` is handled natively, instead of via `dependency` blocks
  (Terragrunt) or `trigger_patterns` (workspace automation).

## Before deploying

1. Verify syntax against current docs (see warning above).
2. Set up an HCP Terraform project with Stacks enabled.
3. Run `terraform stacks init` inside this directory, then
   `terraform stacks plan` / `terraform stacks apply` per the current CLI
   reference — command names have also evolved post-GA, so check
   `terraform stacks -usage` for what's available in your installed version.
