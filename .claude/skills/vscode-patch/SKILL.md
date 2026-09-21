---
name: vscode-patch
description: Create or modify code-server's quilt patches against the lib/vscode submodule. Use when changing VS Code behavior (marketplace, auth, telemetry, CSP, webviews...) or when a task would edit files under lib/vscode.
---

# Patching VS Code (quilt stack)

All customizations to VS Code live in `patches/*.diff`, applied to the `lib/vscode` submodule with quilt. Never edit files under `lib/vscode` directly — untracked edits there are lost and can't be shared.

## Before you start

- The submodule must be initialized (`git submodule update --init`) and the stack applied (`quilt push -a`).
- Check the current stack state with `quilt top` / `quilt series` — quilt applies patches in `patches/series` order and only the top patch can absorb new edits.
- Reuse an existing patch when extending its concern (e.g. marketplace.diff for Open VSX changes); create a new one only for a new concern.

## Workflow

1. Push the stack to the patch you're extending, or create a new one at the top:
   ```
   quilt push -a                  # apply everything first
   quilt new {name}.diff          # new patch (name matches its concern)
   ```
2. **Register every file before editing it** — quilt will not track edits to unregistered files:
   ```
   quilt add [-P {patch}] lib/vscode/path/to/file.ts
   quilt edit lib/vscode/path/to/file.ts   # optional shorthand for add+edit
   ```
3. Make the changes in `lib/vscode/...`.
4. Capture/refresh the diff:
   ```
   quilt refresh
   ```
5. Add a comment at the top of the `.diff` explaining **why the patch exists and how to reproduce the behavior it fixes or adds**. Look at existing patches in `patches/` for the expected format.
6. Every patch needs an e2e test (see the `/e2e` skill for how to build and run them).

## Rules

- Patches may depend on lower patches, but **every intermediate state of the stack must produce a working code-server** — no broken in-between states.
- The patch stack ordering in `patches/series` matters; don't reorder casually.
- Verify the stack still applies cleanly from scratch: `quilt pop -a && quilt push -a`.

## After pulling changes that touch patches

If a patch no longer applies (e.g. after a VS Code version bump):

1. Apply as many as possible: `quilt push -a`
2. On a conflict: `quilt push -f`, manually restore the rejected hunks in the file, then `quilt refresh`
3. Repeat until the full stack applies.

## After changing a patch

Rebuild to verify: patches affect the VS Code build, so a full `VERSION=0.0.0 npm run build:vscode` (very slow) plus release build is needed for e2e. Unit tests (`npm run test:unit`) don't cover VS Code internals — server-side behavior changes in `src/` are what unit tests catch.
