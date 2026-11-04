---
name: lispm-dev-skill
description: Develop or refactor LispM apps, Agent, shared libraries, and Desktop integrations with ordinary-Emacs support, clear ownership, and public APIs. Use for LispM implementation and architecture reviews; keep investigation and validation proportional to the requested change.
license: AGPL-3.0-or-later
---

# LispM development

Every LispM app must work in ordinary Emacs without LispM Desktop, with its legitimate declared domain dependencies and supported capabilities. An Agent plugin may require Agent; native graphics may require a supported Emacs build. Neither permits a Desktop dependency. Desktop's hidden-host “standalone applications” feature does not establish this independence.

## Repository scope

Use the [complete repository map](references/repositories.md) to locate owners, dependencies, public source entrypoints, and related consumers before choosing where a change belongs. The map covers all 30 public organization repositories verified on 2026-11-05:

| Area | Repositories |
| --- | --- |
| Core and distribution | `lispm-agent`, `lispm-desktop`, `lispm-store` |
| Shared libraries and runtimes | `lispm-kit`, `lispm-ui`, `lispm-auth`, `lispm-prebuilt`, `lispm-kernel`, `lispm-security`, `lispm-gl`, `emacs-cl` |
| Applications | `lispm-1password`, `lispm-browser`, `lispm-comfyui`, `lispm-doom`, `lispm-mail`, `lispm-mycin`, `lispm-remote`, `lispm-rotom`, `lispm-shrdlu`, `lispm-stickies` |
| Agent extensions and renderer | `lispm-hermes`, `lispm-holon`, `lispm-openclaw`, `lispm-office` |
| Documentation and development | `LispMachineLab.github.io`, `.github`, `lispM-dev-skill` |
| Archived or historical sources | `lispm-eseq`, `www-shrdlu` |

Select the relevant rows and inspect those repositories; listing the whole ecosystem does not require auditing it for each task. Use the map's change-routing table when a feature crosses repositories. Recheck the organization inventory when maintaining this index or when the requested component is absent.

## Decide ownership before implementation

Identify the behavior's owner, the public API involved, and the affected consumers before editing. Inspect current source, package headers, actual `require` and call sites, extension points, and relevant tests. Repository descriptions and installed checkouts can lag source. Investigate the affected dependency path; a small app change does not require auditing every repository.

| Owner | Responsibility |
| --- | --- |
| App | Domain data, state, queries, validation, commands, and resource lifecycle. |
| Agent | Agent identity, sessions, status, collaboration, project/directory scope, and the default project view. The repository is `lispm-agent`; its package name is `lispm`. |
| Shared library | Reusable mechanisms within its established responsibility. Research its public API before inventing an abstraction. |
| Desktop | Windows/workspaces, sidebar hit regions and folding, desktop appearance, distribution, and host adapters. |
| Office | An optional Agent renderer. It must not own or gate Agent's default project/list view. |
| Store | Catalog and install metadata, including dependency ordering and pinned revisions. It is not an app runtime library. |

The location of a clicked control does not determine ownership. A Desktop sidebar action can invoke an Agent command; it must not reimplement Agent's scope, roster, or fallback view.

## Keep the host outside the app

- Production app and shared-library code must not reference Desktop APIs, including Desktop-specific Platinum APIs. `fboundp`, `defvar`, and `declare-function` guards still leak the host contract. Avoid direct and transitive Desktop runtime dependencies.
- Provide an autoloaded ordinary-Emacs entry command that opens and operates the app. Define public host-neutral callbacks, hooks, generic methods, or metadata in the owning package, with useful defaults such as `pop-to-buffer`, `selected-window`, Eat, or ordinary processes.
- Document callback arguments, results, scope, state/resource ownership, and teardown. Desktop injects adapters externally; the app must not detect Desktop and patch itself to fit it.
- Keep status predicates and command behavior in the app. Declare keys, menus, enable conditions, and button metadata once with `lispm-kit-define-commands`; hosts consume the declarations.
- Use public cross-package APIs. Calling another package's `--` internals remains coupling even when that package is a common library. Add the needed public operation in its owner before migrating consumers.

For concrete host contracts and boundary mistakes, read [boundary patterns](references/boundaries.md) when designing or moving an integration.

## Reuse the appropriate library

Use **lispm-kit** for command metadata and i18n; **lispm-ui** for common controls, forms, dialogs, tabs, fields, and rendering; **lispm-auth** for password-encrypted secret storage and per-app startup password declarations; **lispm-prebuilt** for native artifact lifecycle; **lispm-kernel** for background work; **lispm-security** for inspection and policy mechanisms; **lispm-gl** for supported GPU canvases; and **emacs-cl** for its Common Lisp runtime. The correct name is **prebuilt**, not prebuild.

Read the relevant entry in the [library map](references/libraries.md), then inspect the current public API and consumers. Do not add every library as a dependency or turn Kit, UI, or Agent into a miscellaneous utility package.

Distinguish `Package-Requires`, actual `require`/call dependencies, optional integrations, native capabilities, and Store `:requires`/`:rev`. Declare required package versions honestly; historical omissions are not a pattern to repeat. An optional accelerator must not silently become a hard prerequisite.

## Preserve semantics and lifecycle

Reuse Agent's public identity, roster, status, event, and scope APIs. Do not infer identity from labels or duplicate relationship logic in a renderer. Keep **exact-directory** selection distinct from **project/worktree** grouping; name the intended scope and test subdirectories, worktrees, and descendant sessions where relevant.

Own subscriptions, hooks, timers, processes, native resources, and buffers explicitly. Close, disable, unload, cancellation, and failed initialization must clean up what they created. Preserve other views and later user overrides; restore a callback only if the adapter still owns its installed value. Avoid duplicate subscriptions on reopen and callbacks that revive closed buffers. Shared background work must not require Desktop scheduling to deliver results.

For migrations, implement the lower-level domain/library API first, migrate consumers next, and adapt Desktop last. Keep necessary compatibility at the old boundary, not in the independent core. Migrate generic methods and dispatch deliberately; a function alias alone does not migrate a generic's methods. Update minimum package dependencies and applicable Store install metadata/pins when delivering those components. Stay within the authorized repository scope; identify dependent follow-up work rather than silently editing unrelated repositories.

## Validate the changed behavior

For behavior, dependency, or host-boundary changes, include the applicable checks below. For wording/documentation-only changes, use focused content/layout checks rather than unrelated application suites. Match validation to the touched behavior and its advertised support:

1. Run a clean `emacs -Q` with only the app and its declared dependencies on the load path, with **no Desktop on the load path**. Exercise the real entry command and useful functionality, not just `require`. For graphical behavior, use an interactive clean Emacs; batch ERT alone cannot establish that it works.
2. Exercise absence of relevant optional plugins/accelerators, particularly Office when changing Agent project views. Confirm the ordinary fallback remains usable. Test teardown and reopening, including owned observers and host callback restoration where affected.
3. Add or run focused behavior regressions, compilation checks, and source-level dependency guards appropriate to the change. Distinguish production references from test-only comparison harnesses and compatibility retained in the old host layer.
4. Test the Desktop adapter separately. Check supported GUI, Focus, and TUI behavior as applicable; perform visual verification for appearance/layout changes. Loading a graphics package does not prove native rendering works, and Desktop independence does not promise pixel graphics in a TUI.

Report the checks actually run and their limits. Static source inspection is not runtime validation. Keep implementation, PR merge, release, Store pin, and installed-runtime state separate; recheck volatile facts before relying on them. This skill guides development, not automatic certification of every LispM app.
