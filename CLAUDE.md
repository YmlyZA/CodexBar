# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

CodexBar is a macOS 14+ menu bar application that monitors API usage limits and quotas for multiple AI/LLM providers (Codex, Claude, Cursor, Gemini, Antigravity, Droid/Factory, Copilot, z.ai, Kiro, Vertex AI, and Augment). Built with Swift 6 strict concurrency.

**Requirements:** Swift 6.2+, macOS 14+

## Development Commands

### Prerequisites & Swift Version
CodexBar requires **Swift 6.2 or later**. Check with `swift --version`.

If you have an earlier version, install via Homebrew:
```bash
brew install swift@6.2
export PATH="/usr/local/opt/swift@6.2/bin:$PATH"
```

### Build & Run
```bash
# Primary dev loop - kills old instances, builds, tests, packages, and launches
./Scripts/compile_and_run.sh

# Individual commands
swift build                    # Debug build
swift build -c release         # Release build
swift test                     # Run all tests
./Scripts/package_app.sh       # Package as CodexBar.app
CODEXBAR_ALLOW_LLDB=1 ./Scripts/package_app.sh  # Debug build with LLDB
```

### Architecture-Specific Builds
```bash
# Intel (x86_64) only
ARCHES="x86_64" ./Scripts/package_app.sh

# Universal binary (arm64 + x86_64) - default behavior
./Scripts/package_app.sh

# Verify architecture
file CodexBar.app/Contents/MacOS/CodexBar
lipo -info CodexBar.app/Contents/MacOS/CodexBar  # for universal
```

### Code Quality
```bash
pnpm check                     # Run both format check and lint
swiftformat Sources Tests      # Format code
swiftlint --strict             # Lint code
```

### Testing
```bash
swift test                              # Full suite
swift test --filter TTYIntegrationTests  # Filter specific tests
LIVE_TEST=1 swift test --filter LiveAccountTests  # Live API tests
```

### App Restart After Build
```bash
# Guaranteed restart with fresh bundle (update path as needed)
pkill -x CodexBar || pkill -f CodexBar.app || true
cd /Users/ksmac/github.com/CodexBar && open -n CodexBar.app
```

## Architecture

### Module Structure

```
Sources/
├── CodexBar/              # Main app (UI, state management, menu bar)
├── CodexBarCore/          # Business logic (fetchers, parsers, models)
├── CodexBarCLI/           # Command-line interface
├── CodexBarWidget/        # WidgetKit extension
├── CodexBarMacros/        # SwiftSyntax macros for provider registration
├── CodexBarMacroSupport/  # Shared macro support
├── CodexBarClaudeWatchdog/  # Helper process for Claude CLI
└── CodexBarClaudeWebProbe/   # CLI helper for web diagnostics
```

**Core Separation:** CodexBar handles UI/state management, CodexBarCore contains all provider fetchers, parsers, and models.

### Provider System (Macro-Driven Registration)

Providers use Swift macros to auto-register - no manual list maintenance needed:

```swift
@ProviderDescriptorRegistration
@ProviderDescriptorDefinition
public enum ExampleProviderDescriptor {
    static func makeDescriptor() -> ProviderDescriptor { ... }
}
```

Each provider has:
- **Descriptor:** Labels, URLs, defaults, fetch pipeline
- **Fetch Strategy:** Concrete data fetching implementation
- **Implementation:** Settings/login UI hooks only

Providers support multiple data sources with automatic fallback (e.g., OAuth API → Web API → CLI PTY).

### Data Flow

```
Background Refresh Timer
    ↓
UsageFetcher / Provider Strategies
    ↓
UsageStore (MainActor @Observable)
    ↓
StatusItemController (Icon Rendering)
    ↓
Menu Display + WidgetKit
```

**Key Stores:**
- `UsageStore`: Central state for all provider usage data
- `SettingsStore`: User preferences with UserDefaults persistence
- `ProviderToggleStore`: Provider enablement state

### Concurrency Model

- Swift 6 Strict Concurrency: `@MainActor` for UI, `Sendable` for state
- Observation tracking via `withObservationTracking` for reactive updates
- Background refresh uses `Task.detached` for non-blocking work

## Code Style

- **Indentation:** 4 spaces
- **Line length:** 120 characters (warning at 120, error at 250)
- **Explicit `self`:** Required for Swift 6 concurrency - do not remove
- **MARK comments:** Use for organization
- **SwiftUI:** Prefer `@Observable` models with `@State` ownership and `@Bindable` in views; avoid `ObservableObject`, `@ObservedObject`, `@StateObject`

## Critical Constraints

### Provider Identity Siloing
**Never display identity/plan from provider A in provider B's UI.** Each provider's account info must stay isolated.

### Claude CLI Status Line
Custom and user-configurable - never reliable for usage parsing. Use dedicated probes instead.

### Platform Gating
Use `#if os(macOS)` for platform-specific code (browser cookies, web dashboard, etc.). Linux CLI builds exclude UI components.

### Keychain Migration
One-time migration on first launch converts to `kSecAttrAccessibleAfterFirstUnlockThisDeviceOnly` to prevent prompts on development rebuilds.

## Testing

- Add/extend XCTest cases under `Tests/CodexBarTests/*Tests.swift`
- Mirror new logic with focused tests (usage parsing, status probes, icon patterns, cost usage)
- Run `swift test` before handoff

## Documentation

- `docs/architecture.md` - Module breakdown
- `docs/provider.md` - Provider authoring guide
- `docs/refresh-loop.md` - Background update behavior
- `docs/providers.md` - Per-provider data source details
- `docs/cli.md` - CLI reference
- `docs/DEVELOPMENT.md` - Development setup + troubleshooting

## Important Files

- `Sources/CodexBar/CodexbarApp.swift` - App initialization
- `Sources/CodexBar/UsageStore.swift` - Central state management
- `Sources/CodexBar/StatusItemController.swift` - Menu bar controller
- `Sources/CodexBar/SettingsStore.swift` - User preferences
- `Package.swift` - Dependencies and targets
