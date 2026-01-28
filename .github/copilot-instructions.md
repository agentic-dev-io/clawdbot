# Clawdbot - GitHub Copilot Instructions

## Repository Overview

**Clawdbot** is a personal AI assistant that runs on your own devices. It's a WhatsApp/Telegram/Discord/Slack gateway with multi-channel support, featuring a Pi-based RPC agent. The project provides CLI tools, mobile apps (iOS/Android), macOS menu bar app, and web interfaces.

- **Repository size**: ~1.9GB (includes dependencies, build artifacts, submodules)
- **Primary language**: TypeScript (ESM modules)
- **Runtime**: Node.js ≥22.12.0 (officially); tested with Node 20+ (unsupported engine warnings are normal)
- **Package manager**: pnpm 10.23.0 (via corepack)
- **Alternative runtimes**: Bun supported for development/testing
- **Framework**: Node.js with TypeScript, Swift (iOS/macOS), Kotlin (Android)
- **Test framework**: Vitest with V8 coverage provider
- **Linting**: oxlint (type-aware)
- **Formatting**: oxfmt
- **Workspace**: pnpm workspace (monorepo) with extensions as separate packages

## Build & Development Commands

### Essential Prerequisites

**CRITICAL**: Always run these commands in order when setting up or after pulling changes:

```bash
# 1. Enable corepack and activate pnpm (required before any pnpm commands)
corepack enable
corepack prepare pnpm@10.23.0 --activate

# 2. Install dependencies (frozen lockfile in CI)
pnpm install --frozen-lockfile --config.engine-strict=false

# 3. Build the project (TypeScript compilation + post-build scripts)
pnpm build

# Note: UI build is separate and required for prepack
pnpm ui:build  # Only needed for npm packaging
```

