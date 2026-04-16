# Archived milestones: .ilmu/archive/
# v0.1.0: .ilmu/archive/plan-v0.1.0.md (complete)
# v0.2.0: .ilmu/archive/plan-v0.2.0.md (complete)
# v0.3.0: .ilmu/archive/plan-v0.3.0.md (complete)
# v0.4.0: .ilmu/archive/plan-v0.4.0.md (complete)

---

# v0.5.0 Plan

## Hypothesis

6 new Spectral OAS parity rules (4 OAS 2.x structural + 2 media-level example validation) close remaining Spectral OAS gap except graph-analysis rules. Teams replacing Spectral get near-complete coverage with only `oas3-unused-component` remaining, deferred to v0.6.0 pending reference-graph infrastructure.

## Scope

6 new rules (see ADR-026):

| Rule | Phase | OAS version | Deref | Boon |
|------|-------|-------------|-------|------|
| oas2-parameter-description | 1 | 2.x | yes | no |
| oas2-api-schemes | 1 | 2.x | no | no |
| oas2-anyOf | 1 | 2.x | no | no |
| oas2-oneOf | 1 | 2.x | no | no |
| oas3-valid-media-example | 2 | 3.x | yes | yes |
| oas2-valid-media-example | 2 | 2.x | yes | yes |

Parallel chore if not yet merged: GHA Node.js 24 migration (ADR-025).

## Out of scope

- `oas3-unused-component`: deferred to v0.6.0 (ADR-027). Needs reference graph module.
- `Deref'd<'a>` newtype: deferred again despite >8 trigger (ADR-028). Zero observed bugs.
- Workspace extraction: no consumer trigger (ADR-024 continues).
- HTTP `$ref` support: rejected (ADR-023).
- Spectral custom rule format: out of scope all v0.x.

## ADRs

- ADR-026: v0.5.0 rule set (6 rules)
- ADR-027: `oas3-unused-component` deferred to v0.6.0
- ADR-028: Deref'd<'a> newtype deferred again (overrides ADR-021 >8 trigger)
- ADR-025: GHA Node.js 24 migration (pre-existing, standalone chore)

---

## Build branches

Integration branch: `build/v0.5.0` off `main`.
Phase branches: `phase1/v0.5.0`, `phase2/v0.5.0`.
Phase 1 branches from `build/v0.5.0`. Phase 2 branches from `phase1/v0.5.0`.
Each phase opens PR targeting `build/v0.5.0`.
Optional chore PR for ADR-025 opens against `main` independently.

---

## Phase 1: OAS 2.x structural rules

**PR:** `phase1/v0.5.0` -> `build/v0.5.0`

4 rules. Zero new dependencies. No shared helpers beyond existing `OasVersion::detect` and `resolve_ref`.

### 1. `oas2-parameter-description`

**File**: `src/rules/oas2_parameter_description.rs`

Mirror of `oas3_parameter_description`. Version gate: return early unless `OasVersion::V2`.

Algorithm:
- Walk `paths.*.parameters` and `paths.*.<method>.parameters`.
- For each parameter node, call `resolve_ref` if `$ref` present.
- Flag violation if dereferenced node's `description` missing or empty string.
- Source location at original parameter node (not deref target).

Severity: `warn`. Rule id: `oas2-parameter-description`. Message: "Parameter must have non-empty description."

### 2. `oas2-api-schemes`

**File**: `src/rules/oas2_api_schemes.rs`

Version gate: V2 only.

Algorithm:
- Read top-level `schemes`.
- Violation if absent, not array, or empty array.
- For each string entry, violation if value not in `{"http", "https", "ws", "wss"}`.

Severity: `warn`. Rule id: `oas2-api-schemes`. Not deref-dependent (top-level scalar/array).

### 3. `oas2-anyOf`

**File**: `src/rules/oas2_any_of.rs`

Version gate: V2 only.

**Pre-implementation check (required before writing rule logic):** Feed Swagger 2.0 fixture with `definitions/X/anyOf: [...]` through existing `oas2-schema` rule. If `oas2-schema` already emits violations for `anyOf`, evaluate whether `oas2-anyOf` needs narrowed scope (better message, lower severity) or can defer. If `oas2-schema` silent, proceed as planned. Document finding in PR description.

Algorithm:
- Walk whole document.
- On any object containing key `anyOf`, emit violation ONLY if object is schema-shaped: must contain at least one of `type`, `properties`, `allOf`, `oneOf`, `anyOf`, `items`, `$ref` (same `has_schema_key` check as `oas3_valid_schema_example`). Gate prevents false positives on example payloads or structural OAS objects with literal `anyOf` key.
- Do NOT deref into `$ref` targets. `anyOf` in referenced schema reported at referenced schema's definition, matching Spectral.

Unit test (required): Swagger 2.0 spec with `examples` payload containing literal `anyOf` key must produce zero violations.

Severity: `error`. Rule id: `oas2-anyOf`. Message: "anyOf keyword is not valid in OAS 2.x schemas."

