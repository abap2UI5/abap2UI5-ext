# AGENTS.md — AI Assistant Guide for abap2UI5-ext

## What this repository is

The legacy utility classes of abap2UI5 (`z2ui5_cl_util*`,
`z2ui5_cx_util_error`, table `z2ui5_t_91`), moved here from the core's frozen
package `src/99/01` so the core no longer ships them. Systems whose code (or an
addon) still uses them install this repository next to abap2UI5 and keep
working unchanged.

**Language:** English for code, comments, commits, PRs and docs.

## Rules

- **The code is frozen.** It is the exact `src/99/01` of abap2UI5 at the time
  of the move. Do not refactor, extend or restyle it; fix only what stops it
  from activating against a current abap2UI5 release. New utilities belong in
  [abap-util](https://github.com/abap-util/abap-util), never here.
- **Dependency on the core:** these classes call `z2ui5_cl_ajson`,
  `z2ui5_cl_srt_*` and public types/constants of `z2ui5_cl_ui5_util_context`
  (all in the core's `src/00`). The abaplint configs load only `src/00` of a
  pinned release, because releases up to 1.148.0 still contain `src/99/01`
  themselves and would define every object twice.

## Validation

`npm run check` runs what CI runs: abaplint Standard (`abaplint.jsonc`) and
ABAP Cloud (`.github/abaplint/abap_cloud.jsonc`). `publish-702` writes the
downported branch `702` on every push to `main`, `abap-702` lints it.
All `.abap`/`.xml`/config files are LF-only (`.gitattributes`).