**Common Issues**:
- `command 'pnpm' not found`: Run the corepack commands first
- `WARN Unsupported engine`: Safe to ignore when using Node 20.x (project targets 22.12+)
- `Failed to create bin at extensions/*/node_modules/.bin/clawdbot`: Expected before first build (dist/entry.js doesn't exist yet)

### Build Commands (Validated)

```bash
# Full build (TypeScript + post-build steps)
pnpm build  # ~10-30s
# Compiles: tsc -p tsconfig.json
# Then runs: canvas-a2ui-copy.ts, copy-hook-metadata.ts, write-build-info.ts

# Protocol generation (required for macOS/iOS apps)
pnpm protocol:gen        # Generates dist/protocol.schema.json
pnpm protocol:gen:swift  # Generates Swift protocol models
pnpm protocol:check      # Validates generated files are up-to-date (CI check)

# UI build (separate from main build)
pnpm ui:install  # Install UI dependencies (first time)
pnpm ui:build    # Build control UI (required for npm packaging)
pnpm ui:dev      # Development server for UI
```

### Linting & Formatting (Validated)

```bash
# TypeScript/JavaScript linting (type-aware)
pnpm lint        # oxlint --type-aware src test (~7s on 2479 files)
pnpm lint:fix    # Format + fix auto-fixable issues

# Formatting checks
pnpm format      # oxfmt --check src test (~1s on 2487 files)
pnpm format:fix  # oxfmt --write src test

# Swift linting (macOS/iOS)
pnpm lint:swift   # swiftlint + swiftformat lint
pnpm lint:all     # All linters (TS + Swift)
pnpm format:swift # swiftformat lint (apps/macos/Sources, apps/ios/Sources)
pnpm format:all   # All formatters (TS + Swift)
```

### Testing (Validated)

**IMPORTANT**: Tests can take 2+ minutes. Use targeted test runs for faster feedback.

```bash
# Unit tests (default - uses vitest.config.ts)
pnpm test           # Parallel tests (~120s timeout, 4-16 workers)
pnpm test:watch     # Watch mode for development
pnpm test:coverage  # With coverage report (70% thresholds: lines/functions/branches/statements)

# Specific test configs
pnpm test:e2e       # E2E tests (vitest.e2e.config.ts)
pnpm test:live      # Live API tests (requires CLAWDBOT_LIVE_TEST=1)
pnpm test:ui        # UI tests (in ui/ directory)
pnpm test:force     # Force test run with custom logic

# Docker-based integration tests
pnpm test:docker:onboard           # Onboarding E2E
pnpm test:docker:gateway-network   # Gateway network tests
pnpm test:docker:live-models       # Live model tests in Docker
pnpm test:docker:live-gateway      # Live gateway model tests
pnpm test:docker:plugins           # Plugin tests
pnpm test:docker:cleanup           # Cleanup test
pnpm test:docker:all               # All Docker tests (long-running)

# Full test suite (use with caution - very slow)
pnpm test:all  # lint + build + test + e2e + live + docker:all
```

**Test Configuration Files**:
- `vitest.config.ts` - Main unit tests (src/**/*.test.ts, extensions/**/*.test.ts)
- `vitest.e2e.config.ts` - E2E tests (*.e2e.test.ts)
- `vitest.live.config.ts` - Live API tests (*.live.test.ts)
- `vitest.gateway.config.ts` - Gateway-specific tests
- `vitest.unit.config.ts` - Pure unit tests
- `vitest.extensions.config.ts` - Extension tests

### Mobile App Development

```bash
# iOS (requires macOS with Xcode 26.1+)
pnpm ios:gen    # Generate Xcode project with xcodegen
pnpm ios:open   # Generate + open project in Xcode
pnpm ios:build  # Build for simulator (iPhone 17 by default)
pnpm ios:run    # Build + launch in simulator

# Android (requires Java 21, Android SDK 36)
pnpm android:assemble  # Build APK (./gradlew :app:assembleDebug)
pnpm android:install   # Install APK to connected device
pnpm android:run       # Install + launch app
pnpm android:test      # Run unit tests (./gradlew :app:testDebugUnitTest)

# macOS app
pnpm mac:package  # Package app (scripts/package-mac-app.sh)
pnpm mac:restart  # Restart mac gateway (scripts/restart-mac.sh)
pnpm mac:open     # Open packaged app (dist/Clawdbot.app)
```

### Development Runtime Commands

```bash
# CLI development (uses scripts/run-node.mjs with bun/tsx)
pnpm clawdbot [args]      # Run CLI via development wrapper
pnpm dev [args]           # Alias for clawdbot

# Gateway (server/daemon)
pnpm gateway:watch        # Watch mode with auto-reload
pnpm gateway:dev          # Dev mode (skips channel init)
pnpm gateway:dev:reset    # Dev mode with reset

# TUI (Terminal UI)
pnpm tui          # Launch TUI
pnpm tui:dev      # TUI with dev profile

# Other runtimes
pnpm start        # Run CLI (same as dev)
pnpm clawdbot:rpc # Agent in RPC mode with JSON output
```

## Project Structure

### Root Directory

```
.github/           CI/CD workflows, issue templates
apps/              Mobile apps (ios/, android/, macos/, shared/)
assets/            Static assets (images, icons)
dist/              Build output (TypeScript compiled to JS)
docs/              Documentation (Mintlify-based, hosted at docs.clawd.bot)
extensions/        Extension packages (channels: matrix, msteams, line, zalo, etc.)
git-hooks/         Git hooks (installed by postinstall)
patches/           pnpm patches for dependencies
scripts/           Build/dev/test scripts (Node.js, Shell)
skills/            Bundled AI skills
src/               Source code (TypeScript)
test/              Test helpers and fixtures
ui/                Control UI (separate package)
vendor/            Vendored dependencies
```

### Source Code Structure (`src/`)

```
src/
  agents/           AI agent core (Pi agent integration, tools, compaction)
  browser/          Browser automation (Playwright)
  canvas-host/      Canvas rendering system
  channels/         Channel abstractions and routing
  cli/              CLI framework and commands
  commands/         CLI command implementations
  config/           Configuration system
  cron/             Scheduled tasks
  discord/          Discord channel integration
  gateway/          Gateway server (RPC, WebSocket, HTTP)
  hooks/            Lifecycle hooks system
  imessage/         iMessage channel integration
  infra/            Infrastructure (Tailscale, networking)
  logging/          Structured logging
  media/            Media processing (images, video, audio)
  memory/           Memory/storage systems
  pairing/          Device pairing logic
  plugins/          Plugin system
  plugin-sdk/       Plugin SDK (exported as clawdbot/plugin-sdk)
  providers/        AI model providers
  routing/          Message routing
  security/         Security utilities
  sessions/         Session management
  signal/           Signal channel integration
  slack/            Slack channel integration
  telegram/         Telegram channel integration
  terminal/         Terminal UI components
  tts/              Text-to-speech
  tui/              Terminal UI application
  web/              WhatsApp Web (Baileys)
  whatsapp/         WhatsApp integration
  wizard/           Onboarding wizard
  entry.ts          CLI entry point (exported as bin)
  index.ts          Main package export
```

### Configuration Files

- `tsconfig.json` - TypeScript compiler config (ES2022, NodeNext modules, strict mode)
- `.oxlintrc.json` - oxlint configuration (unicorn, typescript, oxc plugins)
- `.oxfmtrc.jsonc` - oxfmt formatting (indent: 2, print width: 100)
- `pnpm-workspace.yaml` - Workspace config (root, ui, extensions/*)
- `vitest.*.config.ts` - Test configurations (multiple configs for different test types)
- `.pre-commit-config.yaml` - Pre-commit hooks (install with `prek install`)
- `.swiftlint.yml` - Swift linting rules
- `.swiftformat` - Swift formatting rules
- `package.json` - Main package metadata and scripts

## CI/CD Pipeline

### GitHub Actions Workflows

**Main CI Workflow** (`.github/workflows/ci.yml`):

Jobs run in parallel:
1. **install-check** (blacksmith-4vcpu-ubuntu-2404)
   - Checkout with submodule retry logic (5 attempts)
   - Setup Node.js 22.x
   - Setup pnpm via corepack (retry logic, 3 attempts)
   - Install dependencies with frozen lockfile

2. **checks** (blacksmith-4vcpu-ubuntu-2404, matrix build)
   - Matrix: [lint, test, build, protocol, format] × [node, bun]
   - Validates both Node and Bun compatibility
   - All checks must pass

3. **secrets** (blacksmith-4vcpu-ubuntu-2404)
   - Python 3.12 + detect-secrets 1.5.0
   - Scans for secrets using baseline (.secrets.baseline)
   - Failure message: "See docs/gateway/security.md#secret-scanning-detect-secrets"

4. **checks-windows** (blacksmith-4vcpu-windows-2025)
   - Matrix: [lint, test, build, protocol]
   - Tests Windows compatibility (bash shell)
   - NODE_OPTIONS: --max-old-space-size=4096

5. **checks-macos** (macos-latest, PRs only)
   - Runs tests on macOS
   - Validates platform-specific behavior

6. **macos-app** (macos-latest, PRs only)
   - Matrix: [lint, build, test] for Swift code
   - Xcode 26.1 required
   - Tools: xcodegen, swiftlint, swiftformat
   - Build: `swift build --package-path apps/macos --configuration release`
   - Test: `swift test --package-path apps/macos --parallel --enable-code-coverage`
   - Retry logic: 3 attempts with 20s delay

7. **android** (blacksmith-4vcpu-ubuntu-2404)
   - Matrix: [test, build]
   - Java 21 (temurin distribution)
   - Gradle 8.11.1
   - Android SDK 36, build-tools 36.0.0
   - Test: `./gradlew --no-daemon :app:testDebugUnitTest`
   - Build: `./gradlew --no-daemon :app:assembleDebug`

**Critical CI Notes**:
- **Submodules**: All jobs use retry logic (5 attempts, exponential backoff)
- **pnpm setup**: Uses corepack with retry logic (3 attempts)
- **Install command**: `pnpm install --frozen-lockfile --ignore-scripts=false --config.engine-strict=false --config.enable-pre-post-scripts=true`
- **Install retries**: Install command runs twice if first attempt fails
- **iOS tests**: Currently disabled in CI (`if: false`)

### Pre-commit Hooks

Install with: `prek install` (not `pre-commit install`)

Hooks validate:
- Trailing whitespace, EOF newlines
- YAML syntax
- Large files (500KB max)
- Merge conflicts
- Secret detection (same as CI)
- Shell scripts (shellcheck, errors only)
- GitHub Actions (actionlint)
- GitHub Actions security (zizmor)
- oxlint, oxfmt (same as CI)
- swiftlint, swiftformat (same as CI)

## Common Development Patterns

### Making Code Changes

1. **Always run `pnpm install` after pulling changes** (lockfile may have changed)
2. **Run `pnpm build` after install** (ensures dist/ is up-to-date)
3. **Run `pnpm lint` before committing** (catches type errors early)
4. **Run `pnpm format` to check formatting** (or `pnpm format:fix` to auto-fix)
5. **Run targeted tests** during development (avoid full test suite until ready)
6. **Run `pnpm protocol:check`** if modifying protocol/types used by Swift apps

### Debugging Build Issues

**Issue**: "Command 'pnpm' not found"
- **Fix**: Run `corepack enable && corepack prepare pnpm@10.23.0 --activate`

**Issue**: "WARN Unsupported engine" (Node 20.x vs required 22.12.0+)
- **Fix**: Warnings are safe to ignore for development; CI uses Node 22

**Issue**: "Failed to create bin at extensions/*/node_modules/.bin/clawdbot"
- **Fix**: Expected before first build; run `pnpm build` to create dist/entry.js

**Issue**: Protocol check fails
- **Fix**: Run `pnpm protocol:gen && pnpm protocol:gen:swift` to regenerate

**Issue**: Tests timeout
- **Fix**: Tests can take 120+ seconds; increase timeout or run specific test files

**Issue**: Submodule errors
- **Fix**: Run `git submodule sync --recursive && git submodule update --init --force --depth=1 --recursive`

### Extension Development

Extensions are in `extensions/*/` and are separate pnpm workspace packages. When adding dependencies to an extension:
1. Navigate to extension directory: `cd extensions/my-extension`
2. Install with pnpm: `pnpm add <package>`
3. **Do not add extension-specific deps to root package.json**
4. Extension runtime dependencies must be in `dependencies` (not `devDependencies`)
5. Avoid `workspace:*` in extension dependencies (breaks npm install)

### Important Notes for AI Agents

1. **Never modify `node_modules/`** - Changes are lost on reinstall
2. **Always use frozen lockfile in CI** - `--frozen-lockfile` prevents drift
3. **Protocol files are generated** - Don't edit `dist/protocol.schema.json` or `apps/**/GatewayModels.swift` manually
4. **Coverage thresholds** - 70% lines/functions/statements, 55% branches (vitest.config.ts)
5. **Test file naming** - `*.test.ts` for unit, `*.e2e.test.ts` for E2E, `*.live.test.ts` for live API tests
6. **Windows CI** - Uses Bash shell (via Git Bash), watch for path separators
7. **macOS CI** - Only runs on PRs, not all pushes (resource optimization)
8. **Submodules** - Always use retry logic when checking out (network flakiness common)
9. **Postinstall script** - Runs on install, sets up git hooks and applies patches (scripts/postinstall.js)
10. **Canvas A2UI bundle** - Auto-generated hash file (src/canvas-host/a2ui/.bundle.hash), ignore unexpected changes

### File Size Guidelines

- Keep TypeScript files under ~500 LOC when feasible (guideline, not hard limit)
- Refactor/split large files instead of creating "V2" copies
- Use existing patterns for CLI options and dependency injection

### Testing Strategy

- **Unit tests** (*.test.ts): Test individual functions/modules in isolation
- **E2E tests** (*.e2e.test.ts): Test full user workflows
- **Live tests** (*.live.test.ts): Test against real APIs (requires env vars)
- **Docker tests**: Integration tests in isolated containers
- **Coverage exclusions**: Entry points, CLI wiring, manual/e2e flows, interactive UIs (see vitest.config.ts)

### Security

- **Secret scanning**: CI runs detect-secrets on all commits
- **Baseline file**: `.secrets.baseline` contains approved patterns
- **Never commit**: Real API keys, phone numbers, credentials, videos, device names
- **Docs**: Use generic placeholders (e.g., `user@gateway-host`, not real hostnames)

## Quick Reference

**Essential commands to remember**:
```bash
corepack enable && corepack prepare pnpm@10.23.0 --activate  # First-time setup
pnpm install --frozen-lockfile --config.engine-strict=false   # Install deps (CI mode)
pnpm build                                                     # Build project
pnpm lint                                                      # Lint code
pnpm format                                                    # Check formatting
pnpm test                                                      # Run tests (may take 2+ min)
pnpm protocol:check                                            # Verify protocol files
pnpm test:all                                                  # Full validation (slow!)
```

**Trust these instructions**: This document is validated against the current codebase. Only search for additional information if instructions are incomplete or incorrect. The build and test commands above have been verified to work correctly on Ubuntu 24.04 with Node 20.20.0.
