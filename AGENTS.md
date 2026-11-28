# AGENTS.md — AI Assistant Guide for abap2UI5-ext

## What this repository is

The legacy code of abap2UI5 that the core no longer ships. A system whose
code - or an addon - still uses one of these objects installs this
repository next to abap2UI5 and keeps working unchanged.

One object of the core's `src/99` did not come along and must not be added:
`z2ui5_cl_http_handler`, the empty subclass that kept the former handler
name alive, was deleted (maintainer decision 2026-11-17). What named it
names `z2ui5_cl_ui5_http_handler` now.

| Package | What | Came from |
|---|---|---|
| `src/` | the utility classes `z2ui5_cl_util*`, `z2ui5_cx_util_error`, table `z2ui5_t_91` | the core's `src/99/01` (2026-11-02), by way of abap2UI5/abap2UI5-ext |
| `src/01` | the XML view builder `z2ui5_cl_xml_view`, `z2ui5_cl_xml_view_cc`, `z2ui5_if_types` | the core's `src/99` (2026-11-17) |
| `src/02` | the seventeen popups `z2ui5_cl_pop_*` | the core's `src/99/02` (2026-11-17) |

History: the utilities were abap2UI5/abap2UI5-ext (2026-11-02), all three
packages then lived in `abap2UI5/frontend` (2026-11-17 to 2026-11-25), which
is retired - this repository holds them from 2026-11-25 on, with the same
object names. `abap2UI5/frontend` had been a **delivery repository** before
(branches `standard`, `cloud`, ... with the ICF node, the BSP and the cloud
service); those objects are part of abap2UI5 itself now (its `src/03` and
its generated branch `cloud`), nothing of them belongs here.

**Language:** English for code, comments, commits, PRs and docs.

## Rules

