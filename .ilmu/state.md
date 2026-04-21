---
project: refract
project_type: cli-tool
target_ecosystem: cross-platform (macOS, Linux, Windows)
build_team: rust_build_team

current_milestone: v0.5.0
phase: build
current_task: v0.5.0 PR open, awaiting review
completed_tasks:
  - planning/rust-architect
  - planning/rust-critic
  - planning/sdd
  - plan-approved
  - v0.2.0-scoping
  - v0.2.0-build
  - v0.2.0-release
  - rename/refract-pr-opened
  - v0.3.0-scoping
  - v0.3.0-build
  - v0.3.0-release
  - v0.4.0-scoping
  - v0.4.0-build
  - v0.4.0-release
  - v0.5.0-scoping
  - v0.5.0-build

open_questions: []

milestones:
  v0.1.0:
    hypothesis: Non-Node CI teams adopt single-binary Spectral-compatible linter?
    scope: see .ilmu/plan.md
    status: complete
  v0.2.0:
    hypothesis: >
      Non-Node CI: Spectral rule parity + actionable violations with source
      locations = replace Spectral, keep .spectral.yaml files.
    scope: see .ilmu/plan.md ## v0.2.0
    status: complete
  v0.3.0:
    hypothesis: >
      17 structural/correctness rules close Spectral OAS gap. Coverage:
      path hygiene, tag validation, param dedup, enum integrity. No Spectral needed.
    scope: see .ilmu/archive/plan-v0.3.0.md
    status: complete
  v0.4.0:
    hypothesis: >
      Cross-file $ref resolution + JSON Schema validation (boon) + 4 new rules
      = full structural correctness parity with Spectral OAS.
    scope: see .ilmu/archive/plan-v0.4.0.md
    status: complete
  v0.5.0:
    hypothesis: >
      6 new Spectral OAS parity rules (4 OAS 2.x structural + 2 media-level
      example validation) close remaining gap except graph-analysis rules.
      Near-complete Spectral coverage; only oas3-unused-component remains,
      deferred to v0.6.0.
    scope: see .ilmu/plan.md
    status: PR open (build/v0.5.0 -> main, PR #19), awaiting review
    pr: https://github.com/ilmu-org/refract/pull/19
    rule_count: 42
    binary_size: 2.1 MB
  v1.0.0:
    hypothesis: >
      Drop-in Spectral OAS replacement: .spectral.yaml works unchanged
      (custom declarative rules + built-in functions), behaviour matches
      parity corpus, README reflects compatibility boundary.
    scope: see .ilmu/v1-readiness.md
    status: future

last_updated: 2026-04-21
last_agent: claude-sonnet-4-6 (v0.5.0 build)