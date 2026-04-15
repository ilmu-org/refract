# Archived milestones: .ilmu/archive/
# v0.1.0: .ilmu/archive/plan-v0.1.0.md (complete)
# v0.2.0: .ilmu/archive/plan-v0.2.0.md (complete)

---

milestone: v0.3.0
hypothesis: >
  Teams already using Spectral replace it with refract after v0.3.0 because the
  32-rule built-in set covers their active .spectral.yaml rules and produces zero
  false positives on real-world bundled specs.

scope:
  # --- Phase 1: Structural-only rules (PR 1 of 4) ---
  # No resolve_ref calls. Version-gated rules use a temporary inline check on
  # src/model OasVersion; Phase 2 refactors call sites to detect_oas_version in util.rs.
  # Each rule lands with positive and negative fixtures. Tests pass before next step.

  - >-
    Rule: path-keys-no-trailing-slash. Severity: warn. Applies to: 2.x and 3.x.
    Check: each key in the paths object must not end with "/".
    Doc note: matches Spectral permissive default; catches trailing slash on path keys only.
    Positive fixture: tests/fixtures/path-keys-no-trailing-slash/violation.yaml
    (path "/pets/" triggers violation).
    Negative fixture: tests/fixtures/path-keys-no-trailing-slash/clean.yaml
    (path "/pets" passes clean).

  - >-
    Rule: path-not-include-query. Severity: error. Applies to: 2.x and 3.x.
    Check: each key in the paths object must not contain a "?" character.
    Positive fixture: tests/fixtures/path-not-include-query/violation.yaml
    (path "/pets?type=dog" triggers violation).
    Negative fixture: tests/fixtures/path-not-include-query/clean.yaml
    (path "/pets" passes clean).

  - >-
    Rule: path-declarations-must-exist. Severity: error. Applies to: 2.x and 3.x.
    Check: each path key must not contain empty or whitespace-only path parameter braces.
    Detection: regex \{\s*\} on the path string catches "{}" and "{ }" cases (S2 resolution).
    Positive fixture: tests/fixtures/path-declarations-must-exist/violation.yaml
    (path "/pets/{}" triggers violation).
    Negative fixture: tests/fixtures/path-declarations-must-exist/clean.yaml
    (path "/pets/{petId}" passes clean).

  - >-
    Rule: openapi-tags-uniqueness. Severity: error. Applies to: 2.x and 3.x.
    Check: the top-level tags array must not contain two objects with the same name field.
    Positive fixture: tests/fixtures/openapi-tags-uniqueness/violation.yaml
    (two tags both named "pets").
    Negative fixture: tests/fixtures/openapi-tags-uniqueness/clean.yaml
    (all tag names unique).

  - >-
    Rule: tag-description. Severity: warn. Applies to: 2.x and 3.x.
    Check: each object in the top-level tags array must have a non-empty description field.
    Positive fixture: tests/fixtures/tag-description/violation.yaml
    (tag object with no description field).
    Negative fixture: tests/fixtures/tag-description/clean.yaml
    (all tags have non-empty descriptions).

  - >-
    Rule: oas3-server-trailing-slash. Severity: warn. Applies to: 3.x only.
    Check: each server.url in the servers array must not end with "/".
    OAS version gate: skip entire rule if version check resolves to V2.
    Positive fixture: tests/fixtures/oas3-server-trailing-slash/violation.yaml
    (url "https://api.example.com/v1/" triggers violation).
    Negative fixture: tests/fixtures/oas3-server-trailing-slash/clean.yaml
    (url "https://api.example.com/v1" passes clean).

  - >-
    Rule: oas3-server-not-example.com. Severity: warn. Applies to: 3.x only.
    Check: no server.url in the servers array may use the example.com host.
    OAS version gate: skip entire rule if version check resolves to V2.
    Positive fixture: tests/fixtures/oas3-server-not-example.com/violation.yaml
    (url "https://example.com/v1" triggers violation).
    Negative fixture: tests/fixtures/oas3-server-not-example.com/clean.yaml
    (url "https://api.myservice.com/v1" passes clean).

  - >-
    Rule: no-$ref-siblings. Severity: warn. Applies to: OAS 2.x and 3.0 only, skip on 3.1.
    Format gate: if detect_oas_version returns V3_1, return empty violations immediately.
    OAS 3.1 adopts JSON Schema 2020-12 which permits $ref siblings; the rule does not apply.
    This rule is NOT in the OAS-version-gated list in ADR-021; the skip logic lives inside
    the rule check() implementation (C1 resolution).
    Scan positions: Schema Objects and Response Objects only, not every Value node.
    Check: no object that contains a $ref key may also contain sibling fields.
    Positive fixture: tests/fixtures/no-ref-siblings/violation.yaml
    (schema object with both "$ref" and a "description" sibling triggers violation).
    Negative fixture: tests/fixtures/no-ref-siblings/clean.yaml
    ("$ref" used alone with no sibling fields).

  - >-
    Rule: oas3-api-servers. Severity: warn. Applies to: 3.x only.
    Check: the document must define a non-empty top-level servers array.
    OAS version gate: skip entire rule if version check resolves to V2.
    Positive fixture: tests/fixtures/oas3-api-servers/violation.yaml
    (OAS 3.0 document with no servers key).
    Negative fixture: tests/fixtures/oas3-api-servers/clean.yaml
    (servers array with at least one entry).

  - >-
    Rule: operation-success-response. Severity: warn. Applies to: 2.x and 3.x.
    Check: each operation object must define at least one response with a 2xx status code.
    Positive fixture: tests/fixtures/operation-success-response/violation.yaml
    (operation with only 400 and 500 responses).
    Negative fixture: tests/fixtures/operation-success-response/clean.yaml
    (operation with a 200 response).

  - >-
    Rule: operation-operationId-valid-in-url. Severity: warn. Applies to: 2.x and 3.x.
    Check: if present, operationId must consist solely of URL path-segment-safe characters.
    Regex: [A-Za-z0-9\-._~:@!$&()*+,;=]+ (whitespace and non-ASCII rejected).
    Doc note in rule file: "matches Spectral permissive default; catches whitespace and
    non-URL-safe characters only" (M1 resolution).
    Positive fixture: tests/fixtures/operation-operationId-valid-in-url/violation.yaml
    (operationId "get pets list" with space triggers violation).
    Negative fixture: tests/fixtures/operation-operationId-valid-in-url/clean.yaml
    (operationId "listPets" passes clean).

  # --- Phase 2: util.rs additions (PR 2 of 4) ---
  # Adds OasVersion enum and detect_oas_version() to src/rules/util.rs.
  # Updates resolve_ref doc-comment with the deref-before-compare contract.
  # Refactors Phase 1 version-check inlines to use the new helper.
  # No new rules land in this phase.

  - >-
    Add OasVersion enum (variants: V2, V3_0, V3_1, Unknown) and
    detect_oas_version(doc: &Value) -> OasVersion to src/rules/util.rs.
    Detection order: check doc["swagger"] first, then doc["openapi"] (M2 resolution,
    behavior frozen in ADR-021).
    When detect_oas_version returns Unknown, emit one stderr line per lint run:
    "warning: OpenAPI version not recognized, version-gated rules skipped" (S5 resolution).
    Update doc-comment on resolve_ref(doc, pointer, depth) -> Option<&Value> to state the
    deref-before-compare contract: callers must invoke resolve_ref before comparing schema
    or parameter fields; if None is returned (external $ref or depth limit exceeded), treat
    the node as opaque and skip matching to avoid false positives (C2 resolution).
    Refactor Phase 1 version-check inlines in oas3-* rules and no-$ref-siblings to call
    detect_oas_version from util.rs instead of the inline checks added in Phase 1.

  # --- Phase 3: Deref-dependent rules (PR 3 of 4) ---
  # All rules call resolve_ref before comparing schema or parameter fields.
  # Each rule requires at least one fixture where the parameter or schema is a $ref
  # to a component. This is mandatory per S3 resolution.

  - >-
    Rule: array-items. Severity: error. Applies to: 2.x and 3.x.
    Check: every schema object with type "array" must define an items property.
    Deref: call resolve_ref on each schema $ref before checking for items. If
    resolve_ref returns None, skip that schema (treat as opaque, avoids false positives
    on external $ref nodes).
    Inline comment in rule file must reference ADR-021 deref-before-compare contract.
    Positive fixture: tests/fixtures/array-items/violation.yaml
    (inline schema with type "array" and no items field).
    Negative fixture: tests/fixtures/array-items/clean.yaml
    (schema with type "array" and a valid items object).
    Deref fixture: tests/fixtures/array-items/ref-violation.yaml
    ($ref pointing to a components/schemas entry that is type "array" with no items;
    must trigger a violation after resolve_ref resolves the $ref -- required per S3).

  - >-
    Rule: oas3-parameter-description. Severity: warn. Applies to: 3.x only.
    Check: every parameter object must have a non-empty description field.
    OAS version gate: skip entire rule if detect_oas_version returns V2.
    Deref: call resolve_ref on each parameter $ref before checking description. If
    resolve_ref returns None, skip that parameter.
    Inline comment in rule file must reference ADR-021 deref-before-compare contract.
    Positive fixture: tests/fixtures/oas3-parameter-description/violation.yaml
    (parameter object with no description field).
    Negative fixture: tests/fixtures/oas3-parameter-description/clean.yaml
    (all parameters have non-empty descriptions).
    Deref fixture: tests/fixtures/oas3-parameter-description/ref-violation.yaml
    ($ref to a components/parameters entry with no description; must trigger violation
    after resolve_ref -- required per S3).

  - >-
    Rule: operation-parameters. Severity: warn. Applies to: 2.x and 3.x.
    Check: after merging path-level and operation-level parameters, no two entries may
    share the same (name, in) pair.
    Merge rule (S6 resolution): operation-level parameters override path-level parameters
    when (name, in) matches. The overridden path-level copy is DROPPED from the dedup
    comparison set. Dedup comparison uses: (path-level params minus overridden copies)
    plus operation-level params. Without the drop rule, a valid override produces a
    false-positive duplicate violation.
    Deref: call resolve_ref on each parameter $ref entry before extracting (name, in). If
    resolve_ref returns None, skip that entry.
    Inline comment in rule file must reference ADR-021 deref-before-compare contract.
    Positive fixture: tests/fixtures/operation-parameters/violation.yaml
    (two inline operation-level parameters with identical name and in values).
    Negative fixture: tests/fixtures/operation-parameters/clean.yaml
    (operation-level parameter overrides path-level on matching (name, in), no duplicate
    after merge).
    Deref fixture: tests/fixtures/operation-parameters/ref-violation.yaml
    ($ref parameter in components duplicated as an inline parameter at operation level;
    must trigger violation after resolve_ref -- required per S3).

  - >-
    Rule: operation-tag-defined. Severity: warn. Applies to: 2.x and 3.x.
    Check: each string in an operation tags array must appear in the top-level tags array
    by name.
    Deref: call resolve_ref on the operation object before reading its tags array when the
    operation is reached via $ref. If resolve_ref returns None, skip.
    Inline comment in rule file must reference ADR-021 deref-before-compare contract.
    Positive fixture: tests/fixtures/operation-tag-defined/violation.yaml
    (operation tags array contains "pets", top-level tags only has "store").
    Negative fixture: tests/fixtures/operation-tag-defined/clean.yaml
    (operation tag matches a top-level tags entry by name).
    Deref fixture: tests/fixtures/operation-tag-defined/ref-violation.yaml
    ($ref to a component operation object whose tags array references an undefined tag;
    must trigger violation after resolve_ref -- required per S3).

  # --- Phase 4: Type-aware rules (PR 4 of 4) ---
  # Both rules inspect enum arrays using ADR-021 coercion semantics.

  - >-
    Rule: duplicated-entry-in-enum. Severity: error. Applies to: 2.x and 3.x.
    Check: no enum array in any schema may contain two entries equal by serde_json::Value
    PartialEq comparison.
    Positive fixture: tests/fixtures/duplicated-entry-in-enum/violation.yaml
    (enum: [1, 2, 1] triggers violation).
    Negative fixture: tests/fixtures/duplicated-entry-in-enum/clean.yaml
    (enum: [1, 2, 3] passes clean).

  - >-
    Rule: typed-enum. Severity: warn. Applies to: 2.x and 3.x.
    Check: each value in an enum array must be compatible with the schema declared type.
    Coercion semantics (ADR-021): integer and number types accept any JSON numeric Value;
    integer additionally requires fract() == 0.0 to permit YAML-coerced values such as 1.0.
    OAS 3.1 multi-type schemas (type as array) pass if any listed type matches.
    The following edge cases must appear in the fixture matrix to freeze coercion behavior
    (S4 resolution):
      1e30 under type integer: fract() == 0.0 in f64 so this passes per ADR-021; test
      freezes the behavior.
      -0.0 under type integer: fract() == 0.0 so this passes; test freezes the behavior.
      NaN: not representable as a valid JSON number literal in serde_json; treat as absent
      and skip.
      Infinity: not representable as a valid JSON number literal in serde_json; treat as
      absent and skip.
      [1.0, 2.0] under type integer: each entry has fract() == 0.0, all pass.
    Positive fixture: tests/fixtures/typed-enum/violation.yaml
    (enum: ["cat", "dog"] with type: integer triggers violation).
    Negative fixture: tests/fixtures/typed-enum/clean.yaml
    (enum: [1, 2, 3] with type: integer passes clean).
    Edge-case fixture: tests/fixtures/typed-enum/coercion.yaml
    (enum: [1.0, 2.0] with type integer passes; 1e30 with type integer passes; string
    value with type number fails -- freezes fract() == 0.0 semantics).

  # --- Integration check (runs after all 4 phases) ---

  - >-
    Integration: run "cargo test" to confirm all 17 new rules pass and no regression
    exists on the existing 15 rules (32 total). Verify stripped release binary size
    remains within the ~5 MB budget via
    "cargo build --release && ls -lh target/release/refract".
    Update CHANGELOG.md for v0.3.0: document the 17 new rules, note that external $ref
    nodes are treated as opaque (false negatives only, no false positives), and note that
    oas3-schema and related schema-validation rules are deferred to v0.4.0 (ADR-019,
    ADR-020).
    Update README.md: add callouts for the cross-file $ref gap and schema-validation
    deferral, each linking the v0.4.0 milestone (ADR-020).
    Update PR template: add reviewer checklist line "[ ] Rules that call resolve_ref handle
    None by skipping, not by panicking or emitting a false violation."

