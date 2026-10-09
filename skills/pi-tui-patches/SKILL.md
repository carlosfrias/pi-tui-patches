---
name: pi-tui-patches
cluster: pi
description: "Re-apply pi-coding-agent TUI patches (structured thinking block) after a pi/pnpm upgrade"
tags: ["pi","tui","patch","thinking","upgrade","maintenance"]
dependencies: []
composes: []
similar_to: []
called_by: []
authorization_required: true
scope: general
model_hint: claude-sonnet
embedding_hint: "pi tui patch reapply upgrade thinking structured assistant-message pi-coding-agent"
---

# pi-tui-patches

## Purpose
Re-apply local TUI modifications to the installed `@earendil-works/pi-coding-agent` package after it is updated/reinstalled. Currently maintains one patch: **0001-structured-thinking-tui** — renders expanded model thinking as a structured, labeled, indented block instead of letting it flow inline as undifferentiated styled text.

## When to Use
- After any `pi` upgrade or `pnpm` global update that reinstalls `@earendil-works/pi-coding-agent`.
- When verifying whether the TUI patches are still applied (e.g. after a system change).
- When the user reports the thinking output has reverted to "flowing" unstructured text.

## Authorization
**User authorization required before modifying installed package files.** This skill edits compiled `dist` files inside the pnpm global store. Always confirm with the user before applying. Never apply silently.

## Key Capabilities
- Locate the installed `@earendil-works/pi-coding-agent` dist file across pnpm global store layouts.
- Idempotent apply: detects an already-applied patch and no-ops.
- Dry-run status check (`--check`) and revert (`--revert`) from automatic backups.
- Code-block matching (not line numbers) so it survives minor upstream line shifts.

## Usage Patterns

### Check whether the patch is currently applied (no writes)
```bash
node Efforts/PI/pi-coding-agent/patches/reapply.mjs --check
```

### Re-apply after an upgrade
```bash
# 1. Confirm the package was reinstalled (patch likely lost):
node Efforts/PI/pi-coding-agent/patches/reapply.mjs --check
# 2. If status is "not-applied", apply:
node Efforts/PI/pi-coding-agent/patches/reapply.mjs
# 3. Restart pi for the TUI change to take effect.
```

### Revert
```bash
node Efforts/PI/pi-coding-agent/patches/reapply.mjs --revert
```

## Failure Modes & Recovery
- **`ORIGINAL BLOCK NOT FOUND` / status `conflict`**: upstream changed the target code. Open `patches/0001-structured-thinking-tui.patch`, read the intent in the AGENTS.md spec, re-port the transformation against the new code, and update `ORIG_BLOCK`/`NEW_BLOCK` in `reapply.mjs`. Do NOT force-apply a stale block.
- **`Could not locate installed ...`**: the package install layout changed. Update `resolveTarget()` in `reapply.mjs` to find the new path, or run the script from a workspace that has `@earendil-works/pi-coding-agent` as a dependency.
- **Patch lost after every upgrade (expected)**: dist files are compiled output and are overwritten on reinstall. Re-running this skill is the intended maintenance loop.

## Provenance
- Source effort: `Efforts/PI/pi-coding-agent/` (private GitHub repo).
- Agent spec: `AGENTS.md` in that effort.
- The transformation targets `dist/modes/interactive/components/assistant-message.js` in `AssistantMessageComponent.updateContent()`.