---
name: code-standards
description: WaifuCorp Python code standards — apply when writing, scaffolding, or reviewing Python in any WaifuCorp repo. Encodes judgment (does it pay rent?) not a checklist: pydantic+validation at boundaries, pydantic-settings, structlog with secret/PII redaction, mypy --strict, ruff (Google docstrings, English), vertical-slice-by-capability, stdlib-first/YAGNI. Use with the ponytail skill.
---

# WaifuCorp — Python code standards

**The one rule: every addition pays rent.** Structure that removes duplication, validates untrusted input, or prevents a real bug → adopt. Speculative structure, a dep that duplicates stdlib, boilerplate that restates the obvious → cut. Pair with **ponytail** (YAGNI). When unsure: stdlib first, one line before fifty, question if the thing needs to exist.

## Defaults (adopt — they pay rent)
- **Types + validation at boundaries → pydantic v2.** Parse external input (YAML/env/API payloads) into pydantic models; validate at the edge. Once pydantic is in, use it for domain models too (free `model_dump` for serializing payloads). Plain `@dataclass` is fine for purely-internal value holders that never cross a boundary.
- **Config → pydantic-settings.** One `Settings(BaseSettings)` + cached `get_settings()`; never scatter `os.environ.get`.
- **Logging → structlog over stdlib**, configured once. **Mandatory redaction processor**: mask keys containing secret/token/password/dsn/api_key/cpf/email → `***`. **Never log secret/PII values.** Console in dev, JSON in prod (by setting).
- **mypy `--strict`** in pyproject + CI. (ruff doesn't type-check.)
- **ruff** = lint+format+import-sort. `select` includes `E,F,I,UP,B,D`; `[tool.ruff.lint.pydocstyle] convention = "google"`; ignore D100/D104/D107; tests ignore `D`.
- **Docstrings: English, Google style** (Args/Returns/Raises) where they add value. No docstring that just restates the signature.
- **Secrets**: `secret://{owner}/{name}` handles resolved at runtime (katsui-secrets); never the raw value in config/log/timeline/telemetry.

## Defer (don't do speculatively — YAGNI)
- **Vertical slice / folder reorg** only when slices have *mass*. Organize by capability (e.g. `information/`, `state/`) with cross-cutting in `core/` — but **don't reshuffle empty stubs**; let the layout emerge as code fills in.
- **A new dependency** until stdlib genuinely falls short. structlog and pydantic earn it (redaction, boundary validation); a 3-line helper usually beats a framework.
- Caching, abstraction layers, plugin systems, config knobs **nobody asked for yet**.

## Structure
- Cross-cutting (config, settings, logging, telemetry, shared clients, models) → centralize once. Duplication is the enemy; centralizing is *pro*-lazy, not ceremony.
- `src/` layout, package under `src/<pkg>/`; tests in `tests/` (pytest, `pythonpath = ["src"]`).

## How ponytail relates (not a conflict)
Ponytail fights **unpaid complexity** (speculative abstraction, deps duplicating stdlib, boilerplate) — *not* DRY, validation, or a logging standard (those are less code / fewer bugs = its goal). So these standards and ponytail agree on everything except: ponytail vetoes premature structure and unearned deps. Run `/ponytail-review` on a diff as the over-engineering gate.

## Quick checklist (writing/reviewing)
1. New dep? Does stdlib do it? Does it pay rent?
2. External input parsed/validated at the boundary (pydantic)?
3. No secret/PII reachable by a logger/span/export?
4. Types pass `mypy --strict`? Docstrings English/Google, non-trivial?
5. Any structure added before there's code to justify it? → cut.
