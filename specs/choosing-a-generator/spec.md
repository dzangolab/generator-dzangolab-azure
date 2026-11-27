# Choosing a generator — technical spec

## Intent

Running `yo dzangolab-azure` must let a Dzango developer choose which generator to run from the collection, and it must ask the inputs several generators share only once. This is the Choosing a generator feature, area A4 of the PRD, defined in `product-generator-dzangolab-azure/wiki/briefs/cli/choosing-a-generator.feature.md`.

The behaviour follows the `app` generator in [generator-dzangolab-iac](https://github.com/dzangolab/generator-dzangolab-iac), with this package's generator list.

## Place in the package

The root `app` generator, run as `yo dzangolab-azure` with no subgenerator name. It extends Yeoman's `Generator` — not `PulumiGenerator`, because it emits no project of its own. It saves the four shared answers with `this.config.save()` (written to `.yo-rc.json`), then composes exactly one subgenerator.

## Inputs

The four shared questions, in this order:

1. The name of the project.
2. Whether to use a prefix in the folder names.
3. The environment, for example `prod`.
4. Whether to generate the stack config.

Then a single-choice list — arrow keys to move, Enter to pick — asking "What IaC code do you wish to generate?", with one grouped list:

- **Docker**: `swarm`, `leader`, `managers`, `workers`, `ansible`
- **Pulumi**: `pulumi-azure-backend`
- **Other**: every other generator — `resource-group`, `entra`, `vnet`, `static-ip`, `disks`, `nfs`, `key-vault`, `load-balancer`, `secrets`

A group that holds no generator yet is hidden.

The groups are a display arrangement, not a type system: `leader`, `managers`, `workers` (Docker) and `pulumi-azure-backend` (Pulumi) are single-resource generators just like those in Other — they extend `PulumiGenerator` and emit one Pulumi project each. The [resource group spec](../resource-group-generator/spec.md) defines the term.

## Files emitted

None of its own. It writes `.yo-rc.json` (the saved configuration) and delegates to the chosen generator, which emits the files its own spec describes.

## Composition

It composes the single generator the developer chose, passing the four shared answers so the chosen generator does not ask them again. Nothing composes it: it is the entry point to every generator in the collection.

## Acceptance

- **Given** the dev runs `yo dzangolab-azure`, **when** it starts, **then** it asks the name of the project, whether to use a prefix in folder names, the environment, and whether to generate the stack config, before showing the list.
- **Given** the dev has answered those four questions and chosen a generator, **when** that generator runs, **then** it does not ask any of them again.
- **Given** the dev has answered the four questions, **when** the list appears, **then** it asks "What IaC code do you wish to generate?", and the generators sit under the group names Docker, Pulumi and Other, in that order, with the Docker generators in the order swarm, leader, managers, workers, ansible.
- **Given** the list is shown, **when** the dev moves with the arrow keys and presses Enter, **then** exactly one generator is chosen, and it runs.
- **Given** a group has no generator available yet, **when** the list appears, **then** that group is not shown.
- **Given** the list, **when** it is rendered, **then** where generator-dzangolab-iac shows `undefined`, this shows the group's name — Docker, Pulumi, Other — and the "`list` prompt is deprecated" warning is not reproduced.

## Boundaries

- Checking whether a chosen generator depends on another not yet generated (for example, workers before the swarm leader).
- What each generator asks once chosen: that is each resource generator's own spec (R2 in the PRD).
