# PdfCraft, patched

Builds of [PdfCraft](https://github.com/storytold/pdfcraft) with a few patches on top of
each official release. A scheduled workflow watches for new PdfCraft releases, applies the
patches, builds Windows and macOS, and publishes the result here under the same version tag.

## The patches

| Patch | What it does |
|---|---|
| 0001 | AI assistant: Summarize, Ask and Translate with Claude (by hermi72, from upstream PR #5744) |
| 0002 | AI assistant on Windows: finds `claude.exe` in `%USERPROFILE%\.local\bin`, runs it without a console window, prefers Claude Code over an API key |
| 0003 | Edit a PDF: marks the group Ready, so Watermark, Header and footer, Bates and Background appear in the menus |
| 0004 | AI assistant: the section title says it uses Claude Code, not an API key |
| 0005 | AI assistant in the browser build: asks the host page to run Claude Code (for a browser extension that provides `window.pdfcraftClaude`) |
| 0006 | Updates: the in-app updater reads **this** repository's releases, so updating keeps the patches. No startup update check on macOS (see below) |
| 0007 | Opening a file that is already open re-reads it: a newer version at the same path (an email attachment sent again under the same name) shows at once instead of the old tab |

## Downloads

[Releases](../../releases): `pdfcraft-<version>-windows-x64-portable.zip` (and an MSI), and
`pdfcraft-<version>-macos-aarch64.dmg` for Apple silicon. `SHA256SUMS.txt` lists every file.

The builds are **not code-signed**:

- **Windows:** SmartScreen may warn the first time. Use the portable zip. Once installed,
  Help ▸ Check for updates installs new patched releases from here.
- **macOS:** ad-hoc signed, so Gatekeeper refuses a DMG downloaded in a browser. Clear the
  quarantine flag after copying the app (`xattr -dr com.apple.quarantine /Applications/PdfCraft.app`).
  The built-in updater only installs bundles carrying the official Developer ID, so patched
  macOS builds don't check for updates at start; install new versions by hand.

## How it works

`.github/workflows/release.yml` runs every 4 hours:

1. `plan`: finds the latest PdfCraft release; stops if it's already built here. Otherwise clones
   it and checks that `git am --3way patches/*.patch` applies.
2. `windows`, `macos`: the same checkout and patches, then upstream's own packaging scripts
   (`packaging/windows/package.ps1`, `packaging/macos/package.sh`) with the release's
   craft-fonts and craft-i18n pins, unsigned.
3. `publish`: `SHA256SUMS.txt`, then a release under the upstream tag.

Run it by hand from the Actions tab (optionally with a tag, or "rebuild" to replace files).

## When a new PdfCraft release breaks a patch

The `plan` job fails and nothing is published; the previous patched release stays current.
To rebase:

```sh
git clone https://github.com/storytold/pdfcraft && cd pdfcraft
git checkout -b patched <new-tag>
git am --3way ../pdfcraft-patched/patches/*.patch   # fix conflicts, git am --continue
rm ../pdfcraft-patched/patches/*.patch
git format-patch --no-signature --zero-commit -o ../pdfcraft-patched/patches <new-tag>..
```

then commit the refreshed `patches/` and re-run the workflow.

## Licence

PdfCraft is MIT OR Apache-2.0 ([LICENSE-MIT](LICENSE-MIT), [LICENSE-APACHE](LICENSE-APACHE)).
The patches are offered under the same terms.
