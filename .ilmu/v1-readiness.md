# v1.0.0 Readiness Checklist

## End-goal statement

v1.0.0 = drop-in replacement for Spectral OAS linting in teams using declarative `.spectral.yaml` configs. Team running `spectral lint spec.yaml` today can swap binary for `refract spec.yaml`, keep existing `.spectral.yaml` unchanged (custom declarative rules via `given`/`then.function`/`message`, all built-in Spectral functions), see equivalent violations. JS custom function rules: permanent exclusion, documented with migration guidance. `spectral:asyncapi`: out of scope. Parity verified by corpus of real-world specs run through both tools.

---

## Tier 1 — Hard blockers

Must ship before v1.0.0. Any open item kills the drop-in claim.

- [ ] **Custom rule definitions in `.spectral.yaml`** — `given` (JSONPath), `then.function`, `message`, `then.functionOptions`. No ADR yet; ADR-008 deferred. Needs new ADR: YAML schema, JSONPath dialect (see below), function dispatch. Largest remaining build item.
- [ ] **JSONPath dialect ADR** — Must precede custom-rule impl. Spectral uses Nimma (Goessner-draft + extensions: `~` for keys, `@property` in filters). RFC 9535 diverges. ADR must pin crate and document supported subset. Without it: same `.spectral.yaml`, different violation sets, silent.
- [ ] **Declarative function library** — `truthy`, `falsy`, `pattern`, `length`, `alphabetical`, `casing`, `enumeration`, `defined`, `undefined`, `schema`, `unreferencedReusableObject`, `xor`. `schema` reuses boon (ADR-022). `unreferencedReusableObject` needs reference graph from ADR-027. `casing` needs exact case-style vocabulary match with Spectral (`camel`, `pascal`, `snake`, `kebab`, `flat`, `macro`). Audit against pinned Spectral version, not memory. Folds into custom-rule ADR.
- [ ] **Multiple `extends` inheritance chains** — Merge semantics: later rulesets override earlier, severity overrides compose, rule additions accumulate. Needs ADR covering edge case where parent defines rule, child overrides to `off`, later sibling re-enables. No ADR exists.
- [ ] **Remote `extends` URL resolution — permanently forbidden** — ADR-023 bans HTTP `$ref`: refract is local CI linter, no network I/O, CI reproducibility. Remote extends strictly worse (pulls executable policy from network at lint time, supply-chain risk). New ADR mirrors ADR-023, emits `LintError::HttpExtendsNotSupported`, documents vendoring as supported pattern.
- [ ] **`aliases` and `formats` in ruleset YAML** — `aliases` fold into custom-rule ADR (substitute-then-parse pass). `formats` field must map to `OasVersion` detection; current `oas2-`/`oas3-` prefix convention insufficient for rules scoped to `oas3_0` vs `oas3_1`. Scope in custom-rule ADR.
- [ ] **Unknown-key policy in `.spectral.yaml`** — Silent ignore worst (config appears to work, silently degrades). Hard error blocks migration. Preferred: warn per unknown key + `--strict` flag promotes to error. Needs decision and tests. Folds into ruleset parsing.
- [ ] **OAS 3.1 `$ref` siblings false positive** — ADR-023 documents as known limitation, sketches fix (merge siblings from original `$ref` object onto inlined node after eager pre-pass; siblings win per OAS 3.1 spec). Needs own ADR and scheduled build. Currently documented as known gap in README.
- [ ] **`oas3-unused-component` rule** — Deferred to v0.6.0 per ADR-027. Requires reference graph module covering `$ref`, `discriminator.mapping`, `security`, `operationRef`, nested refs, callbacks. v1.0.0 depends on v0.6.0 shipping this with correct traversal; do not inherit Spectral's documented false-positive history.

---

## Tier 2 — Compatibility edge cases

In scope for v1.0.0. Tighten parity but don't individually block drop-in claim if Tier 1 complete.

- [ ] **Parity test corpus** — Non-negotiable. Harness in `tests/parity/` runs same real-world OpenAPI specs (~50 fixtures) through Spectral (pinned Node version) and refract, asserts violation-set equivalence: rule ID, severity, message text. Only external verification of drop-in claim. Unit tests prove rules work in isolation; corpus proves they work as Spectral would. Cannot defer to v1.0.1.
- [ ] **Error message wording aligned with Spectral** — CI log grep patterns and dashboards may key on message text. Message parity is subset of parity corpus assertion; corpus harness flags every drift automatically.
- [ ] **Per-rule severity overrides and `off`/`recommended: false`** — Supported in `.spectral.yaml` parsing since v0.1.0 per ADR-008. Verify full coverage and document in compatibility table.
- [ ] **`resolved` vs `unresolved` rule targeting** — Spectral rules declare which document form they run against. Needs explicit decision: does refract always run on resolved doc (current default via eager pre-pass), and how does that interact with custom rules specifying `resolved: false`? Document and test.
- [ ] **`except` paths in ruleset YAML** — Spectral supports per-file or per-JSONPath suppression of specific rules via `except`. Needs explicit in/out decision for v1.0.0.

---

## Tier 3 — Permanent incompatibilities

Deliberate permanent exclusions. Each needs clear docs and migration guidance so users don't silently misconfigure.

- **JavaScript custom function rules** — refract executes rules as native Rust, cannot load or run JavaScript. Rules using `functions/*.js` or JS-only Spectral functions won't run. Migration: if expressible with `pattern`, `schema`, `casing`, `length`, or another declarative function, port it. If requires arbitrary JS, keep Spectral for that ruleset or extract logic into shape refract can represent. README gets dedicated section listing Spectral's JS surface and declarative equivalents.
- **`spectral:asyncapi` ruleset** — refract v1.0.0 targets OpenAPI only. AsyncAPI support not planned. Users needing AsyncAPI linting should keep Spectral for that purpose.
- **Remote `extends` URLs** — refract runs offline by design for CI reproducibility. Vendor external rulesets locally or pull via git submodules or package manager at checkout. Same policy as HTTP `$ref` (ADR-023).

---

## Operational gates

Non-rule changes that must land before v1.0.0 ships.

- [ ] **ADR-025: GHA Node.js 24 migration** — Not yet merged as of v0.5.0 scoping. Deadline June 2026. Ship as standalone chore PR before v0.5.0 build opens (ADR-025 already accepted).
- [ ] **README "When to use what" rewrite** — Current wording narrows to "teams with `.spectral.yaml` they do not want to touch." After Tier 1 custom-rule work ships, rewrite to reflect drop-in claim with explicit scoping of JS exclusion.
- [ ] **Remove "Renamed from openapi-linter" README breadcrumb** — Still present in README line 3. Remove as part of v1.0.0 prep.
- [ ] **crates.io publish under `refract` name** — Cargo.toml package is `refract-cli`, binary is `refract`. Not yet published to crates.io. Needs ADR: publish as `refract` (rename), `refract-cli` with thin `refract` proxy crate, or something else. Resolve before v1.0.0.
- [ ] **Performance benchmark vs Spectral** — Cold-start and steady-state on representative specs. Build alongside parity corpus (same fixture set). Publish numbers in v1.0.0 release notes.

---

## How to use this document

Before scoping any milestone from v0.6.0 onward, check whether candidate work serves a checkbox above or is explicit v1.x deferral. Work not addressing a checkbox and not an acknowledged v1.x deferral needs justification before entering the plan. ADR-029 records the decision defining this document's scope.