architecture_decisions:
  - "detect_oas_version in src/rules/util.rs checks doc[\"swagger\"] first, then doc[\"openapi\"]. Behavior frozen per ADR-021 M2 resolution. Unknown variant emits one-time stderr diagnostic per lint run."
  - "no-$ref-siblings is format-gated inside its own check() implementation, not listed in the shared OAS-version-gated rule table from ADR-021. Rule returns empty violations for V3_1. Scan positions: Schema Objects and Response Objects only."
  - "resolve_ref(doc, pointer, depth) -> Option<&Value> is the canonical deref utility. No thin deref() wrapper is added. Rules call resolve_ref directly and handle None by treating the node as opaque. All plan references use resolve_ref by name (C2 resolution)."
  - "operation-parameters merge: operation-level entries override path-level on matching (name, in). The overridden path-level entry is dropped from the dedup comparison set to prevent false-positive violations on valid overrides (S6 resolution)."
  - "Each of the 4 phases ships as a separate PR. Phases can be reviewed and merged independently. This bounds reviewer load and permits partial milestone delivery (M3 resolution)."
  - "Unknown rule IDs in .spectral.yaml already emit a stderr warning in v0.2.0. No behavior change needed in v0.3.0 (S1 resolution)."

agent_gaps: []

out_of_scope:
  - "oas3-schema, oas2-schema, oas3-valid-schema-example, oas2-valid-schema-example: deferred to v0.4.0. Requires a JSON Schema evaluator; boon is the leading candidate (pure Rust, MIT, drafts 4 through 2020-12). See ADR-019."
  - "Cross-file $ref resolution: deferred to v0.4.0. Internal-only deref from ADR-015 stays the v0.3.0 contract. External $ref nodes are treated as opaque (false negatives only, no false positives). README and CHANGELOG must document the gap. See ADR-020."
  - "Homebrew tap: conditional on traction signal from v0.2.0. Carries from v0.2.0 out_of_scope."
  - "JSON spec line/col: hard deferral from v0.2.0 unless user demand materialises."
  - "Strict typed-enum mode: could be a --strict flag in v0.4.0 if demand emerges."
  - "Deref'd<'a> newtype to enforce deref-before-compare at the type level: re-evaluate if v0.4.0 brings the deref-dependent rule count above 8 (ADR-021 escalation trigger)."
  - "refract-core workspace crate extraction: no external library consumer exists yet. Milestone: v0.4.0 or later."

