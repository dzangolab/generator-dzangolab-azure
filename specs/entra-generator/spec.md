# Entra generator — technical spec

## Intent

A generator that produces a ready-to-run Pulumi project creating an Entra group, adding its users, and giving them the permissions to provision the swarm's resources. This is the Entra generator feature, delivering S1 and R1 of the PRD, defined in `product-generator-dzangolab-azure/wiki/briefs/single-resource-type-generation/entra-generator.feature.md`.

## Place in the package

A single-resource generator. It extends `PulumiGenerator`, the base class described in the [resource group spec](../resource-group-generator/spec.md). It is chosen from the CLI's Other group, and receives the four shared answers through `_optionOrPrompt`, so it does not ask them again. It is **not** part of the Docker swarm generator.

## Inputs

After the four shared answers, the generator asks:

- the group's name;
- the ID of an existing group to take over, if any;
- the group's owners and members, by email;
- the permissions, each a role on a resource group or other resource;
- the directory roles each member gets — read access to the directory unless the developer says otherwise;
- whether to protect the resources against deletion;
- whether to keep them when the project is torn down.

## Files emitted

A ready-to-run Pulumi project with a readme, identical to the existing project apart from the project-specific data. It does not create user accounts: it adds people who already have an account.

## Composition

The [resource group generator](../resource-group-generator/spec.md) and the swarm projects rely on Entra groups that already exist with their permissions; this generator is where those groups come from. The [Docker swarm generator](../docker-swarm-generator/spec.md) does **not** compose it — the Entra generator pre-exists the swarm.

## Acceptance

- **Given** the dev has chosen the entra generator and answered the four common questions, **when** it runs, **then** it asks the group's name; the ID of an existing group to take over, if any; the owners and members, by email; the permissions; the directory roles each member gets; whether to protect against deletion; and whether to keep on teardown.
- **Given** the dev has answered every question, **when** the generator finishes, **then** they get a ready-to-run Pulumi project, with a readme, identical to the existing project apart from the project-specific data.
- **Given** the dev gave no existing group's ID, **when** the project is run, **then** it creates the group; **given** they gave one, **then** it takes that group over instead.
- **Given** the dev gives the same answers as the existing project's data, **when** the generator runs against the existing project under `iam/entra`, **then** there is no change.

## Boundaries

- It is not part of the Docker swarm generator; it pre-exists it.
- Creating user accounts: the project only adds people who already have an account.
- Running the generated project: that is the administrator's job, not the generator's.
