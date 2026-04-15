milestone: v0.2.0
hypothesis: >
  CI pipeline users on non-Node stacks see enough Spectral rule parity and actionable
  violation output (with source locations) that they replace Spectral in their pipelines
  without modifying existing .spectral.yaml files.

# Priority rationale: items are ranked by user-facing impact for CI pipeline users
# (Go, Python, Java teams) — not library consumers. Binary UX and rule parity dominate.
#
# NOTE — items from the original v0.2.0 deferred list that were ALREADY SHIPPED in v0.1.0:
#   - JSON output (--format json): fully implemented in src/main.rs + src/reporter/mod.rs
#   - Non-zero exit code on violations: implemented (exit 1)
#   - Severity level CLI plumbing: ruleset severity overrides already applied end-to-end
# These three are complete and do not appear in scope below.

scope:
  # --- Prerequisite housekeeping (first commit on v0.2.0 branch, before any feature work) ---
  - "Rename mod.rs files to named module style: src/rules/mod.rs -> src/rules.rs with
    src/rules/ subfolder, same for src/parser/, src/ruleset/, src/reporter/, src/model/.
    Pure rename refactor — no logic changes. Reason: removes the confusing mod.rs pattern
    before the codebase grows further. (From v0.1.0 out_of_scope.)"

  - "Add #![warn(missing_docs)] to lib.rs and fill doc comments on the Rule trait methods
    (id, message, default_severity, check) and all public types. (From v0.1.0 out_of_scope.)"

  - "Collapse reporter::write_text / reporter::write_json into a single
    pub fn report(violations: &[Violation], spec_path: &str, format: Format, color: ColorMode,
    out: &mut dyn Write) -> std::io::Result<()>. Format enum: Text | Json.
    ColorMode enum: Auto | Always | Never. Eliminates the bool use_color anti-pattern.
    Update main.rs call sites. (From v0.1.0 out_of_scope.)"

  # --- Tech debt (addresses known gaps before shipping new features) ---
  - "Improve parser error messages: include file path and, for YAML errors, the line/col
    from serde_yaml::Error (which exposes location()). Current LintError::Yaml wraps the
    raw serde_yaml error with no context — add the spec file path to the error message.
    (From v0.1.0 known tech debt.)"

  - "Warn on unknown rule IDs in .spectral.yaml: after loading RulesetConfig, compare
    severity_overrides keys against the registry rule IDs. For any key not matching a
    known rule, emit a warning to stderr: 'unknown rule id in ruleset: {id}'.
    Do not error — Spectral itself warns but continues. (From v0.1.0 known tech debt.)"

  # --- Line/col violation locations (high CI impact: actionable output) ---
  - "Add line and column numbers to violations. Approach: two-pass parsing.
    Pass 1: serde_yaml / serde_json parse to Value (existing).
    Pass 2: scan the source with serde_yaml's event-based API (serde_yaml::Deserializer
    + serde::de::Deserializer) or yaml-rust2 (maintained yaml-rust fork with Marker support)
    to build a PositionIndex: HashMap<String, Span> mapping JSON Pointer paths to
    (line, col) u32 pairs. Span is cheaply cloneable (two u32s).
    Add optional line: Option<u32> and col: Option<u32> to Violation struct.
    Rules populate path (JSON Pointer) as before; lib::lint resolves Span from PositionIndex
    after rule.check() returns and fills line/col.
    Update text reporter format to: {file}:{line}:{col}  {severity}  {rule_id}  {message}
    (matches ADR-009 intent). Update JSON reporter to include line and col fields.
    For JSON specs: serde_json also lacks span support natively — use a two-pass scan with
    a custom serde_json::de::StrRead visitor or post-parse byte-offset lookup. Defer JSON
    line/col if the approach is unduly complex; ship YAML line/col first.
    (Deferred from v0.1.0 scope per plan.md and ADR-002.)"

  # --- Rule parity (high CI impact: closer Spectral compatibility) ---
  - "Implement path-params rule: every path parameter token in a path template
    (e.g. {petId} in /pets/{petId}) must have a matching parameter object with
    in: path in the operation or path-level parameters array. Works for OAS 2.x and 3.x.
    Rule ID: path-params. Default severity: error.
    Implementation: for each path string, extract tokens between { }; for each operation
    on that path, collect parameters from path-level and operation-level parameters arrays;
    verify every token has a matching name with in=path. Emit a violation per missing param.
    This is the highest-priority deferred rule — path parameter mismatches are CI-blocking
    errors in practice. (From v0.1.0 out_of_scope.)"

  - "Implement the 6 deferred cosmetic/quality rules from ADR-006:
      1. info-license — info object must have a license field. Severity: warn.
      2. license-url — license object should have a url field. Severity: warn.
      3. contact-properties — contact object should have name, url, and email. Severity: warn.
      4. openapi-tags-alphabetical — top-level tags array should be sorted alphabetically
         by name. Severity: warn.
      5. no-eval-in-markdown — no eval() call in any description or summary field.
         Severity: error. Traverses all string fields named description or summary.
      6. no-script-tags-in-markdown — no <script> tag in any description or summary field.
         Severity: error. Same traversal as no-eval-in-markdown.
    Brings built-in rule count from 8 to 15, matching ADR-006 original target.
    (From v0.1.0 out_of_scope.)"

  # --- Recursive directory scanning (high CI impact: lint whole repo at once) ---
  - "Add recursive directory scanning. When <spec> argument is a directory, walk it
    recursively and lint every .yaml, .yml, and .json file found. Skip files that do not
    parse as valid OpenAPI (emit a warning to stderr, continue).
    Output all violations with per-file prefixes. Summary line at end: N files linted,
    M violations found (E errors, W warnings).
    Exit code: 0 if all files clean, 1 if any violations, 2 if any files failed to parse.
    No async runtime needed — synchronous walkdir traversal is sufficient for v0.2.0.
    Add walkdir = '2' dependency (MIT/Apache, widely used, no transitive C deps).
    (Proposed new scope for v0.2.0.)"

  # --- SARIF output (medium CI impact: GitHub Code Scanning integration) ---
  - "Add SARIF 2.1.0 output format (--format sarif). SARIF is the GitHub Code Scanning
    ingestion format — enables native GitHub PR annotations without a problem matcher.
    Schema: emit a minimal SARIF run with tool.driver.rules (one per rule ID, with
    shortDescription and helpUri), and results array (one per violation) with
    ruleId, level (error/warning/note), message.text, and locations[0].physicalLocation
    (artifactLocation.uri + region.startLine + startColumn, populated from Span if available).
    No external SARIF crate needed — hand-roll the JSON structure with serde_json::json!.
    Add Format::Sarif variant to the reporter Format enum.
    (From v0.1.0 out_of_scope.)"

  # --- Distribution (medium CI impact: install friction reduction) ---
  - "Add brew tap distribution. Create a homebrew tap repo (ilmu-org/homebrew-tap or
    ilmu-org/homebrew-refract). Add a Formula file that downloads the macOS
    universal binary from GitHub Releases and verifies SHA256. Document install in README:
    brew tap ilmu-org/tap && brew install refract. No binary changes required.
    (From v0.1.0 out_of_scope. Only if traction signal from v0.1.0 release confirms demand.)"

  # --- Tests ---
  - "Extend integration tests in tests/integration.rs and tests/cli.rs to cover:
    - Text output format includes line and col numbers (once Span is implemented)
    - SARIF output structure (violations array, ruleId, level, line/col in physicalLocation)
    - path-params rule: positive and negative fixtures for OAS 2.x and 3.x
    - All 6 new cosmetic rules: positive and negative fixtures
    - Directory scanning: fixture directory with multiple specs, assert per-file violations
    - Unknown rule ID in .spectral.yaml emits a warning to stderr
    - Improved parser error messages include the file path"

