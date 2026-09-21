---
name: e2e
description: Build and run code-server's Playwright e2e tests. Use when e2e verification is explicitly requested — the suite needs built artifacts first and is slow, so it is never the default verification (prefer npm run test:unit).
---

# E2E tests (Playwright)

E2E runs against a **build**, not raw source. Only **Chromium** is enabled (Firefox/WebKit projects are disabled in `test/playwright.config.ts` due to known bugs).

## One-time setup

```bash
cd test && npm install && cd ..            # also done by root postinstall
./test/node_modules/.bin/playwright install --with-deps chromium
```

## Build requirements

`test-e2e.sh` sanity-checks for `out/` (TS build) and `lib/vscode/out` (VS Code build) and exits with help text if either is missing:

```bash
git submodule update --init               # if lib/vscode is empty
quilt push -a                             # if patches not yet applied
npm install
npm run build                             # TS -> out/ (fast)
VERSION=0.0.0 npm run build:vscode        # very slow
```

## Run

Unset `CODE_SERVER_TEST_ENTRY` tests the repo root build — the normal local loop:

```bash
npm run test:e2e
```

To test a packaged release instead (what CI does):

```bash
KEEP_MODULES=1 VERSION=0.0.0 npm run release
CODE_SERVER_TEST_ENTRY=./release npm run test:e2e
```

Useful variations (args pass through to Playwright):

```bash
npm run test:e2e -- --grep login
npm run test:e2e -- --workers 1
PWDEBUG=1 npm run test:e2e                 # Playwright inspector
```

## Iteration cheatsheet

- Server-only change (`src/**`): re-run `npm run build` (fast), rerun tests — no VS Code rebuild needed. With `CODE_SERVER_TEST_ENTRY=./release` you must re-run `release` instead, so prefer the unset form while iterating.
- Patch / `lib/vscode` change: full `build:vscode` again.
- Test-extension changes: `test/e2e/extensions/test-extension` is rebuilt automatically by the runner.

## Notes

- Global setup (`test/utils/globalE2eSetup.ts`) pre-authenticates and creates temp `CODE_WORKSPACE_DIR` / `CODE_FOLDER_DIR`; reuse the page-object models in `test/e2e/models/` instead of raw selectors.
- CI runs with 60s timeout, 2 retries, video retained on failure, against `./release`.
- `npm run test:e2e:proxy` runs the suite behind Caddy (`ci/Caddyfile`) with `USE_PROXY=1`.
- New patches to VS Code require an accompanying e2e test (see `/vscode-patch`).
