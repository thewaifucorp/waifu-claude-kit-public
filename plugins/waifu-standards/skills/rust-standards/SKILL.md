---
name: rust-standards
description: WaifuCorp/Katsui Rust code standards — apply when writing, scaffolding, or reviewing Rust in any WaifuCorp repo (Bastion substrate, Iai Gate plugin, Agent Dojo backend). Judgment, not a checklist. Covers — complete-not-MVP + lean, config-over-code, thiserror(lib)/anyhow(bin)+Result alias, no unwrap in non-test paths, clippy -D warnings + rustfmt + forbid(unsafe), tracing (structured, secret-redacted, never println), tokio non-blocking, English rustdoc, CI(fmt+clippy+test). Pairs with the ponytail, dev-cycle, and review-standards skills.
---

# WaifuCorp — Rust code standards

Same two rules as everywhere: **complete, never an MVP** (all real paths: the error arm, the empty case, cancellation), and **every addition pays rent** (lean, no duplication). **Ponytail only slims, never cuts a scenario.** **Config over code**: a client variant is per-owner YAML (`config`/`serde` at the edge), never a fork.

## Defaults (adopt — they pay rent)
- **Errors: `thiserror` enums at library/domain seams, `anyhow` only at the binary/app boundary.** Define a typed error enum (`#[derive(thiserror::Error)]`, `#[non_exhaustive]` on public ones) per module that has real failure modes; expose a crate `pub type Result<T> = std::result::Result<T, Error>;`. `anyhow` belongs in `main`/command handlers where you just bubble up with context (`.context(...)`). *(Bastion today leans the wrong way — `anyhow` in ~44 files, `thiserror` in 1; new code adds typed errors at the seams.)*
- **No `.unwrap()` / `.expect()` in non-test code.** Use `?` and real errors. `expect("…")` is allowed ONLY on a proven invariant, and the message states *why it cannot fail*. Tests/benches may unwrap freely.
- **Lints, enforced.** Crate root: `#![forbid(unsafe_code)]` (Bastion has zero `unsafe` — keep it) and `#![warn(missing_docs)]` on library surface. A `[lints.clippy]` table at `clippy::pedantic` (curated `allow`s as needed); CI runs `cargo clippy --all-targets --all-features -- -D warnings`. Commit a `rustfmt.toml`; CI runs `cargo fmt --check`.
- **Observability: `tracing`, never `println!`.** Spans around request/agent loops and I/O; **structured fields** (`info!(owner = %owner, n = count, "loaded")`), not string interpolation; JSON subscriber in prod (the `json` feature is already on). **Redact secrets** — no tokens/keys/PII in span fields or events (mirror the Python structlog redaction rule).
- **Async (tokio): never block the runtime.** No sync/blocking I/O inside `async fn` — use `tokio::task::spawn_blocking` or async APIs. Bounded channels (`mpsc::channel(n)`) for backpressure; make long ops cancellation-aware. Don't hold a `std::sync::Mutex` across `.await` (use `tokio::sync` or drop the guard first).
- **`unsafe` is forbidden by default.** If genuinely unavoidable, the smallest possible block with a `// SAFETY:` comment proving the invariants, behind a local `#[allow(unsafe_code)]`.
- **Rustdoc: English** (house rule — code/docs comments in English; prose docs/specs may be pt-BR). `///` on public items; `cargo doc` warning-free. Document errors and panics.

## Structure (YAGNI)
- One crate until a second consumer justifies a workspace split — **don't** pre-carve a workspace. Modules by capability; keep the public API minimal (`pub` only what callers need). Don't add a Cargo feature nobody toggles.

## CI (Bastion has none today — add it)
`.github/workflows/ci.yml` on PR + push to main, running: `cargo fmt --check` → `cargo clippy --all-targets --all-features -- -D warnings` → `cargo test`. (Reference: `katsui-agent-dojo` already gates exactly this.)

## Quick checklist (writing/reviewing)
1. **Complete?** Every `Err` arm, empty case, and cancellation handled (not an MVP)?
2. Library error is a `thiserror` enum (not `anyhow`)? `anyhow` only at the binary edge?
3. Zero `unwrap`/`expect` in non-test paths (or `expect` with a proven-invariant message)?
4. `cargo fmt --check` + `cargo clippy -- -D warnings` clean? `#![forbid(unsafe_code)]` holds?
5. Logs via `tracing` with structured fields; no secret/PII in spans; no `println!`?
6. No blocking call inside `async`; no `std::Mutex` held across `.await`?
7. Could the next client's variant be **config** (YAML), not new code? Then `/ponytail-review` the diff + run `review-standards`.
