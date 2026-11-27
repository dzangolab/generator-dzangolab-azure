# Implementation plan — generator-dzangolab-azure

## Status

Draft, not yet reviewed. No code is written until the design, testing plan, and this plan are reviewed — at least one night between writing and review.

## Prerequisites (blocking)

- Confirm the golden tree path: `devops/infrastructure/swarm/azure` (specs) vs `infrastructure/swarm/azure` (Olivier's brief). The production repo is private; the exact path must be checked before the diff fixtures are pinned.
- Confirm whether `pulumi-azure-backend` is a prerequisite outside the swarm (like entra) or part of the swarm tree.
- Confirm where the three docs live (this repo vs product KB).

## Phases

### Phase 0 — Package scaffold

Mirror `generator-dzangolab-iac`'s tooling: `package.json` (ESM), `biome.json`, husky commit hooks, shipjs release workflow, `.editorconfig`, `.gitignore`, license, and a `__tests__` layout. Bring over `generators/pulumi/index.js` (`PulumiGenerator`) and `generators/pulumi/optionOrPrompt.js` as the base class.

### Phase 1 — Root `app` generator

Implement the CLI entry point (`yo dzangolab-azure`): the four shared questions, `.yo-rc.json` save, the grouped list (Docker / Pulumi / Other), and `composeWith` delegation. Fix the iac grouping bug (no `undefined` headings, no deprecated `list` warning).

### Phase 2 — resource-group generator (the template)

Implement `resource-group` first. It is the pattern every other single-resource generator copies, so it sets the convention for `PulumiGenerator`, `_optionOrPrompt`, `templates/<name>`, and the `writing()` flow. Land its golden-tree diff test before moving on.

### Phase 3 — Remaining single-resource generators

Clone the resource-group pattern for: vnet, static-ip, disks, nfs, key-vault, load-balancer, secrets, swarm-leader, swarm-managers, swarm-workers. Add each to the CLI list and add its golden-tree fixture.

### Phase 4 — entra generator

Implement `entra` (group, users, permissions). It is a single-resource generator but is **not** composed by the swarm.

### Phase 5 — swarm composite + ansible

Implement `swarm` as the composite generator: compose the single-resource generators, ask shared questions once, include the Ansible folder. The Ansible generator itself has no brief yet, so implement only the folder emission the swarm needs, and flag it as parked.

### Phase 6 — Port and re-check the iac drafts

The unreviewed `feat/azure-swarm-leader` and `feat/azure-swarm-managers` branches on iac are a starting point for the swarm-node generators, **not** a contract. Port them, then correct the managers generator so it follows the production tree, not `mvps-infra/azure/managers`.

## Milestones and cadence

| Milestone | Deliverable | Review |
| --- | --- | --- |
| Fri | Specs fixed + design/test/implementation docs | Olivier reviews over weekend |
| Mon | Discuss review; start Phase 0–1 | Team |
| Mid-week | resource-group template + first diff test green | Prem + Sachin |
| Late week | swarm composite reaches zero-diff against golden tree | Olivier |
| Early next week | MyFiani staging swarm provisioned | Olivier |

## Definition of done

- Every implemented generator has a passing golden-tree diff test (zero differences).
- Composition tests confirm shared questions are asked once and the swarm composes the correct set.
- CI (lint + tests) is green.
- The unreviewed iac drafts are reconciled against the production tree, not `mvps-infra`.