architect_notes:
  # Team-lead resolutions of critic findings C1, C2, S5, S6 reflected here.
  # ADR-018 and ADR-021 are the primary sources; notes extend them with resolved specifics.

  - title: "no-$ref-siblings is format-gated, not listed in OAS-version-gated rules"
    adr: ADR-018
    change: >
      Rule applies to OAS 2.x and 3.0 only. OAS 3.1 adopts JSON Schema 2020-12
      which permits $ref siblings in Schema Objects, so the rule returns empty violations
      when detect_oas_version returns V3_1. The skip logic lives inside the rule check()
      implementation; it is not in the shared version-gated rule list from ADR-021.
      Scan positions are Schema Objects and Response Objects only.

  - title: "resolve_ref is the canonical deref utility, no wrapper added"
    adr: ADR-021
    change: >
      ADR-021 referred to a deref() function aspirationally. The shipped function is
      resolve_ref(doc, pointer, depth) -> Option<&Value> in src/rules/util.rs. No thin
      deref() wrapper is added in v0.3.0. Rules call resolve_ref directly and handle None
      by treating the node as opaque (skip, do not emit a violation) to avoid false
      positives on external $ref nodes.

  - title: "detect_oas_version Unknown branch emits a one-time diagnostic"
    adr: ADR-021
    change: >
      When detect_oas_version returns Unknown, refract emits one stderr line per lint run:
      "warning: OpenAPI version not recognized, version-gated rules skipped".
      This prevents silent coverage gaps for users on OAS 3.2 pre-releases or
      non-standard version strings such as "3.1.0-rc1".

  - title: "operation-parameters merge drops overridden path-level entries"
    adr: ADR-018
    change: >
      When an operation-level parameter shares (name, in) with a path-level parameter,
      the path-level copy is dropped from the dedup comparison set. Dedup comparison uses:
      path-level parameters minus overridden entries, plus operation-level parameters.
      Without this rule, a valid operation-level override produces a false-positive
      duplicate violation against the path-level copy it was intended to replace.
