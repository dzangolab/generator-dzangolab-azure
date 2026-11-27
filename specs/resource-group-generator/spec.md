# Resource group generator — technical spec

## Intent

A Dzango developer chooses the resource group generator and gets a ready-to-run Pulumi project, with a readme, that provisions an Azure resource group. This is the Resource group generator feature, area A1 of the PRD, defined in `product-generator-dzangolab-azure/wiki/briefs/single-resource-type-generation/resource-group-generator.feature.md`.

This generator is the pattern every other single-resource generator copies, so this spec also names those generators.

## Place in the package

A single-resource generator. The term describes a *class*, not a place in the CLI list: a single-resource generator extends `PulumiGenerator` and emits one Pulumi project. The CLI groups these by theme — the swarm nodes sit under Docker, `pulumi-azure-backend` under Pulumi, and the rest (this generator, entra, vnet, static-ip, disks, nfs, key-vault, load-balancer, secrets) under Other — but all of them are single-resource generators.

This generator extends `PulumiGenerator`, the base class the other single-resource generators share. It is chosen from the CLI's Other group, and receives the four shared answers through `_optionOrPrompt`, so it does not ask them again.

## Inputs

After the four shared answers, the generator asks:

- the Azure region — Germany West Central unless the developer says otherwise;
- the resource group's own name, when it differs from the project name;
- whether to protect it against deletion — no unless the developer says otherwise;
- whether to keep it when the project is torn down — no unless the developer says otherwise;
- its tags — none unless the developer gives some.

## Files emitted

A ready-to-run Pulumi project, with a readme, identical to the existing project apart from the project-specific data. What it provisions is one resource group per stack, as per Pulumi practice. The tree it must match is `devops/infrastructure/swarm/azure/resource-group`.

## Composition

The [Docker swarm generator](../docker-swarm-generator/spec.md) includes one of these projects, produced from the same answers.

The day-one single-resource generators copy this generator's pattern, each matching its own folder under `devops/infrastructure/swarm/azure`:

- pulumi-azure-backend
- vnet
- static-ip
- disks
- nfs
- key-vault
- load-balancer
- secrets
- swarm-leader
- swarm-managers
- swarm-workers

The [entra generator](../entra-generator/spec.md) is the other single-resource generator, and it has its own spec. The Ansible generator is not one of these: it emits a folder, not a Pulumi project.

A draft branch `feat/azure-swarm-leader` on generator-dzangolab-iac already holds `generators/azure-swarm-leader/` and `generators/azure-swarm-managers/`. Its leader commit says it follows `swarm/azure/swarm-leader`; its managers commit says it follows `mvps-infra/azure/managers`. Neither is reviewed, and the contract stays the production tree under `devops/infrastructure/swarm/azure` — `mvps-infra` is not used as that tree.

## Acceptance

- **Given** the dev has chosen the resource group generator and answered the four common questions, **when** it runs, **then** it asks the Azure region (Germany West Central by default); the resource group's own name when it differs; whether to protect against deletion (no); whether to keep on teardown (no); and its tags (none).
- **Given** the dev has answered every question, **when** the generator finishes, **then** they get a ready-to-run Pulumi project, with a readme, identical to the existing project apart from the project-specific data.
- **Given** the generated project, **when** the dev looks at what it provisions, **then** it is one resource group per stack.
- **Given** the dev gives the same answers as the existing project's data, **when** the generator runs against `devops/infrastructure/swarm/azure/resource-group`, **then** there is no change.

## Boundaries

- The entra generator: its own spec.
- Generators for the other existing folders (bastion, services, docs): parked for now.
- What happens when the dev runs the generator again over a project they already have: the dev is asked what to do, and that is not part of this feature.
