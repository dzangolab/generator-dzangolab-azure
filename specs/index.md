# Technical specs

One technical spec per feature, describing how the generator behaves and what files it emits. The functional requirements these specs are written from live in the product knowledge base (`product-generator-dzangolab-azure`).

- [Choosing a generator](choosing-a-generator/spec.md) — the `yo dzangolab-azure` CLI: four shared questions, then one grouped list.
- [Entra generator](entra-generator/spec.md) — the Entra group, users and permissions project.
- [Resource group generator](resource-group-generator/spec.md) — the pattern every other single-resource generator copies.
- [Docker swarm generator](docker-swarm-generator/spec.md) — the composite swarm generator plus Ansible.

## Day-one single-resource generators

The [resource group generator](resource-group-generator/spec.md) is the template for these; each will get its own spec as its brief is written. They extend `PulumiGenerator` and emit one Pulumi project each:

- resource-group (spec written)
- entra (spec written)
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
- pulumi-azure-backend

## Parked / not yet specified

These are referenced by the CLI or the swarm generator but have no brief in the product KB yet, so no spec has been written. They are out of scope until Olivier writes the brief:

- **ansible** — listed in the CLI's Docker group and included by the swarm generator, but it emits a folder (not a Pulumi project) and no brief describes it.
- **pulumi-azure-backend** — listed in the CLI's Pulumi group and pre-exists the swarm as the state backend, but no brief describes it.
- **bastion, services, docs** — folders in the production tree that the swarm generator does not include for now.

## Out of scope

- The other day-one single-resource generators above are named so the swarm spec and CLI spec stay consistent, but only resource-group and entra are specified so far.

## Reference tree

The golden tree each generator must reproduce is written as `devops/infrastructure/swarm/azure` in the specs. Olivier's brief referred to it as `infrastructure/swarm/azure`. The production repo is private, so the exact path must be confirmed against it before implementation; do not change the specs silently either way.

