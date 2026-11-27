# Docker swarm generator — technical spec

## Intent

One run of the generator produces a collection of Pulumi projects — one for each resource a Docker swarm needs — plus the Ansible folder and a readme for the collection. This is the Docker swarm generator feature, area A2 of the PRD, defined in `product-generator-dzangolab-azure/wiki/briefs/provisioning-a-docker-swarm/docker-swarm-generator.feature.md`.

## Place in the package

A composite generator. It extends `PulumiGenerator`, is chosen as `swarm` (first in the CLI's Docker group), and receives the four shared answers through `_optionOrPrompt`. It composes the single-resource generators rather than emitting each project's files itself.

## Inputs

The four shared answers, asked once by the CLI. Then, for the whole collection:

- each question shared by several projects — such as the Azure region — asked once;
- each question specific to one project, asked once for that project;
- whether to provision managers, and how many;
- whether to provision workers, and how many.

## Files emitted

One ready-to-run Pulumi project, each with a readme, for: resource-group, vnet, static-ip, disks, nfs, key-vault, load-balancer, secrets and swarm-leader; one for managers and one for workers if the developer chose them; the Ansible folder; and a readme for the collection as a whole. Each project is identical to what that resource's own generator produces from the same answers. The tree the collection must match is `devops/infrastructure/swarm/azure`.

The `pulumi-azure-backend` project is **not** emitted by the swarm: it pre-exists as the Pulumi state backend, like the entra group. *(To confirm with Olivier: whether the backend project is a prerequisite outside the swarm, or part of the swarm tree in the production repo.)*

## Composition

It composes each single-resource generator in the collection, passing the shared answers and this generator's own options through `_optionOrPrompt` so nothing is asked twice. It also includes the Ansible generator's folder. The [entra generator](../entra-generator/spec.md) is **not** part of it: it pre-exists the swarm.

## Acceptance

- **Given** the dev has chosen swarm and answered the four common questions, **when** the generator runs, **then** it asks each question shared by several projects (such as the Azure region) only once, and each question specific to one project once.
- **Given** the generator is running, **when** it comes to the nodes, **then** it asks whether the dev wants to provision managers, and how many, and whether they want to provision workers, and how many.
- **Given** the dev says no to managers or to workers, **when** the generator finishes, **then** the collection has no project for them.
- **Given** the dev has answered every question, **when** the generator finishes, **then** they get one ready-to-run Pulumi project, with a readme, for each of resource-group, vnet, static-ip, disks, nfs, key-vault, load-balancer, secrets and swarm-leader; one for managers and one for workers if chosen; the Ansible folder; and a readme for the collection as a whole.
- **Given** the collection, **when** the dev compares one of its projects with what that resource's own generator produces from the same answers, **then** they are identical.
- **Given** the dev gives the same answers as the existing projects' data, **when** the generator runs against `devops/infrastructure/swarm/azure`, **then** there is no change.

## Boundaries

- Bastion, services and docs: not included for now; they may be added later.
- The entra generator is not part of the swarm generator.
- The pulumi-azure-backend generator is not part of the swarm generator: it pre-exists it.
- `mvps-infra/azure/managers`, which an unreviewed draft commit on generator-dzangolab-iac follows, is not the contract: the production tree under `devops/infrastructure/swarm/azure` is.
