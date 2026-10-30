# Host contracts and boundary examples

Read this when designing host integration or moving code across packages. The requirements in `SKILL.md` are durable; the linked implementations and migration examples below are **2026-10-30 source snapshots**, not current release or installation guarantees.

## Existing contracts to study

The [Desktop application contract](https://github.com/LispMachineLab/lispm-desktop/blob/3351180c1cb3123c84366b752a638a44dda2f81a/README.org#L418-L458) assigns entrypoints, public status queries, and command declarations to the app. Desktop skins and adapters consume those interfaces from the outside.

[Agent's host callbacks](https://github.com/LispMachineLab/lispm-agent/blob/62fd3d3036f39be0f9feb0ecc481a93baa1caca7/lisp/lispm-agents.el#L84-L126) illustrate useful defaults: `lispm-agents-show-function` uses `pop-to-buffer`, `lispm-agents-main-window-function` uses `selected-window`, and terminal callbacks use Eat/process operations. [Desktop supplies its implementations](https://github.com/LispMachineLab/lispm-desktop/blob/3351180c1cb3123c84366b752a638a44dda2f81a/lisp/lispm-desktop-ai.el#L157-L170). The app does not probe for Desktop.

[Stickies' presentation callbacks](https://github.com/LispMachineLab/lispm-stickies/blob/15f6496f0ab322d0152f4da4acdef2870c8271e4/lispm-stickies.el#L275-L349) cover showing, hiding, visibility, the front note, placement, and change events. Ordinary Emacs windows provide the defaults; app data and operations stay in Stickies. Use these as contract examples, not a requirement to copy every hook into every app.

When adding a callback, specify its arguments/result, which buffer or session it concerns, when it runs, and who owns resources. An adapter should install once, remove its own subscriptions on disable, and restore previous callback values only while its own values remain installed. Include failed initialization and repeated enable/disable in lifecycle checks when relevant.

## Agent project views and Office

Reuse [Agent project/group semantics](https://github.com/LispMachineLab/lispm-agent/blob/62fd3d3036f39be0f9feb0ecc481a93baa1caca7/lisp/lispm-agents.el#L1758) and the public [roster API](https://github.com/LispMachineLab/lispm-agent/blob/62fd3d3036f39be0f9feb0ecc481a93baa1caca7/lisp/lispm-agents-roster.el#L268-L284). `lispm-agents-group-by-directory` groups by project despite its name. A directory filter and a project group containing subdirectories/worktrees answer different questions. Preserve identity keys and relationship/status semantics; do not reconstruct them from rendered labels or eagerly read transcripts for list views.

For project-view changes, assign responsibilities explicitly:

- Agent owns scope queries, renderer selection/extension contracts, subscriptions to domain changes, and a usable default project/list view.
- Office implements an optional renderer using Agent's public contract.
- Desktop translates sidebar actions into Agent commands and owns folding, hit regions, and window/workspace placement.

The design in [Desktop #167](https://github.com/LispMachineLab/lispm-desktop/pull/167) is a **boundary counterexample**: scope/filtering, fallback roster logic, renderer dispatch, subscriptions, and the default list were placed in Desktop. Treat it as historical design evidence, not an approved integration pattern or a current-state claim.

Office's inspected production code registers a [Desktop project-view hook](https://github.com/LispMachineLab/lispm-office/blob/be4b864a9ce5b330f43fb47b951f7fd5213b8582/lispm-office-plugin.el#L118), references [Desktop tab drawing](https://github.com/LispMachineLab/lispm-office/blob/be4b864a9ce5b330f43fb47b951f7fd5213b8582/lispm-office.el#L124), and sets a [Desktop buffer marker](https://github.com/LispMachineLab/lispm-office/blob/be4b864a9ce5b330f43fb47b951f7fd5213b8582/lispm-office.el#L1035). Plain-Emacs entrypoints and guards do not cure these source-level boundary violations. Move the extension contract to Agent, generic tab controls to public UI APIs, and host adaptation to Desktop.

## Reuse still requires a public API

[ComfyUI's form internals access](https://github.com/LispMachineLab/lispm-comfyui/blob/1e3c762678310f5e2b975d7275caff44034b43ae/lispm-comfyui-simple.el#L393) illustrates cross-package coupling through `lispm-ui-form--*`. A needed public form-update operation belongs in UI; callers should migrate to it. This is source evidence of coupling, not a demonstrated runtime failure.

The shared-UI consumer migrations [Desktop #169](https://github.com/LispMachineLab/lispm-desktop/pull/169) and [#170](https://github.com/LispMachineLab/lispm-desktop/pull/170) were pending at the research snapshot. [Office #1](https://github.com/LispMachineLab/lispm-office/pull/1) and [Desktop #171](https://github.com/LispMachineLab/lispm-desktop/pull/171) had merged. Recheck status before continuing any migration; older Desktop duplicates do not establish the right ownership for new code.

## Evidence and validation limits

[Agent's no-Desktop source guard](https://github.com/LispMachineLab/lispm-agent/blob/62fd3d3036f39be0f9feb0ecc481a93baa1caca7/test/lispm-test.el#L9415-L9422) and [Store's production dependency guard](https://github.com/LispMachineLab/lispm-store/blob/1a41fa5550c08ccecd2760dcb35052b72d3f5474/test/lispm-store-test.el#L274-L292) support the stronger boundary than “does not require Desktop at load time.” Guards supplement clean-Emacs functional tests; they cannot prove ordinary-Emacs usability, optional-plugin behavior, teardown, or appearance.

Test-only comparisons may legitimately load Desktop, while compatibility aliases may remain in the old Desktop adapter. Inspect the role of the code instead of classifying every textual match as a production dependency. The source survey behind these examples inspected 24 active code repositories; it did not run their apps or certify their runtime independence.
