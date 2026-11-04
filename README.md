# lispM-dev-skill

A reusable development skill for LispM apps, Agent, shared libraries, and Desktop integrations. It guides ownership before implementation: apps work in ordinary Emacs, Desktop stays a thin host, and common mechanisms use existing public library APIs.

The repository name is `lispM-dev-skill`; the skill identifier is `lispm-dev-skill`.

The [repository map](references/repositories.md) covers all **30 public LispMachineLab repositories** verified on 2026-11-05: 3 core/distribution repositories, 8 shared libraries/runtimes, 14 apps/extensions, and 5 documentation, development, or historical repositories. Each development entry identifies its responsibility, dependency boundary, and source starting point. A change-routing table shows which related repositories to inspect together.

## Use

Point a skill-capable coding agent at [SKILL.md](SKILL.md), or explicitly ask it to follow that file while developing or reviewing a LispM change. Keep the `references/` directory alongside it. For example:

> Follow SKILL.md in this checkout to add an Agent project view. Identify ownership, existing public APIs, and plain-Emacs validation before implementing it.

If you choose to install it in an agent's skill directory, use `lispm-dev-skill` as the installed folder name and follow that agent's installation mechanism. This repository does not install itself or change Emacs/global configuration. Its Markdown development skill is distinct from LispM Store's executable Org skill format.

## Contents

- [SKILL.md](SKILL.md): ownership, host contracts, API reuse, lifecycle, migrations, and validation.
- [Repository map](references/repositories.md): the complete organization inventory, app dependencies, source entrypoints, and cross-repository routing.
- [Library map](references/libraries.md): where shared mechanisms belong and how the runtimes differ.
- [Boundary examples](references/boundaries.md): public contracts, counterexamples, and pinned source evidence.
- [LICENSE](LICENSE): GNU Affero General Public License, version 3 or later, following LispM's library convention.

Durable requirements are separated from implementation observations dated 2026-11-05. Recheck current APIs and delivery state before acting on the pinned examples. The skill supports scoped development; it is not an automatic organization-wide audit or a claim that all apps passed runtime testing.

For structural validation, run the skill-creator bundle's `scripts/quick_validate.py` against this checkout. Behavioral review should also check whether realistic requests preserve ownership, standalone functionality, and task scope.
