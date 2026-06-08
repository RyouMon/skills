# Maestro Debugging & Troubleshooting Guide

Complete guide for debugging failing Maestro tests and resolving common issues.

## Table of Contents

1. [Debugging Tools](#debugging-tools)
2. [Common Issues & Solutions](#common-issues--solutions)
3. [Flaky Tests](#flaky-tests)
4. [Platform-Specific Issues](#platform-specific-issues)
5. [Known Limitations](#known-limitations)
6. [Getting Help](#getting-help)

---

## Debugging Tools

### Maestro Studio (Interactive IDE)

Maestro Studio is a desktop application for building and debugging tests visually.

```bash
# Launch Studio
maestro studio

# Features:
# - Visual element inspector (right-click to generate commands)
# - Step-by-step test execution
# - MaestroGPT integration (AI command generation)
# - Interactive command building
```

> **Note**: Maestro Studio cannot run simultaneously with CLI tests (driver timeout conflict). Close Studio before running CLI commands.

### View Hierarchy Inspection

```bash
# Dump full view hierarchy to terminal
maestro hierarchy

# Use for:
# - Finding element IDs, text, and properties
# - Debugging "Element not found" errors
# - Understanding the UI structure
```

### Element Query

```bash
# Search for specific elements in the current view
maestro query "Submit"
maestro query --id "login_button"
```

### Screenshots on Failure

Maestro automatically captures screenshots on test failure:

```bash
# Saved to: ~/.maestro/tests/
# Or specify output directory:
maestro test --output ./reports Flow.yaml

# Manual screenshot in flow:
- takeScreenshot: before_checkout
```

### Video Recording

```bash
# Record test execution
maestro record --local Flow.yaml

# In-flow recording (max 2 minutes):
- startRecording: test_name
# ... test commands ...
- stopRecording
```

> Use `--local` flag for local rendering (recommended over remote rendering).

### Debug Logging

```bash
# Enable debug output
maestro test --debug-output Flow.yaml

# Generates maestro.log in working directory
# Known bug in v1.39.0: DEBUG logs included by default, causing 500x log growth
```

### Network Inspection

Maestro has limited native network inspection. Use external tools:

- **Proxy tools**: Charles Proxy, Proxyman, Wireshark
- **Mock servers**: WireMock, Mocks Server + Docker
- **In-app logging**: Add network logging to your app for test builds

---

## Common Issues & Solutions

### "Element not found" Error

**Systematic debugging workflow:**

1. **Run `maestro hierarchy`** — Verify element exists in the current view
2. **Check selector accuracy** — Verify id/text matches exactly (case-sensitive)
3. **Check regex escaping** — Maestro uses regex for text. Escape special chars:
   ```yaml
   # BAD — "$" is regex end-of-string
   - tapOn: "Price: $9.99"
   # GOOD — Escape special regex characters
   - tapOn: "Price: \$9\.99"
   ```
4. **Try scrolling first** — Element may be below the fold:
   ```yaml
   - scrollUntilVisible:
       element:
         text: "Target"
       direction: DOWN
       timeout: 10000
   ```
5. **Check WebView context** — Maestro sees native layer, not WebView content
6. **Check iOS accessible prop** — Set `accessible={false}` on parent containers

### Test is Flaky (Intermittent Failures)

**Built-in tolerance**: Maestro handles ~80% of flakiness automatically.

**For remaining flakiness:**

```yaml
# Strategy 1: Retry wrapper
- retry:
    maxRetries: 3
    commands:
      - tapOn:
          id: "unstable_button"
      - assertVisible: "Success"

# Strategy 2: Extended wait
- extendedWaitUntil:
    visible:
      id: "dynamic_content"
    timeout: 30000

# Strategy 3: Optional tap for race conditions
- tapOn:
    id: "maybe_present_banner"
    optional: true
```

**Causes of flakiness:**
- Network timing variations
- Animation timing
- Lazy-loaded content
- Race conditions with background processes
- Platform-specific timing differences

### iOS Keychain Data Persists Between Tests

`clearState` does NOT clear iOS Keychain. Use `clearKeychain`:

```yaml
# iOS: Explicitly clear keychain
- clearKeychain
- launchApp:
    clearState: true
```

### Keyboard Won't Hide

```yaml
# Try multiple approaches (each with optional in case it doesn't apply)
- hideKeyboard: "Done"
- hideKeyboard
- pressKey: Escape
  optional: true

# Or tap on a neutral area to dismiss
- tapOn:
    id: "screen_container"
    optional: true
```

> Known limitation: `hideKeyboard` is unreliable on iOS. It does NOT support `optional: true` — use `pressKey: Escape` as fallback instead.

### Unicode Characters on Android

**Not supported** for `inputText` on Android. Use ASCII only:

```yaml
# BAD on Android — Unicode characters
- inputText: "Hello 世界"

# Workaround — Use JavaScript to generate ASCII
- evalScript: ${output.name = "Test User"}
- inputText: ${output.name}
```

### Slow Test Execution

```yaml
# config.yaml — Disable animations
platform:
  ios:
    disableAnimations: true
  android:
    disableAnimations: true
```

```bash
# Use parallel execution
maestro test --shards 4 --shard-split flows/

# Use watch mode for development (auto-rerun on save)
maestro test --continuous Flow.yaml
```

---

## Flaky Tests

### Diagnostic Checklist

- [ ] Is the element actually present when the test runs?
- [ ] Is there a loading state or animation that could interfere?
- [ ] Does the test depend on network conditions?
- [ ] Is there a race condition with background processes?
- [ ] Does the test have sufficient retry/wait logic?
- [ ] Is the selector stable enough (using `id` vs. `text`)?

### Stabilization Strategies

```yaml
# 1. Use stable selectors (id > text > point)
- tapOn:
    id: "stable_button"       # Best
- tapOn: "Submit"             # OK (but text may change)
- tapOn: "50%,50%"            # Worst (breaks on screen size change)

# 2. Wait for loading states
- extendedWaitUntil:
    notVisible: "Loading..."
    timeout: 15000

# 3. Handle optional elements (banners, popups)
- tapOn:
    id: "cookie_banner_close"
    optional: true

# 4. Add retry for known-flaky interactions
- retry:
    maxRetries: 2
    commands:
      - tapOn: "Refresh"
      - assertVisible: "Updated Content"

# 5. Use proper scroll before tap
- scrollUntilVisible:
    element:
      id: "bottom_button"
    direction: DOWN
    timeout: 10000
```

---

## Platform-Specific Issues

### iOS

| Issue | Solution |
|-------|----------|
| Physical iOS devices not supported locally | Use Maestro Cloud for physical iOS devices |
| macOS Sequoia + Xcode 16.2 connection hangs | Update Maestro CLI to latest version |
| Elements not found on iOS | Set `accessible={false}` on parent `<View>` components |
| Keychain persists | Use `clearKeychain` command |
| WebView element IDs not recognized | Use text-based selectors or JavaScript evaluation |
| `hideKeyboard` unreliable | Use `optional: true` or tap on neutral area |
| Multi-app journeys | Use `id: "breadcrumb"` for system navigation |

### Android

| Issue | Solution |
|-------|----------|
| Emulator API 29-34 supported | Use compatible emulator images |
| Physical device connection | Ensure ADB is enabled and authorized |
| `inputText` Unicode not supported | Use ASCII characters only |
| Back button | Use `pressKey: Back` or `- back` command |
| App link disambiguation dialog | Use `autoVerify: true` in `openLink` |
| Slow emulator startup | Use `--start-device` flag or pre-start emulator |

### React Native

| Issue | Solution |
|-------|----------|
| Elements not found | Add `testID` prop to all testable elements |
| iOS nesting | Set `accessible={false}` on parent containers |
| Expo support | Use `openLink: exp://` for Expo Go |
| Component tree depth | Use `containsDescendants` for deep nesting |

### Flutter

| Issue | Solution |
|-------|----------|
| Elements not found | Use `Semantics(identifier:)` not Flutter Keys |
| WebView testing | Call `ensureSemantics()` in app init |
| Desktop not supported | Use mobile emulators/simulators only |

---

## Known Limitations

1. **iOS Physical Devices**: Not supported locally. Use Maestro Cloud.
2. **WebView Content**: Limited support. Maestro sees native layer, not WebView DOM. Use text selectors or JavaScript.
3. **Unicode on Android**: `inputText` does not support Unicode on Android.
4. **`optional` on `scrollUntilVisible`**: Not supported. Wrap in `runFlow` with `optional`.
5. **Subflow `env` inheritance**: Subflows do NOT inherit parent `env`. Pass explicitly.
6. **Parameter interpolation in `runFlow.file`**: Does not work in file paths.
7. **Network mocking**: Built-in mocking deprecated. Use external tools (WireMock).
8. **Maestro Studio + CLI conflict**: Cannot run simultaneously.
9. **Log file growth** (v1.39.0 bug): DEBUG logs included by default.
10. **Android API 35/36**: Support may be pending. Check latest CLI version.

---

## Getting Help

### Official Resources

- **Documentation**: https://docs.maestro.dev
- **GitHub**: https://github.com/mobile-dev-inc/maestro
- **Slack Community**: https://slack.maestro.dev
- **Blog**: https://maestro.dev/blog

### Effective Bug Reports

When reporting issues, include:

1. Maestro CLI version: `maestro --version`
2. Platform and OS version (iOS/Android)
3. Minimal reproducible flow YAML
4. `maestro hierarchy` output
5. Screenshots or video recording
6. Debug logs (`--debug-output`)

```bash
# Get version
maestro --version

# Get debug info for bug report
maestro bugreport
```
