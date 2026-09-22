# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

code-server runs VS Code on a remote server, accessed in the browser. TypeScript Node server lives in `src/` (`node/` server, `browser/` static login/error assets, `common/` shared). VS Code itself is a git submodule at `lib/vscode`, customized by a quilt-managed patch stack in `patches/` (`patches/series` is the stack order).

This is a downstream fork: PRs target `MotusLabs/code-server` (not coder/code-server) and are squashed into `main`. Commit messages are plain imperative summaries. Claude's commits stay unsigned — the user signs before pushing.

## Setup (order matters)

1. `git submodule update --init` — `lib/vscode` must exist before `npm install` (postinstall installs its deps; `SKIP_SUBMODULE_DEPS=1` skips that slow step)
2. `quilt push -a` — applies the patch stack
3. `npm install`
4. `npm run watch` — dev server at http://localhost:8080

"Forbidden access" in the browser means patches didn't apply: `quilt pop -a && quilt push -a`.

npm only — `preinstall` throws under yarn. Node 24 (`.node-version`; the flake.nix `nodejs_22` pin is stale).

## Commands

- `npm run build` — compiles TS to `out/`
- `npm run test:unit` — jest; the default verification for changes. Single file: `npm run test:unit -- test/unit/node/cli.test.ts` (add `--coverage=false` to skip the 60% coverage gate). `npm run test` is a stub that prints a pointer to these two and exits 1.
- e2e requires a full release build first and is slow — run only when explicitly asked (`/e2e` skill)
- `npm run lint:ts` — eslint `--max-warnings=0` over tracked ts/js, excluding `lib/vscode`
- `npm run lint:scripts` — shellcheck over tracked shell scripts
- `npm run prettier` — format; CI enforces `npx prettier --check .`
- `npm run fmt` — prettier + doctoc (regenerates docs TOCs; CI fails if they drift)
- `npm run build:vscode` / `npm run release` — require the `VERSION` env var (e.g. `VERSION=0.0.0 npm run build:vscode`); VS Code builds take a very long time

## Style

- Prettier (enforced): printWidth 120, **no semicolons**, trailing commas everywhere, double quotes.
- ESLint: `eqeqeq` required; `import/order` alphabetized. `no-explicit-any`, `no-non-null-assertion` etc. are off.
- Never edit files under `lib/vscode` directly — changes there go through quilt patches (`/vscode-patch` skill).

## Patches (patches/*.diff)

`quilt new {name}.diff` → `quilt add {file}` (mandatory before editing) → edit → `quilt refresh`. Each patch needs a comment explaining the reason and reproduction, plus an e2e test. Patches may depend on each other, but every intermediate state of the stack must yield a working code-server. User-facing changes should update CHANGELOG.md under `## Unreleased` (Keep-a-Changelog format).

i18n/display languages only work in full builds, not `npm run watch` dev mode.
