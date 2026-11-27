# Testing plan — generator-dzangolab-azure

## Status

Draft, not yet reviewed. Mirrors the test approach of [generator-dzangolab-iac](https://github.com/dzangolab/generator-dzangolab-iac), which uses Yeoman's `helpers` test runner plus a per-generator `__tests__` file.

## Goal

Prove two things: (1) each generator reproduces the golden tree with no diff, and (2) the composition model asks shared questions once and delegates correctly.

## Test layers

### 1. Golden-tree diff tests (the acceptance test that matters most)

For each single-resource generator, take the existing project's answers from the production tree and assert the generator's output is byte-identical.

- Seed answers from the existing project under `devops/infrastructure/swarm/azure/<resource>`.
- Run the generator into a temp directory with `helpers.run(...)`.
- `diff -r` the temp output against the existing project; expect no differences apart from nothing.
- If the golden tree is not vendored into the test fixtures, fetch/checkout the reference tree into `__tests__/fixtures/` (read-only).

Covered generators: resource-group, vnet, static-ip, disks, nfs, key-vault, load-balancer, secrets, swarm-leader, swarm-managers, swarm-workers, entra, and (once specified) pulumi-azure-backend and ansible.

### 2. Composition tests

- **Shared answers asked once.** Run the root `app` generator end to end and assert the four shared questions are prompted exactly once, not once per subgenerator.
- **Root delegates.** Choosing each list entry composes the correct subgenerator (`composeWith` is called with the matching generator class and path).
- **Swarm composes the right set.** Run `swarm` and assert it composes resource-group, vnet, static-ip, disks, nfs, key-vault, load-balancer, secrets, swarm-leader (plus managers/workers when chosen) and the Ansible folder; assert it does **not** compose entra or pulumi-azure-backend.
- **No double prompts.** A question shared by several projects (e.g. region) is asked once for the whole collection.

### 3. CLI grouped-list tests

- The list renders the groups Docker, Pulumi, Other in that order.
- Docker generators appear in order swarm, leader, managers, workers, ansible.
- A group with no available generator is hidden.
- No `undefined` group heading appears, and the "`list` prompt is deprecated" warning is not reproduced (the iac bug this package fixes).

### 4. Per-generator unit tests

- `_optionOrPrompt` uses a provided option value when present, and prompts when absent.
- Folder-name and prefix normalisation (lowercase, non-alphanumerics to `-`) matches iac.
- `createStackConfig` emits `Pulumi.<environment>.yaml`; when false, it does not.
- Hidden files (`.env.example`, `.gitignore`) are emitted via dot-glob.

## Fixtures and tooling

- Jest (or the same test runner as iac) with `yeoman-test` helpers.
- Fixture directory holds the golden tree snapshots and the answer fixtures per generator.
- CI runs lint (biome), then the full test suite on every push.

## Acceptance for this plan

A generator is done when its golden-tree diff test passes with zero differences, its composition test passes, and its unit tests are green.