architecture_decisions:
  - "PositionIndex built from two-pass YAML scan using yaml-rust2 (maintained fork with
    Marker span support). Avoids unsafe-libyaml C bindings. PositionIndex is
    HashMap<String, Span> where key is JSON Pointer. Populated in parser::parse() and
    threaded through lib::lint() to rule violations. (Extends ADR-002.)"
  - "walkdir = '2' for recursive directory scanning. Synchronous traversal — no async
    runtime needed for v0.2.0. If parallel scanning becomes necessary (v0.3.0+),
    rayon can be layered on top. (ADR-010 unchanged.)"
  - "SARIF hand-rolled with serde_json::json! — no external SARIF crate. Schema is stable
    (SARIF 2.1.0 is an OASIS standard). Avoids a large optional dependency. (Extends ADR-009.)"
  - "Reporter API collapsed to report(Format, ColorMode) before feature work begins.
    Format enum gains Sarif variant. ColorMode replaces bool use_color. (Extends ADR-009.)"
  - "no-eval-in-markdown and no-script-tags-in-markdown require traversing all string-valued
    fields named 'description' or 'summary' across the entire document tree, not just at
    known paths. Implement a recursive Value walker helper in src/rules/util.rs that yields
    (path: String, value: &str) for all string fields matching a predicate."

