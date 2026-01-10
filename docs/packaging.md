---
summary: "Packaging, signing, and bundled CLI notes."
read_when:
  - Packaging/signing builds
  - Updating bundle layout or CLI bundling
---

# Packaging & signing

## Prerequisites

**Swift 6.2+ required** - Check with `swift --version`. For earlier versions, install via Homebrew:

```bash
brew install swift@6.2
export PATH="/usr/local/opt/swift@6.2/bin:$PATH"
```

## Scripts
- `Scripts/package_app.sh`: builds host arch by default; accepts `ARCHES` for multi-arch builds. See Architecture options below.
- `Scripts/compile_and_run.sh`: uses host arch; pass `--release-universal` or `--release-arches="arm64 x86_64"` for release packaging.
- `Scripts/sign-and-notarize.sh`: signs, notarizes, staples, zips (accepts `ARCHES` for universal).
- `Scripts/make_appcast.sh`: generates Sparkle appcast and embeds HTML release notes.
- `Scripts/changelog-to-html.sh`: converts the per-version changelog section to HTML for Sparkle.

## Architecture options

### Apple Silicon (arm64)
```bash
./Scripts/package_app.sh                    # host arch (arm64 on Apple Silicon)
```

### Intel (x86_64)
```bash
ARCHES="x86_64" ./Scripts/package_app.sh    # Intel only
```

### Universal binary (arm64 + x86_64)
```bash
./Scripts/package_app.sh                    # defaults to universal
ARCHES="arm64 x86_64" ./Scripts/package_app.sh  # explicit universal
```

### Verify architecture
```bash
file CodexBar.app/Contents/MacOS/CodexBar
lipo -info CodexBar.app/Contents/MacOS/CodexBar  # for universal
```

## Bundle contents
- `CodexBarWidget.appex` bundled with app-group entitlements.
- `CodexBarCLI` copied to `CodexBar.app/Contents/Helpers/` for symlinking.
- SwiftPM resource bundles (e.g. `KeyboardShortcuts_KeyboardShortcuts.bundle`) copied into `Contents/Resources` (required for `KeyboardShortcuts.Recorder`).

## Releases
- Full checklist in `docs/RELEASING.md`.

See also: `docs/sparkle.md`.