- **The code is frozen.** It is the core's `src/99` at the time of each move
  (it differs in five places, listed below). Do not refactor,
  extend or restyle it; fix only what stops it from activating against a
  current abap2UI5 release. New utilities belong in
  [abap-util](https://github.com/abap-util/abap-util), a new popup in
  [abap2UI5-addons/popups](https://github.com/abap2UI5-addons/popups), a new
  view in `z2ui5_cl_ui5_view_builder` of the core - never here.
- **Where it differs from the core's former `src/99`** - each for
  activation, none for behaviour a caller relies on:
  - `z2ui5_cl_util_xml` reads its XML-attribute escape format from
    `z2ui5_cl_util` (the dependency rule below).
  - Four popups - `z2ui5_cl_pop_file_ul`, `_get_range`, `_get_range_m`,
    `_to_select` - no longer call `client->popup_model_update( )` (seven
    calls, 2026-11-22): the core removed the method after 1.150.0, and it
    had been empty since before 1.149.0.
  - `z2ui5_cl_util` calls the core's ajson and S-RTTI by the names they
    have since abap2UI5/abap2UI5#3093 (2026-11-28): `z2ui5_cl_ui5_ajs`,
    `z2ui5_if_ui5_ajs` and `z2ui5_cl_ui5_srt_type`, where it had
    `z2ui5_cl_ajson`, `z2ui5_if_ajson` and `z2ui5_cl_srt_typedescr`. The
    core renamed its vendored copies; this repository may not add objects,
    so it does not vendor one of its own.
  - The view builder and the popups call `z2ui5_cl_util` where they
    called a `FROZEN-ONLY` symbol of `z2ui5_cl_ui5_util_context`
    (2026-11-28, the dependency rule below) - same method, same code.
  - `z2ui5_cl_util_ext=>lock_set_wait` pauses between two tries with the
    function module `ENQUE_SLEEP`, called by name, where it had
    `WAIT UP TO` (2026-11-22): `WAIT` is no statement of ABAP Cloud, and a
    newer abaplint than the one locked here says so. The module takes
    whole seconds - `delay_ms` is rounded up, the default of 500 ms is one
    second.
- **No object is renamed and none is added.** Every name here is one an
  installation already has, from the core. The packages are `src/`, `src/01`
  and `src/02` for that reason too: the utilities stay where a system that
  installed abap2UI5-ext has them.
- **Dependency on the core, by package:**
  - `src/` calls `z2ui5_cl_ui5_ajs`, `z2ui5_cl_ui5_srt_type` and the public types
    `ty_s_name_value` / `ty_t_name_value` of `z2ui5_cl_ui5_util_context` (all
    in the core's `src/00`) - nothing marked `FROZEN-ONLY` there.
    `z2ui5_cl_util_xml` reads its XML-attribute escape format from
    `z2ui5_cl_util` for that reason - a line that differs from the core's
    former `src/99/01`.
  - `src/01` and `src/02` call the public API (`z2ui5_if_client`,
    `z2ui5_if_app`, `z2ui5_cl_ui5_view_builder`) and members of
    `z2ui5_cl_ui5_util_context` the core uses itself. What the core had
    marked `FROZEN-ONLY` - kept for exactly these classes - they call on
    `z2ui5_cl_util`, which holds the original (2026-11-28), so the core
    can drop those symbols. **Never call a member of
    `z2ui5_cl_ui5_util_context` from here that `z2ui5_cl_util` has as
    well** - the core trims that class to its own callers.
- **The abaplint configs load `src/00` to `src/02` of the core's `main`**
  (`702` for the 7.02 config), not all of `src/`: a release up to 1.149.0
  still ships these classes itself in `src/99`, and loading that would define
  every object twice. Not a release yet: the ajson and S-RTTI names
  `z2ui5_cl_util` calls exist from abap2UI5/abap2UI5#3093 on, which no
  release up to 1.151.0 has - pin the first release that has them (from
  1.151.0 to 2026-11-28 the configs pinned that tag, before the rename). The
  namespace rename (below) pairs with the core at the same pin. For the same
  reasons the classes of `src/01` and `src/02` can only be installed next to
  a core release **after 1.149.0**, and `z2ui5_cl_util` only next to one
  **after 1.151.0**.
- **The test classes moved with their objects and run on a system only.**
  In the core they ran in the transpiled unit suite; this repository lints,
  it does not transpile.

## Validation

`npm run check` runs what CI runs: abaplint Standard (`abaplint.jsonc`),
ABAP Cloud (`.github/abaplint/abap_cloud.jsonc`), the abap2UI5-linter
(`abap2ui5lint.jsonc`, workflow `abap2ui5-lint`) and the namespace rename
(`npm run rename`, workflow `check-rename`). The linter's findings at
adoption are in `abap2ui5lint-baseline.json` - frozen code is checked, not
restyled; a new finding fails, a fixed one makes its entry stale (run
`npx abap2ui5lint --update-baseline`). `publish-702` writes the
downported branch `702` on every push to `main`, `abap-702` lints it.
All `.abap`/`.xml`/config files are LF-only (`.gitattributes`).

## Namespace rename

`npm run rename` (CI: `check-rename`) and the manual `build-rename` workflow
are abap2UI5's namespace rename for this repository -
`.github/scripts/rename.mjs`, the same file in every addon that has one (change
it everywhere; never here alone). Every `z2ui5_` object becomes `<name>_...`,
for a customer name such as `ZMYUI5` or a registered namespace such as
`/CA2UI5/` (at most 10 characters), and so does every reference to abap2UI5:
the branch `rename_<name>` runs on an abap2UI5 renamed with the same name by
its own `build-rename`, which renames the core's `main` - hence the `main`
pin. An app on the legacy API that moves to a renamed abap2UI5 needs these
objects renamed with the same name. The rename touches no source here: it
writes the renamed copy to the branch. What follows from it for the frozen
code: every object name keeps at most 25 characters (a transparent table 11),
and no literal names a `z2ui5_` object.