agent_gaps: []

out_of_scope:
  - "Custom rule definitions (given/then/message/function). Requires a JSON Path engine
    and JS function execution — antithetical to the no-runtime value prop. Milestone: v0.3.0
    or never (separate product decision). (Carried from v0.1.0.)"
  - "Remote ruleset URL resolution. Network dependency breaks offline CI and adds latency.
    Would require reqwest + TLS + a caching layer. Milestone: v0.3.0. (Carried from v0.1.0.)"
  - "spectral:asyncapi ruleset. Different domain; v0.2.0 hypothesis is still OAS-specific.
    Milestone: v0.3.0. (Carried from v0.1.0.)"
  - "Multi-file $ref resolution. Requires a resolver that follows relative file paths and
    cycles gracefully. High complexity; violates single-file scope. Milestone: v0.3.0."
  - "Watch mode and LSP server mode. Editor integration is post-validation scope.
    Milestone: v0.3.0. (Carried from v0.1.0.)"
  - "refract-core workspace crate extraction. No external library consumer exists yet.
    Natural shape will emerge when one does. Milestone: v0.3.0 or later."
  - "cargo install distribution. GitHub Releases binary download is sufficient for v0.2.0
    CI use case. cargo install requires publishing to crates.io; premature without a
    stable public API. Milestone: v0.3.0."
  - "Parallel multi-file linting with rayon. Synchronous walkdir is sufficient for v0.2.0.
    Milestone: v0.3.0 if benchmarks show a bottleneck."
  - "JSON spec line/col (if serde_json two-pass scan proves unduly complex). Ship YAML
    line/col in v0.2.0; JSON line/col deferred to v0.3.0 if needed."
  - "Inline $col offset within a string value (e.g. column of eval() within a description).
    JSON Pointer path + start-of-node line/col is sufficient for v0.2.0. (From ADR-002.)"

