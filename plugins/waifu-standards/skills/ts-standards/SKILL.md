---
name: ts-standards
description: WaifuCorp TypeScript / Next.js / React code standards — apply when writing, scaffolding, or reviewing TS/TSX in any WaifuCorp/Katsui frontend (katsui-control-tower, the site, 4gentes). Judgment, not a checklist. Covers — complete-not-MVP + lean, config-over-code, tsconfig strict floor, one ESLint flat baseline, Prettier, no-console (logger helper), zod-validated env, Next App-Router/use-client discipline, per-repo frozen styling, stdlib/platform-first/YAGNI. Pairs with the ponytail and dev-cycle skills.
---

# WaifuCorp — TypeScript / Next.js standards

Same two rules as everywhere: **complete, never an MVP** (all real states: loading/empty/error/success), and **every addition pays rent** (lean, no duplication). **Ponytail only slims, never cuts a scenario.** **Config over code**: a client variant is per-owner config, never a fork — for Control Tower, **per-owner views = config, not a forked component.**

## Defaults (adopt — they pay rent)
- **`strict: true` is the floor** in every `tsconfig` (+ `noUncheckedIndexedAccess`). No `any` to dodge a type — model it. (Reference: katsui-control-tower already strict; katsui + 4gentes must raise.)
- **One ESLint baseline, flat config** (`typescript-eslint` strict + `react-hooks` + `react-refresh`), with `eslint-config-next` layered **only** on Next repos. (katsui's flat config is the model to copy.)
- **One formatter: Prettier**, committed (`.prettierrc` + `format`/`format:check` scripts). Zero-migration pick — just adopt it everywhere.
- **No `console.*` in `src/`** (dev/test exempt). Route through one tiny typed `log` helper per repo (`src/lib/log.ts`) so prod logging is consistent — state the rule; don't ship a logger here.
- **Validate env once at startup with zod**, exported as a typed object; every repo carries a committed `.env.example`. Never read `process.env.X` scattered across components.
- **Components**: function components + hooks only; hooks obey rules-of-hooks; data fetching via TanStack Query (the house default), not ad-hoc `useEffect` fetches. Keep components presentational; push logic into hooks/lib.
- **Scripts every repo exposes**: `lint`, `format:check`, `typecheck` (`tsc --noEmit`), `test`, `build` — these are exactly what the CI quality-gate runs.

## Next.js discipline (App Router repos — Control Tower)
- **App Router only.** Server Components by default; `'use client'` only where interactivity needs it, and hooks live **only** in client files.
- **API access via BFF route handlers / server actions**, not client-side calls holding secrets. Secrets stay server-side (`secret://` resolved server-side), never shipped to the bundle.
- **Styling is frozen per-repo, no mixing**: Control Tower = own CSS matched **1:1 to the design artifacts** (no UI lib); 4gentes = MUI/Emotion; the site = vanilla CSS. Don't introduce a second styling system into a repo.

## Defer (YAGNI)
- A component library, state-management lib, or design-system package before there's duplication to justify it. A new dep before the platform (`<input type=...>`, CSS, `fetch`, `URL`) falls short. Forcing the three frontends onto one framework (they're Next/Vite/CRA — fine as-is). Uniform test framework across repos — document per-repo (`node --test` / Jest / Vitest), align later.

## Quick checklist (writing/reviewing)
1. **Complete?** loading / empty / error / success states all handled?
2. `tsconfig` strict (+ `noUncheckedIndexedAccess`)? No `any` escape hatch?
3. No `console.*` in `src/`; logging via the `log` helper?
4. Env read through the zod-validated typed object; `.env.example` updated?
5. (Next) Server Components by default; `'use client'` minimal; no secret in the client bundle?
6. Could the next owner's variant be **config**, not a fork?
7. `typecheck` + `lint` + `format:check` clean? Then `/ponytail-review` the diff.
