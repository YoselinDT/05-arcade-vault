# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

## Project state

Arcade Vault is a planned platform for playing games online and competing for points (see README.md). The repository currently contains only the unmodified `create-next-app` scaffold — no game features, routes, or components have been built yet. Expect to be building most things from scratch rather than following existing patterns.

## Critical: Next.js version mismatch

This project pins `next@16.2.10`, a version ahead of your training data — App Router APIs, conventions, and config may differ from what you expect. Per AGENTS.md, consult `node_modules/next/dist/docs/` (organized as `01-app`, `02-pages`, `03-architecture`, `04-community`) before writing Next.js code, especially for routing, data fetching, and config APIs. Do not assume older-Next.js patterns still apply.

## Commands

```bash
npm run dev      # start dev server
npm run build    # production build
npm run start    # serve production build
npm run lint     # eslint (flat config, eslint.config.mjs)
```

There is no test setup yet (no test runner in package.json).

## Architecture

- App Router only, under `app/` (`app/layout.tsx`, `app/page.tsx`). No `pages/` directory.
- Path alias `@/*` maps to the repo root (`tsconfig.json`).
- Styling is Tailwind CSS v4 via `@tailwindcss/postcss` (no `tailwind.config.*` file — theme tokens are defined inline in `app/globals.css` using `@theme inline`).
- `eslint.config.mjs` uses the flat-config format with `eslint-config-next`'s `core-web-vitals` and `typescript` presets.

## Spec-driven workflow

Per README.md, this project follows spec-driven design using the `Klerith/fernando-skills` skill set (`npx skills@latest add Klerith/fernando-skills`), with work organized through `/spec` and `/spec-impl` commands. No spec files exist in the repo yet.