architect_notes:
  # Appended by rust-architect 2026-04-14. Refines, does not rewrite, the SDD scope above.
  # Each note corresponds to an ADR in .ilmu/decisions.md.

  - title: "Reporter signature reshaped for SARIF multi-file correctness"
    adr: ADR-013
    change: >
      The proposed per-file signature
        report(violations, spec_path, format, color, out)
      breaks for SARIF output, which MUST be a single document across all scanned
      files (GitHub Code Scanning requires one .sarif upload with a single run).
      Revised signature:
        report(files: &[(PathBuf, Vec<Violation>)], format: Format, color: ColorMode,
               out: &mut dyn Write) -> std::io::Result<()>
      Single-file invocations wrap in a one-element slice. main.rs makes exactly one
      report() call regardless of single-file vs directory-scan invocation.
    breaking: >
      JSON output shape changes from v0.1.0's
        { "violations": [...], "summary": {...} }
      to v0.2.0's
        { "files": [{"file": "...", "violations": [...]}], "summary": {...} }
      This is an acceptable pre-1.0 break; document in the v0.2.0 release notes.

  - title: "path-params requires document-internal $ref resolution utility"
    adr: ADR-015
    change: >
      Real-world OAS 3 specs overwhelmingly use
        parameters: [{ $ref: '#/components/parameters/PetId' }]
      Without resolving internal refs, path-params will false-positive on nearly
      every well-structured spec. Add src/rules/util.rs with resolve_internal_ref()
      and deref() helpers as a PREREQUISITE scope item, landed before path-params.
      Utility is pub(crate), available to future rules. External ($ref to another
      file) stays permissive: skip, do not error.

  - title: "JSON spec line/col is a hard deferral, not a stretch goal"
    adr: ADR-011
    change: >
      The plan scope line 'Defer JSON line/col if the approach is unduly complex'
      is hardened: v0.2.0 ships WITHOUT JSON line/col. JSON specs get
      PositionIndex::empty(); text output drops the :line:col suffix, SARIF emits
      region.startLine = 1. This removes optional branching from the scope and lets
      the v0.2.0 implementation focus on the YAML two-pass scanner (the load-bearing
      new code). Revisit for v0.3.0 if user demand materialises.

  - title: "Rule trait signature is unchanged; rules never see PositionIndex"
    adr: ADR-011
    change: >
      Rules continue to emit Violation with path: String only. lib::lint() resolves
      each violation's path against the PositionIndex AFTER rule.check() returns,
      filling line/col in place. This keeps rule authorship ergonomic and keeps
      PositionIndex an implementation detail of the parser/lint-orchestration layer.
      Add a Violation::new(rule_id, message, severity, path) constructor so existing
      rule code does not need to know about the new fields.

  - title: "lint_dir is additive; lint() stays atomic"
    adr: ADR-014
    change: >
      Proposed signatures:
        pub fn lint(spec_path, ruleset_path)
            -> Result<Vec<Violation>, LintError>;              // unchanged
        pub fn lint_dir(dir_path, ruleset_path)
            -> Result<Vec<(PathBuf, Result<Vec<Violation>, LintError>)>, LintError>;
      Per-file parse errors become inner Err entries; lint_dir does not abort.
      Exit code policy stays in main.rs: 0 clean, 1 violations, 2 any file errored
      or directory-level failure. Symlinks are followed (walkdir default);
      .git/node_modules are NOT auto-excluded in v0.2.0.

  - title: "Violation gains flat line/col fields, no Location wrapper"
    adr: ADR-012
    change: >
      Violation adds line: Option<u32>, col: Option<u32> directly — no Location
      struct. Rationale: the fields share no invariants with each other or with
      path, and there is no second 'thing with a location' in the domain. SARIF
      maps directly to the flat fields. A wrapper can be introduced later
      mechanically if multi-file $ref resolution (v0.3.0+) needs it.
      Add #[serde(skip_serializing_if = "Option::is_none")] to keep JSON output
      clean on JSON-spec inputs.

  - title: "Module rename pattern confirmed (src/rules.rs + src/rules/ subdir)"
    adr: ADR-016
    change: >
      Rename applies to: parser, model, rules, ruleset, reporter. error.rs is
      already a named module. Rust 2018+ supports src/rules.rs + src/rules/
      directory coexistence with no change to submodule declarations. Single
      mechanical first commit on v0.2.0 branch; cargo build passes identically
      before and after. Do not bundle with feature work.

  - title: "Dependency additions summary"
    adr: ADR-011, ADR-014
    change: >
      New [dependencies] entries for v0.2.0:
        walkdir = "2"        # recursive traversal, pure Rust, MIT/Apache
        yaml-rust2 = "0.10"  # two-pass YAML scan with span markers, pure Rust,
                             # MIT/Apache (verify latest stable before pin)
      Both are pure-Rust with no transitive C FFI. musl build story unchanged.
      Binary size delta expected: +300-500 KB stripped. Within budget (~5 MB target).

  - title: "Implementation sequencing constraint"
    adr: ADR-011, ADR-013, ADR-014, ADR-015
    change: >
      Recommended build order to avoid rework:
        1. Housekeeping: mod.rs rename, #![warn(missing_docs)], reporter API
           collapse to batch signature (ADR-013 scaffolding without Sarif impl).
        2. Tech debt: parser error context, unknown-rule-id warning.
        3. PositionIndex + Violation line/col (ADR-011, ADR-012). Update text
           reporter to use line/col. Ship this first — SARIF depends on it.
        4. $ref utility (ADR-015). Unit-tested in isolation.
        5. Rule parity: path-params first (exercises $ref utility), then the 6
           cosmetic rules (exercise the recursive Value walker in util.rs).
        6. Directory scanning + lint_dir (ADR-014). Exit code wiring in main.rs.
        7. SARIF output (ADR-013 completion). Uses line/col from step 3.
        8. Homebrew tap (independent; conditional on v0.1.0 traction signal).
      Tests expand alongside each step, per the existing test scope item.
