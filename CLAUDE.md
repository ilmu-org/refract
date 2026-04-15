# refract — Project Standards

Extends org-level standards in `../_ops/CLAUDE.md`. Read that first.

## Project

Fast OpenAPI linter. Ships as single static binary, no Node.js dependency.
Validates OpenAPI 2.x/3.x/3.1 specs against Spectral-compatible OAS ruleset.
Reads existing `.spectral.yaml` / `.spectral.yml` files for zero-migration compatibility.

## Approved idea

ilmu-org/ideas#2

## State

Read `.ilmu/state.md` before any task.
Read `.ilmu/decisions.md` (ADR index) before any architectural choice, then fetch relevant ADR files in `.ilmu/decisions/`. Completed milestone plans archived in `.ilmu/archive/`.
Read `.ilmu/plan.md` before any build task.

## Key constraints (from research)

- Spectral OAS ruleset compatibility = core migration hook — prioritise
- Target: non-Rust CI pipelines (Go, Python, Java) — binary UX matters
- Single static binary: no runtime deps, no install friction
- v0.1.0 scope cut by rust-critic — do not expand
## Writing .ilmu files and prompts

Any agent writing or updating files in `.ilmu/` (state, plan, decisions, ADRs) or creating a prompt file must run the output through the caveman compress skill before writing it to disk:

```
cd ~/.claude/plugins/cache/caveman/caveman/63e797cd753b/caveman-compress
python3 -m scripts <absolute_path_to_file>
```

Delete the `.original.md` backup after verifying compressed file looks correct.
