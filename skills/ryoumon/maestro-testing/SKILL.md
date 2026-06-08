---
name: maestro-testing
description: >
  Comprehensive skill for writing and running mobile UI tests using Maestro
  (https://maestro.dev). Covers all YAML commands, selectors, flow control,
  JavaScript integration, CI/CD setup, cloud testing, debugging, and community
  best practices for iOS, Android, React Native, and Flutter apps.

  Use this skill whenever the user asks to:
  - Write, create, or generate Maestro test flows / YAML files
  - Run Maestro tests locally or on CI/CD
  - Set up Maestro for a project (iOS, Android, React Native, Flutter)
  - Debug failing Maestro tests or troubleshoot flakiness
  - Integrate Maestro with GitHub Actions, GitLab CI, or other CI systems
  - Use Maestro Cloud for device farm testing
  - Organize or refactor an existing Maestro test suite
  - Convert Appium/Detox/Espresso tests to Maestro
  - Use Maestro Studio, MaestroGPT, or MCP server for AI-assisted testing

  Triggers: "maestro test", "maestro flow", "mobile UI test", "e2e test mobile",
  "write a maestro test", "maestro yaml", "maestro CI/CD", "maestro cloud"
---

# Maestro Testing Skill

Guide for writing, running, and maintaining Maestro mobile UI tests. Maestro is an open-source framework that uses human-readable YAML syntax for cross-platform black-box testing of iOS, Android, and web apps.

## Core Philosophy

Maestro operates via the accessibility layer — like a real user. No source code access required. Tests are:
- **Declarative**: YAML describes *what* to do, not *how*
- **Tolerant**: Built-in flakiness handling, smart waiting, auto-retry
- **Fast**: No compilation, interpreted YAML with file watchers
- **Universal**: Works with React Native, Flutter, native iOS, native Android, hybrid

## Workflow: Writing a Maestro Test

Follow these steps when creating Maestro tests:

1. **Analyze the app screen** — Identify elements, flows, testIDs
2. **Write the YAML flow** — Use `references/commands-reference.md` for syntax
3. **Apply best practices** — Check `references/best-practices.md` for patterns
4. **Test locally** — Run with `maestro test Flow.yaml`
5. **Debug if needed** — Use `references/debugging-guide.md`
6. **Integrate to CI/CD** — Follow `references/ci-cd-integration.md`

## Quick Reference

### Flow File Anatomy

```yaml
# === Configuration Section ===
appId: com.example.myapp          # Required: bundle ID or package name
tags:                              # Optional: tags for filtering
  - smoke
  - auth
env:                               # Optional: flow-level constants
  TIMEOUT: 10000
  USER_EMAIL: "test@example.com"

---                                 # Separator: required

# === Commands Section ===
- launchApp:
    clearState: true
    permissions: { all: allow }

- tapOn:
    id: "login_button"
    optional: true                  # Skip if element not found

- inputText: "user@example.com"

- assertVisible: "Welcome"
```

### Essential Commands

| Task | Command | Example |
|------|---------|---------|
| Launch app | `launchApp` | `- launchApp: { clearState: true }` |
| Tap element | `tapOn` | `- tapOn: { id: "submit_btn" }` |
| Type text | `inputText` | `- inputText: "hello world"` |
| Assert visible | `assertVisible` | `- assertVisible: "Success"` |
| Swipe | `swipe` | `- swipe: { direction: UP }` |
| Scroll to element | `scrollUntilVisible` | `- scrollUntilVisible: { element: { text: "Footer" }, direction: DOWN }` |
| Run subflow | `runFlow` | `- runFlow: login.yaml` |
| Take screenshot | `takeScreenshot` | `- takeScreenshot: checkout_page` |
| Run JavaScript | `runScript` / `evalScript` | `- evalScript: ${output.count = 5}` |

### Selector Priority (Most to Least Reliable)

1. **`id`** — testID / resource-id / accessibilityIdentifier (most stable)
2. **`text`** with regex — `text: "Submit.*"` (Maestro uses regex by default)
3. **`text` exact match** — `text: "Submit"`
4. **Relational** — `below`, `above`, `containsChild` (anchor with stable element)
5. **`point`** — `"50%,50%"` (avoid when possible, use only as last resort)

**Critical**: Maestro treats `text` selectors as **regular expressions by default**. Escape special regex characters: `text: "Price: \$9\.99"`.

### Environment Variables

```yaml
# In flow: reference with ${VAR_NAME}
- inputText: ${USER_EMAIL}

# Via CLI: maestro test -e USER_EMAIL=user@example.com Flow.yaml
# Via shell: export MAESTRO_USER_EMAIL=user@example.com (auto-reads MAESTRO_* prefix)
# In config.yaml: env: { USER_EMAIL: user@example.com }
```

### Data Passing Between Flows

```yaml
# Parent flow sets env
- runFlow:
    file: login.yaml
    env:
      EMAIL: "user@example.com"

# Child flow reads via ${EMAIL}
# Child flow writes to output object
- evalScript: ${output.authToken = "abc123"}

# Parent flow reads output
- assertTrue: ${output.authToken.length > 0}
```

### JavaScript Integration

```yaml
# Inline expression
- evalScript: ${output.result = Math.random() > 0.5}

# External JS file
- runScript: utils/faker.js

# Conditional with JS
- tapOn:
    id: "premium_btn"
    when:
      true: ${output.isPremiumUser === true}
```

**Available in JS**: `output` (global data store), `maestro.copiedText`, `maestro.platform` (`ios`/`android`/`web`), `http.get/post/put/delete()`, `json()`.

**Important**: `output` must be assigned before use. Initialize in a setup script:
```javascript
// scripts/init.js
output.testData = {}
output.selectors = {}
```

## Key Resources (Load When Needed)

| File | When to Load |
|------|-------------|
| `references/commands-reference.md` | Need full command syntax, all parameters, or selector details |
| `references/best-practices.md` | Organizing test suite, writing maintainable tests, design patterns |
| `references/ci-cd-integration.md` | GitHub Actions, GitLab CI, Maestro Cloud setup |
| `references/debugging-guide.md` | Tests failing, flaky tests, element not found errors |
| `references/platform-specific.md` | iOS-only, Android-only, React Native, Flutter specifics |

## Project Structure Template

For a new Maestro test suite, create this structure:

```
maestro/
├── config.yaml                    # Test discovery, tags, execution order
├── .env                           # Environment variables (gitignored)
│
├── flows/
│   ├── auth/
│   │   ├── login.yaml
│   │   └── logout.yaml
│   ├── checkout/
│   │   ├── add_to_cart.yaml
│   │   └── complete_purchase.yaml
│   └── common/                    # Reusable subflows
│       ├── setup.yaml
│       ├── teardown.yaml
│       └── login_subflow.yaml     # Reusable login sequence
│
├── scripts/                       # JavaScript helpers
│   ├── selectors.js               # Page Object Model definitions
│   ├── faker.js                   # Test data generation
│   └── api.js                     # HTTP helper functions
│
└── artifacts/                     # Screenshots, videos, reports
```

## config.yaml Template

```yaml
# === Maestro Configuration ===
flows:
  - "flows/**/*.yaml"              # Glob pattern for test discovery

includeTags:                        # Default tags to include
  - smoke
excludeTags:                        # Default tags to exclude
  - skip
  - wip

executionOrder:
  flowsOrder:                       # Run order (optional)
    - flows/common/setup.yaml
    - flows/auth/login.yaml

env:
  BASE_URL: "https://api.example.com"
  TIMEOUT: 10000

platform:
  ios:
    disableAnimations: true
  android:
    disableAnimations: true
```

## Critical Best Practices (Summary)

- **Always use `id` selectors** (testID/resource-id) when available — most stable
- **Never use `sleep()`** — Maestro has built-in smart waiting
- **Use `runFlow` for shared sequences** — Login, setup, teardown as subflows
- **Tag utility flows with `skip`** — Prevent standalone execution of subflows
- **Pass data via `env` + `output`** — Explicit data flow between parent and child flows
- **Use `optional: true` for non-critical elements** — Ads, popups, one-time prompts
- **Escape regex in text selectors** — Maestro uses regex matching by default
- **Platform conditionals**: `when: { platform: ios }` for platform-specific steps
- **Test locally with `--continuous`** — Auto-rerun on file save during development
- **Always use named CLI flags in CI** — `--app-file`, `--flows` instead of positional args

## Anti-Patterns to Avoid

| Anti-Pattern | Better Approach |
|-------------|-----------------|
| `tapOn: "50%,50%"` (coordinates) | `tapOn: { id: "button" }` or `tapOn: "Submit"` |
| Hard-coded `sleep: 2000` | Rely on Maestro's auto-wait or use `extendedWaitUntil` |
| Monolithic 500-line flows | Break into `runFlow` subflows |
| Duplicate login in every test | Reusable `login_subflow.yaml` via `runFlow` |
| Copy-paste selectors everywhere | Centralize in `scripts/selectors.js` (POM pattern) |
| Platform-specific flows for everything | Use `when: { platform: ios/android }` in single flow |

## CLI Quick Reference

```bash
# Run a single flow
maestro test Flow.yaml

# Run with app file
maestro test --app-file app.apk Flow.yaml

# Run all flows in directory
maestro test flows/

# Run with tags
maestro test --include-tags smoke --exclude-tags skip flows/

# Watch mode (rerun on save)
maestro test --continuous Flow.yaml

# Sharded parallel execution
maestro test --shards 4 --shard-split flows/

# JUnit XML report
maestro test --format junit --output report.xml flows/

# Interactive studio
maestro studio

# View hierarchy for debugging
maestro hierarchy

# Cloud upload
maestro cloud --api-key $KEY --project-id $ID app.apk flows/

# MaestroGPT chat
maestro chat
```

## Troubleshooting Quick Fixes

| Problem | Solution |
|---------|----------|
| "Element not found" | Run `maestro hierarchy` → verify selector → try `scrollUntilVisible` |
| Test flaky | Add `retry: { maxRetries: 2 }` or use `extendedWaitUntil` |
| iOS keychain data persists | Use `clearKeychain` (separate from `clearState`) |
| Keyboard won't hide | Use `hideKeyboard: "Done"` then `pressKey: Escape` with `optional: true` on each |
| Slow tests | Disable animations in `config.yaml` (`disableAnimations: true`) |
| Unicode on Android | Not supported — use ASCII only for `inputText` on Android |
| Subflow can't access parent's env | Pass explicitly via `runFlow: { env: { ... } }` |
| `optional: true` not working | Not supported on `scrollUntilVisible` — wrap in `runFlow` instead |

## AI-Assisted Testing (Modern Maestro)

Maestro CLI 2.4.0+ ships with MCP server and MaestroGPT:

- **`maestro chat`** — Natural language to YAML command generation
- **MCP Server** — `maestro mcp` for Claude/Cursor/Copilot integration
- **Maestro Studio** — Visual IDE with element inspector and AI command generation
- **Maestro Viewer** — Web-based device visualizer (CLI 2.6.0+)

For AI agent integration, the MCP server exposes tools: `run`, `run_on_cloud`, `take_screenshot`, `open_maestro_viewer`, etc.
