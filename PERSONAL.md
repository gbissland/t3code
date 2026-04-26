# Personal Fork Notes

This file lives only on the `personal/nova-editor` branch in `gbissland/t3code`. It does not exist upstream.

## Purpose

Personal fork of `pingdotgg/t3code` that adds **Panic Nova** to the "Open in" editor picker. The change is too niche to upstream, so it lives on a long-lived branch that gets rebased on each upstream release.

## Remotes

- `origin` → `git@github.com:gbissland/t3code.git` (this fork — push/pull personal branch here)
- `upstream` → `https://github.com/pingdotgg/t3code.git` (Theo's repo — fetch new releases from here)

## The Patch

Single commit: `feat(editor): add Panic Nova to Open In picker`. Four files, all additive:

| File | Change |
|------|--------|
| `packages/contracts/src/editor.ts` | Added `{ id: "nova", label: "Nova", commands: ["nova"], launchStyle: "direct-path" }` to `EDITORS` array |
| `apps/web/src/components/Icons.tsx` | Added `NovaIcon` (4-point star SVG) after `AntigravityIcon` |
| `apps/web/src/components/chat/OpenInPicker.tsx` | Imported `NovaIcon`, added Nova entry to `baseOptions` between IntelliJ and File Manager |
| `apps/server/src/open.test.ts` | Added Nova assertion to `resolveEditorLaunch` test |

The `nova` CLI ships with Panic Nova (Extras → "Install Nova Command Line Tool"). Detection probes `$PATH` for it.

## Updating to a new upstream release

```bash
cd ~/Projects/t3code
git fetch upstream
git checkout personal/nova-editor
git rebase upstream/main
git push --force-with-lease
```

Conflicts are unlikely because the patch is small and additive. If upstream adds a new editor near our entry, just re-place the Nova entry next to its new neighbour.

## Building the macOS DMG (Apple Silicon)

```bash
bun install
bun scripts/build-desktop-artifact.ts --platform mac --target dmg --arch arm64
```

Output: `release/T3-Code-<version>-arm64.dmg`.

**Important**: do NOT use `bun run dist:desktop:dmg:arm64` — the npm script invokes the build via `node`, but the script uses `import.meta.main` (a Bun-ism) so nothing runs and you get exit code 0 with no output. Always invoke via `bun scripts/...` directly.

## Installing the built DMG

1. Drag the existing T3 Code app from `/Applications` to Trash.
2. Mount the DMG, drag the app to Applications.
3. First launch is blocked by Gatekeeper (unsigned ad-hoc build). Either:
   - Right-click the app → Open → Open, or
   - `xattr -dr com.apple.quarantine "/Applications/T3 Code (Alpha).app"`

The app is labelled "T3 Code (Alpha)" — that's the upstream dev branding, not related to our patch.

## Tests

```bash
cd ~/Projects/t3code/apps/server
bunx vitest run src/open.test.ts
```

Should print 15 passing tests including the Nova assertion.

Note: `bun test` at the repo root crashes with a Bun/vitest incompatibility (`vi.setSystemTime` panic). The proper way to run the full test suite is `bun run test --filter=t3` (uses Turbo). Two pre-existing GitManager tests time out on this machine — not related to our patch.

## Known issues / future investigation

- **Nova focus behaviour** (confirmed 2026-04-26): when Nova is already running, T3's "Open in Nova" button just brings the existing window to focus instead of switching to the project folder. Running `nova /path` from terminal works correctly — Nova opens the folder. So the issue is how Nova's CLI handles being invoked while the app is already running. Two possible fixes if this ever bothers us enough:
  1. Switch Nova's launch to use `open -na "Nova" --args "$path"` (macOS pattern for forcing a new instance) — would need a Nova-specific branch in `apps/server/src/open.ts`.
  2. Check `nova --help` for a new-window flag and add it to the editor entry's `baseArgs` in `packages/contracts/src/editor.ts`.
