# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Debug build for development (use Swift 6.2 on Intel Macs)
export PATH="/usr/local/opt/swift/bin:$PATH" && swift build -c debug

# Release build
export PATH="/usr/local/opt/swift/bin:$PATH" && swift build -c release

# Package app (builds CodexBar.app in-place)
export PATH="/usr/local/opt/swift/bin:$PATH" && CODEXBAR_SIGNING=adhoc ./Scripts/package_app.sh

# Ad-hoc signing (no Apple Developer account needed)
export PATH="/usr/local/opt/swift/bin:$PATH" && CODEXBAR_SIGNING=adhoc ./Scripts/package_app.sh

# Development loop: compile and run
export PATH="/usr/local/opt/swift/bin:$PATH" && ./Scripts/compile_and_run.sh

# Run tests
export PATH="/usr/local/opt/swift/bin:$PATH" && swift test

# Format and lint
swiftformat .
swiftlint
```

## macOS 14+ & Build Notes

- **Intel Macs**: Requires Swift 6.2+ (install from swift.org and use `export PATH="/usr/local/opt/swift/bin:$PATH"` before builds)
- **Apple Silicon**: Use system Swift (no PATH adjustment needed)

## Project Architecture

CodexBar is a SwiftPM-based macOS 14+ menu bar app with modular targets:

- **CodexBarCore**: Fetching, parsing, and provider implementations (browser cookies, CLI probes, API clients)
- **CodexBar**: Main app (SwiftUI + AppKit), StatusItemController, menus, icon rendering
- **CodexBarWidget**: WidgetKit extension mirroring the menu card snapshot
- **CodexBarCLI**: Bundled CLI (`codexbar`) for scripts and CI
- **CodexBarMacros/MacroSupport**: SwiftSyntax macros for provider registration
- **CodexBarClaudeWatchdog/ClaudeWebProbe**: Helper processes for Claude CLI stability

Key data flow:
- Background refresh → UsageFetcher/provider probes → UsageStore → menu/icon/widgets
- SettingsStore controls refresh cadence and feature flags

Concurrency: Swift 6 strict concurrency enabled with Sendable state and explicit MainActor hops.

## Key Files

- `Package.swift`: SwiftPM configuration, platform targets (macOS 14+)
- `Sources/CodexBar/StatusItemController.swift`: Main menu bar controller
- `Sources/CodexBar/UsageStore.swift`: Central state for usage data
- `Sources/CodexBar/Providers/Shared/ProviderImplementation.swift`: Provider protocol
- `Sources/CodexBarCore/CostUsageFetcher.swift`: Local cost tracking from logs

## Providers

10+ provider implementations (Codex, Claude, Cursor, Gemini, Antigravity, Factory, Copilot, z.ai, Kiro, Vertex AI, Augment). Each provider lives in `Sources/CodexBar/Providers/<Name>/` with login flow and usage fetching.

## Release Process

See `docs/RELEASING.md` for full checklist. Key steps:

1. **Build universal binary**: Run on Apple Silicon Mac with `./Scripts/sign-and-notarize.sh` (builds both arm64 + x86_64)
   - Intel-only build: `export PATH="/usr/local/opt/swift/bin:$PATH" && CODEXBAR_SIGNING=adhoc ./Scripts/package_app.sh`
2. Generate appcast: `./Scripts/make_appcast.sh`
3. Update Homebrew cask at `../homebrew-tap/Casks/codexbar.rb` (remove `depends_on arch: :arm64` for Intel support)

The build scripts support universal binaries but require host architecture libraries. Build on Apple Silicon for full universal binaries.

## Documentation

- `docs/architecture.md`: Module breakdown
- `docs/providers.md`: Provider overview
- `docs/refresh-loop.md`: Update cycle
- `docs/ui.md`: Icon and menu design
- `docs/cli.md`: CLI reference
