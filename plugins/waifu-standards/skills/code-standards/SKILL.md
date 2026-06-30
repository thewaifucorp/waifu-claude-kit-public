---
name: code-standards
description: WaifuCorp Python code standards — apply when writing, scaffolding, or reviewing Python in any WaifuCorp/Katsui repo. Judgment, not a checklist (does it pay rent?). Covers — complete-not-MVP + lean/no-dup, config-over-code, structlog (NEVER print) with secret/PII redaction, pydantic+validation at boundaries, pydantic-settings, mypy --strict, ruff (Google docstrings, English — CI-enforced), vertical-slice-by-capability, FastAPI house shape, stdlib-first/YAGNI. Pairs with the ponytail and dev-cycle skills.
---

# WaifuCorp — Python code standards

**Two rules above all:**
1. **Complete, never an MVP.** Every implementation covers *all real scenarios* (branches, edge cases, error paths). We ship senior-impeccable, not quick-and-dirty.
2. **Every addition pays rent.** Lean and duplication-free — *fewer lines for the same complete coverage, never fewer cases.* Structure that removes duplication, validates untrusted input, or prevents a real bug → adopt. Speculative structure, a dep that duplicates stdlib, boilerplate → cut.

**Ponytail (YAGNI) only slims — it never cuts a scenario.** A "lazy" solution is allowed *only when it already resolves every possible case*; otherwise completeness wins and you write the full thing (still lean). Run `/ponytail-review` on a diff as the over-engineering gate (it checks bloat only — correctness/security is a separate pass).

**North-star — config over code.** The product is a *generic engine*; a client project is solved by **YAML config**, never by writing code (generic image, per-owner config outside it, loaded at runtime by `KATSUI_OWNER`). If a client needs new code, the engine is missing a capability → add it generically, never fork per-client.

## Defaults (adopt — they pay rent)
- **Logging → structlog, configured once. NEVER `print()`.** Not for diagnostics, not for "structured stdout events" — those go through the logger too (`log.info("savings", **event)`, never `print(json.dumps(event))`). **Mandatory redaction processor** masks keys containing secret/token/password/dsn/api_key/cpf/email → `***` (mask the *secret-bearing* key, **not a count metric** — `token`/`access_token` mask, but a plural `tokens` usage/metric key must survive or you silently kill your own telemetry; see iai-gate's carve-out); **never log a secret/PII value.** Console in dev, JSON in prod (by setting). Don't re-author the processor per repo — import the shared one (`katsui-logging`); copy-pasting is how a repo forgets it.
- **Types + validation at boundaries → pydantic v2.** Parse external input (YAML/env/API payloads) into pydantic models; validate at the edge. Domain models too (free `model_dump`). Plain `@dataclass` only for purely-internal value holders that never cross a boundary.
- **Config → pydantic-settings.** One `Settings(BaseSettings)` + cached `get_settings()`. **Never** scatter `os.environ.get` / `os.getenv` across modules.
- **mypy `--strict`** in pyproject + CI, on all hand-written code. Don't `exclude` a real module to dodge it (generated `*_pb2` is the only legit exclusion).
- **ruff** = lint+format+import-sort. `select` includes `E,F,I,UP,B,D`; `[tool.ruff.lint.pydocstyle] convention="google"`; ignore D100/D104/D107; tests ignore `D`.
- **Docstrings: English, Google style** (Args/Returns/Raises) where they add value — no docstring that just restates the signature. *Language is CI-enforced* (`check_docstring_lang.py` in the quality gate) because ruff `D` checks structure, not language. Prose docs/READMEs stay pt-BR; **docstrings are English.**
- **Errors are handled, never swallowed.** No bare `except Exception: pass`. Catch narrowly, log with context, re-raise or return a typed result.
- **Secrets**: `secret://{owner}/{name}` handles resolved at runtime (katsui-secrets); never the raw value in config/log/timeline/telemetry.

```python
# ❌ what we keep finding            →   # ✅ standard
print(f"done {n}")                       log.info("done", n=n)
print(json.dumps(event))                 log.info("savings", **event)
key = os.getenv("ANTHROPIC_API_KEY")     key = get_settings().anthropic_key   # secret:// handle, resolved at use
```

## Structure (vertical-slice, AI-readable)
- Organize by **capability** (e.g. `adapters/`, `squeezer/`, `state_extract/`) with cross-cutting (config, settings, logging, telemetry, shared clients, models) centralized in the package root. Optimized for an AI/dev to read one slice end-to-end.
- **Flat until a capability earns its own dir** (YAGNI on structure — don't reshuffle empty stubs). A cohesive single-purpose component legitimately stays flat forever. (Reference: `mugen` = 7 slices; `kekkai` = flat core + one `faces/` slice; `iai-gate/workers` = fully flat gRPC module — same rule, different mass.)
- `src/` layout, package under `src/<pkg>/`; tests in `tests/` (pytest, `pythonpath=["src"]`).

## Patterns to use (house conventions — codify them)
- **Deliberate shortcut → `# ponytail:` comment naming the ceiling + upgrade path**, e.g. `# ponytail: char/4 token heuristic; swap tiktoken if cost/limits bite`. (Harvested by `ponytail-debt`.)
- **Real-backend stage faked in tests → lazy import + `# pragma: no cover` with a reason.** Keeps the core offline-testable.
- **Dependency tiering** — core deps stay light and offline-runnable; cloud/backends are `[project.optional-dependencies]` extras. This is what lets `pytest`/`mypy` run green in CI with no creds.
- **FastAPI services** follow the house shape (reference: `kekkai-shield/service.py`): `build_app(engine)` app factory · pydantic request/response models with `response_model=` · fully type-hinted routes · `__main__.py` only bootstraps.

## Defer (don't do speculatively — YAGNI)
- Vertical-slice reorg before slices have mass; a new dependency before stdlib genuinely falls short (structlog/pydantic earn it; a 3-line helper usually beats a framework); caching, abstraction layers, plugin systems, config knobs nobody asked for.

## Quick checklist (writing/reviewing)
1. **Complete?** Every real scenario/branch/error path handled (not an MVP)?
2. New dep? Does stdlib do it? Does it pay rent?
3. External input parsed/validated at the boundary (pydantic)? Config via `Settings`, not scattered `os.getenv`?
4. No `print()`; logging via structlog; no secret/PII reachable by a logger/span/export; no `except: pass`?
5. `mypy --strict` clean (no module dodged)? Docstrings English/Google, non-trivial?
6. Could the next client's variant be expressed in **config**, not code?
7. Any structure added before there's code to justify it? → cut. Then `/ponytail-review` the diff.
