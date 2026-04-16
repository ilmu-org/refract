---
project: refract
project_type: cli-tool
target_ecosystem: cross-platform (macOS, Linux, Windows)
build_team: rust_build_team

current_milestone: v0.5.0
phase: planning
current_task: v0.5.0 scoping done, awaiting build
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

open_questions: []

milestones:
  v0.1.0:
    hypothesis: Non-Node CI teams adopt single-binary Spectral-compatible linter?
    scope: see .ilmu/plan.md
    status: complete
  v0.2.0:
    hypothesis: >
      Non-Node CI users: Spectral rule parity + actionable violations with
      source locations = replace Spectral, keep existing .spectral.yaml files.
    scope: see .ilmu/plan.md ## v0.2.0
    status: complete
  v0.3.0:
    hypothesis: >
      17 structural/correctness rules close Spectral OAS gap. Teams get
      equivalent coverage: path hygiene, tag validation, param dedup,
      enum integrity. No Spectral needed.
    scope: see .ilmu/archive/plan-v0.3.0.md
    status: complete
  v0.4.0:
    hypothesis: >
      Cross-file $ref resolution + JSON Schema validation (boon) + 4 new rules
      bring refract to full structural correctness parity with Spectral OAS ruleset.
    scope: see .ilmu/archive/plan-v0.4.0.md
    status: complete
  v0.5.0:
    hypothesis: >
      6 new Spectral OAS parity rules (4 OAS 2.x structural + 2 media-level example
      validation) close remaining Spectral OAS gap except graph-analysis rules.
      Teams replacing Spectral get near-complete coverage; only oas3-unused-component
      remains, deferred to v0.6.0.
    scope: see .ilmu/plan.md ## v0.5.0
    status: scoping complete, build pending
  v1.0.0:
    hypothesis: public launch -- promote when feature-complete + stable
    scope: TBD
    status: future

last_updated: 2026-04-16
last_agent: claude-sonnet-4-6 (v0.5.0 scoping)