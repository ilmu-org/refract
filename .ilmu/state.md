---
project: refract
project_type: cli-tool
target_ecosystem: cross-platform (macOS, Linux, Windows)
build_team: rust_build_team

current_milestone: v0.3.0
phase: planning
current_task: v0.3.0 scoping complete -- awaiting build
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

open_questions: []

milestones:
  v0.1.0:
    hypothesis: API teams on non-Node CI adopt single-binary Spectral-compatible linter?
    scope: see .ilmu/plan.md
    status: complete
  v0.2.0:
    hypothesis: >
      CI users on non-Node stacks see enough Spectral rule parity + actionable
      violations (with source locations) to replace Spectral without touching
      existing .spectral.yaml files.
    scope: see .ilmu/plan.md ## v0.2.0
    status: complete
  v0.3.0:
    hypothesis: >
      17 structural/correctness rules close Spectral OAS gap enough that teams
      get equivalent lint coverage on path hygiene, tag validation, param dedup,
      enum integrity -- no Spectral needed.
    scope: see .ilmu/plan.md ## v0.3.0
    status: planning
  v1.0.0:
    hypothesis: public launch -- promote when feature-complete and stable
    scope: TBD
    status: future

last_updated: 2026-04-14
last_agent: v0.3.0 scoping pipeline (architect + critic + sdd)