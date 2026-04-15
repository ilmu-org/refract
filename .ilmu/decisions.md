# Architecture Decision Records

Written by rust-architect only.
All build team agents must read this index before starting any task.
Fetch specific ADR files from `.ilmu/decisions/` for details relevant to your work.
Contradicting a decision requires filing an escalation issue on _ops before proceeding.

## Index

| ADR | Title | Status | Summary |
|-----|-------|--------|---------|
| [ADR-001](decisions/ADR-001.md) | Single-Crate Structure for v0.1.0 | Accepted | Use a single crate with internal module boundaries. `src/lib.rs` owns all bus... |
| [ADR-002](decisions/ADR-002.md) | YAML and JSON Parsing with serde + serde_y... | Accepted | - `serde_yaml = "0.9"` for YAML deserialization to `serde_yaml::Value`. |
| [ADR-003](decisions/ADR-003.md) | OpenAPI Validation via Hand-Rolled Rule En... | Accepted | Represent parsed OpenAPI documents as `serde_json::Value` (normalised from YA... |
| [ADR-004](decisions/ADR-004.md) | CLI with clap v4 | Accepted | Use `clap = "4"` with the `derive` feature. |
| [ADR-005](decisions/ADR-005.md) | Error Handling with thiserror in lib, anyh... | Accepted | - `src/lib.rs` and all modules under `src/`: use `thiserror` to define typed ... |
| [ADR-006](decisions/ADR-006.md) | Built-in OAS Ruleset — 15 High-Value Spect... | Accepted | Implement these 15 rules as `Rule` structs in `src/rules/`. Each rule struct ... |
| [ADR-007](decisions/ADR-007.md) | Static Binary Compilation Strategy | Accepted | - Release targets: `x86_64-unknown-linux-musl`, `aarch64-unknown-linux-musl`,... |
| [ADR-008](decisions/ADR-008.md) | Spectral Ruleset YAML Compatibility Scope ... | Accepted | v0.1.0 supports: |
| [ADR-009](decisions/ADR-009.md) | Output Formats — Text (Default) and JSON | Accepted | Two output formats selected by `--format text|json`: |
| [ADR-010](decisions/ADR-010.md) | No Async Runtime for v0.1.0 | Accepted | No async runtime. All I/O is synchronous (`std::fs::read_to_string`). The rul... |
| [ADR-011](decisions/ADR-011.md) | Position Indexing via yaml-rust2 Two-Pass ... | Accepted | For YAML spec files, v0.2.0 uses a two-pass approach: |
| [ADR-012](decisions/ADR-012.md) | Flat `line`/`col` Fields on Violation (Not... | Accepted | Adopt Option A. Extend `Violation`: |
| [ADR-013](decisions/ADR-013.md) | Reporter API — Batch Signature for Multi-F... | Accepted | The v0.2.0 reporter API is: |
| [ADR-014](decisions/ADR-014.md) | Directory Scanning with walkdir; lint_dir ... | Accepted | **API shape**: |
| [ADR-015](decisions/ADR-015.md) | Document-Internal `$ref` Resolution Utilit... | Accepted | Add a shared utility in `src/rules/util.rs`: |
| [ADR-016](decisions/ADR-016.md) | mod.rs → Named Module Rename Confirms Rust... | Accepted | Rename pattern (applied to `parser`, `model`, `rules`, `ruleset`, `reporter`): |
| [ADR-017](decisions/ADR-017.md) | Project Rename — openapi-linter → refract | Accepted | Rename the project from `openapi-linter` to `refract` at the pre-v0.3.0 stage... |
| [ADR-018](decisions/ADR-018.md) | v0.3.0 Rule Set, 17 Structural and Correct... | Accepted | v0.3.0 ships exactly 17 class-1 structural/correctness rules; no schema-eval ... |
| [ADR-019](decisions/ADR-019.md) | JSON Schema Validation Rules Deferred to v... | Accepted | Defer all four schema-evaluation rules to v0.4.0. Defer the boon (or alternat... |
| [ADR-020](decisions/ADR-020.md) | Cross-File $ref Resolution Deferred to v0.4.0 | Accepted | Defer cross-file `$ref` resolution to v0.4.0. The internal-only `deref` utili... |
| [ADR-021](decisions/ADR-021.md) | OAS-Version Gating Helper and Deref-Before... | Accepted | **Version gating helper.** Add a small helper to `src/rules/util.rs`: |