### 4. `oas2-oneOf`

**File**: `src/rules/oas2_one_of.rs`

Identical shape to `oas2-anyOf`, including pre-implementation check against `oas2-schema` and `has_schema_key` gate. Key searched: `oneOf`. Rule id: `oas2-oneOf`. Severity: `error`.

### Registration

Add 4 entries to `default_registry()` in `src/rules.rs`, alphabetically positioned.

### Test fixtures

Each rule gets unit tests in its own file (pattern from v0.3.0 rules):
- Passing fixture.
- Missing-field / wrong-version / violating-keyword fixture.
- OAS version gating check (V3 spec = no violations for oas2-* rules).

Integration test fixture: `tests/fixtures/v0.5.0/oas2-*.yaml`, one per rule.

### PR gate

Phase 1 PR merges only after all 4 rules pass unit + integration tests + CI checks (fmt, clippy, test, audit, deny, doc). Use `gh pr checks --watch --fail-fast` per CLAUDE.md.

---

## Phase 2: Media-level example validation

**PR:** `phase2/v0.5.0` -> `build/v0.5.0` (depends on Phase 1 merge)

2 rules. Reuse v0.4.0 boon pattern from `oas3_valid_schema_example.rs`.

### 5. `oas3-valid-media-example`

**File**: `src/rules/oas3_valid_media_example.rs`

Version gate: V3_0 or V3_1.

Scope: MediaType objects, Parameter objects, Header objects. All three have `example` / `examples` alongside sibling `schema` at same level (ADR-026). Matches Spectral's coverage, broader than MediaType-only.

- MediaType objects: `paths.*.<method>.requestBody.content.*` and `paths.*.<method>.responses.<status>.content.*`.
- Parameter objects: `paths.*.parameters` and `paths.*.<method>.parameters`. Deref `$ref` before inspecting.
- Header objects: `paths.*.<method>.responses.<status>.headers.*`. Deref `$ref` before inspecting.

MediaType detection within content maps: object contains `schema` key AND does NOT contain schema-shape keys (`type`, `properties`, `items`, `allOf`, `oneOf`, `anyOf`, `$ref`). Distinguishes MediaType from Schema with embedded `schema` property.

Algorithm per node (MediaType, Parameter, or Header):
- Resolve `schema` if `$ref`.
- If `example` present: validate against schema.
- If `examples` (object map) present: for each entry, if `value` present, validate; skip `externalValue`.
- Reuse `strip_example_keys` (hoist to `src/rules/util.rs` if both media-example rules share it, or duplicate once).
- Truncate at 64 violations, matching ADR-022.

Reuse helpers from `oas3_valid_schema_example`: `validate_example`, `strip_example_keys`, `collect_leaves`. Hoist to `src/rules/util.rs` on this PR (expected util.rs growth: ~60 lines, still under 300-line split threshold).

Severity: `error`. Rule id: `oas3-valid-media-example`.

### 6. `oas2-valid-media-example`

**File**: `src/rules/oas2_valid_media_example.rs`

Version gate: V2 only.

OAS 2.x places examples differently:
- Parameter Object: no top-level `example`, but `x-example` (extension) or `schema.example`. Rule targets documented `examples` map on Response Objects (OAS 2.x Response.examples is `{mime-type: value}` map).
- Response Object `examples` map: keys are media types, values are literal example values. Validate each against response's `schema`.

Algorithm:
- Walk `paths.*.<method>.responses.<status>` objects.
- Deref response if `$ref`.
- If response has both `schema` and `examples`: for each example entry, validate value against schema using boon.
- Skip if `schema` or `examples` missing.

Severity: `error`. Rule id: `oas2-valid-media-example`.

### Registration

Add 2 entries to `default_registry()`, alphabetically.

### Shared helper hoist

On Phase 2, move `validate_example`, `strip_example_keys`, `collect_leaves` from `oas3_valid_schema_example.rs` to `src/rules/util.rs`. All three rules (original v0.4.0 rule plus 2 new) consume from util. Signature unchanged.

### Test fixtures

Per-rule unit tests: valid example, invalid example (type mismatch), missing schema, missing examples, version mismatch. Integration fixtures: `tests/fixtures/v0.5.0/oas3-valid-media-example.yaml`, `oas2-valid-media-example.yaml`.

### PR gate

Same as Phase 1. All checks green before merge.

---

## Final integration PR

`build/v0.5.0` -> `main`. Opens after Phase 2 merges. Description lists all 6 new rules, links ADR-026 / ADR-027 / ADR-028, notes ADR-025 chore status (merged or pending). CI matrix runs on this PR. Leave for human review, do not merge.

---

## Out-of-PR chore

ADR-025 (GHA Node.js 24 migration) pending as of v0.5.0 scope. If not merged before `build/v0.5.0` opens, developer agent opens standalone PR against `main` in parallel with Phase 1. Merge order does not block build branch.