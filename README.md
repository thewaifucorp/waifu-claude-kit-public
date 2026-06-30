# waifu-claude-kit-public

Marketplace **público/compartilhado** de skills da WaifuCorp (padrões de código por linguagem). Sem skills de pod/consultoria — essas vivem no marketplace **privado** `waifu-claude-kit`.

## Instalar
```
/plugin marketplace add thewaifucorp/waifu-claude-kit-public
/plugin install waifu-standards@waifu-claude-kit-public
```

## Plugins
- **waifu-standards** — padrões de código compartilhados.
  - skill **code-standards** — padrões Python (complete-not-MVP, config-over-code, structlog/never-print+redação, pydantic, settings, mypy strict, ruff/Google-EN, vertical-slice, FastAPI). Pareia com `ponytail` + `dev-cycle`.
  - skill **ts-standards** — padrões TypeScript/Next/React (tsconfig strict, ESLint flat, Prettier, no-console, env via zod, App-Router/`use client`, estilo congelado por repo). Pareia com `ponytail` + `dev-cycle`.
  - skill **rust-standards** — padrões Rust (thiserror lib/anyhow bin, sem unwrap fora de teste, clippy -D warnings + rustfmt + forbid(unsafe), tracing estruturado/sem println, tokio non-blocking, rustdoc EN, CI). Pareia com `ponytail` + `dev-cycle` + `review-standards`.

## Fronteira
Acesso = **repo**, não pasta. Público aqui = ok pra OSS. Skills de consultoria/pod (Katsui, hosting…) ficam no repo privado.
