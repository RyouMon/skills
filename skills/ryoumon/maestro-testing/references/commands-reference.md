# Maestro Commands & Selectors Complete Reference

Complete reference for all Maestro YAML commands, selectors, and parameters. Based on docs.maestro.dev and community sources.

## Table of Contents

1. [App Lifecycle Commands](#app-lifecycle-commands)
2. [Touch & Gesture Commands](#touch--gesture-commands)
3. [Text Input Commands](#text-input-commands)
4. [Assertion Commands](#assertion-commands)
5. [Scroll Commands](#scroll-commands)
6. [Flow Control Commands](#flow-control-commands)
7. [Device Control Commands](#device-control-commands)
8. [AI-Powered Commands](#ai-powered-commands)
9. [Selector System](#selector-system)
10. [Command Parameters](#command-parameters)
11. [Conditions](#conditions)

---

## App Lifecycle Commands

### `launchApp`
Launch the app under test or another app by bundle ID / package name.

```yaml
- launchApp                                         # Launch app under test
- launchApp: com.example.otherapp                  # Launch different app
- launchApp:
    clearState: true                                # Clear app data before launch
    clearKeychain: true                             # Clear iOS Keychain (iOS only)
    stopApp: false                                  # Bring to foreground without restart
    permissions:
      all: allow                                    # Grant all permissions
      camera: deny                                  # Deny specific permission
      location: unset                               # Leave unset
    arguments:
      userId: "12345"                               # Launch arguments (string/bool/number)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `appId` | string | Bundle ID (iOS) or package name (Android). Defaults to flow's `appId` |
| `clearState` | boolean | Clear app preferences, databases, accounts |
| `clearKeychain` | boolean | Clear entire iOS Keychain (iOS only) |
| `stopApp` | boolean | `false` = bring backgrounded app to foreground. Default `true` |
| `permissions` | map | Permission overrides: `all`, `camera`, `location`, `notifications`, `photos`, `microphone`, `contacts`, `calendar`, `reminders`, `bluetooth`, `health`, `motion`, `mediaLibrary`, `homeKit`, `siri`, `speech`, `appTrackingTransparency`. Values: `allow`, `deny`, `unset` |
| `arguments` | map | Key-value launch arguments passed to app |

**Permission values:** `allow` (grant), `deny` (deny), `unset` (system prompt).

---

### `stopApp`
Stop the currently running app.

```yaml
- stopApp
- stopApp: com.example.app                         # Stop specific app
```

---

### `killApp`
Force kill an application.

```yaml
- killApp
- killApp: com.example.app
```

---

### `clearState`
Clear app state including preferences, databases, and accounts.

```yaml
- clearState
```

> Note: Does NOT clear iOS Keychain. Use `clearKeychain` for that.

---

### `clearKeychain`
Clear the entire iOS Keychain (iOS only).

```yaml
- clearKeychain
```

---

### `openLink`
Open a URL or deep link.

```yaml
- openLink: https://example.com
- openLink:
    link: awesomeapp://settings
    autoVerify: true                                # Android only: skip app picker
    browser: true                                   # Android only: force Chrome
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `link` | string | Required. URL or custom scheme to open |
| `autoVerify` | boolean | Android only. Auto-verify app for web links |
| `browser` | boolean | Android only. Force open in Google Chrome |

---

### `back`
Android system back button.

```yaml
- back
```

---

## Touch & Gesture Commands

### `tapOn`
Tap on a UI element.

```yaml
- tapOn: "Login"                                    # Shorthand text selector
- tapOn:
    id: "submit_button"
    enabled: true
    optional: true                                  # Skip if not found
    retryTapIfNoChange: true
    waitToSettleTimeoutMs: 5000
- tapOn:
    text: "Buy Now"
    index: 2                                        # Third matching element
- tapOn: "50%,50%"                                  # Relative coordinate
- tapOn:
    point: "100,200"                                # Absolute pixel coordinate
    repeat: 3
    delay: 500
```

| Parameter | Type | Description |
|-----------|------|-------------|
| selector | string/map | Element to tap (text, id, point, etc.) |
| `point` | string | `"X%,Y%"` (relative) or `"X,Y"` (absolute pixels) |
| `repeat` | integer | Number of repeated taps |
| `delay` | integer | Delay in ms between repeated taps. Default `100` |
| `retryTapIfNoChange` | boolean | Retry tap if UI doesn't change |
| `waitToSettleTimeoutMs` | integer | Max ms to wait for screen to settle |
| `optional` | boolean | Skip command if element not found |
| `longPress` | boolean | If `true`, acts as long press |

---

### `doubleTapOn`
Double tap on an element.

```yaml
- doubleTapOn: "Image"
- doubleTapOn:
    id: "zoom_image"
```

---

### `longPressOn`
Long press on an element.

```yaml
- longPressOn: "List Item"
- longPressOn:
    id: "map_pin"
    duration: 2000                                  # Press duration in ms
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `duration` | integer | Press duration in milliseconds |

---

### `swipe`
Swipe gesture in any direction.

```yaml
- swipe:
    direction: LEFT                                 # LEFT, RIGHT, UP, DOWN
- swipe:
    start: "90%,50%"
    end: "10%,50%"
    duration: 600
- swipe:
    from:
      id: "carousel"
    direction: LEFT
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `direction` | string | `LEFT`, `RIGHT`, `UP`, `DOWN`. Not with `start`/`end` |
| `start` | string | Starting coordinate: `"X%,Y%"` or `"X,Y"` |
| `end` | string | Ending coordinate |
| `from` | map | Element selector as swipe starting point |
| `duration` | integer | Swipe duration in ms. Default `400` |
| `waitToSettleTimeoutMs` | integer | Max ms to wait for screen to settle |

**Direction defaults:**
| Direction | Start | End |
|-----------|-------|-----|
| LEFT | 90%,50% | 10%,50% |
| RIGHT | 10%,50% | 90%,50% |
| UP | 50%,90% | 50%,10% |
| DOWN | 50%,10% | 50%,90% |

---

## Text Input Commands

### `inputText`
Type text into the focused field or a specified field.

```yaml
- inputText: "hello@example.com"
- inputText:
    text: "Password123"
    id: "password_field"
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `text` | string | Text to input. Required if using map syntax |

> **Note**: Unicode not supported on Android for `inputText`. Use ASCII-only on Android.

---

### `eraseText`
Erase text from the focused field.

```yaml
- eraseText                                         # Erase all
- eraseText: 5                                      # Erase last 5 characters
```

| Parameter | Type | Description |
|-----------|------|-------------|
| (integer) | integer | Number of characters to erase. Omit to erase all |

---

### `hideKeyboard`
Hide the on-screen keyboard.

```yaml
- hideKeyboard
- hideKeyboard: "Done"                              # iOS: tap "Done" button
```

> Known limitation: Unreliable on iOS. Use `optional: true` to prevent failures.

---

### `pasteText`
Paste clipboard content into focused field.

```yaml
- pasteText
```

---

### Random Data Generation Commands

```yaml
- inputRandomEmail                                  # Generates: user@domain.com
- inputRandomPersonName                             # First Last
- inputRandomNumber                                 # Random number
- inputRandomText                                   # Random lorem ipsum text
- inputRandomEmail:
    label: "Email"                                  # Optional custom label
```

---

## Assertion Commands

### `assertVisible`
Assert an element is visible on screen. Auto-retries for 7 seconds.

```yaml
- assertVisible: "Welcome"                          # Shorthand text
- assertVisible:
    id: "success_icon"
    enabled: true
    timeout: 15000                                  # Custom timeout in ms
- assertVisible:
    text: "Submit"
    index: 0
```

| Parameter | Type | Description |
|-----------|------|-------------|
| selector | string/map | Element to check |
| `timeout` | integer | Timeout in ms. Default `7000` |

---

### `assertNotVisible`
Assert an element is NOT visible.

```yaml
- assertNotVisible: "Loading..."
- assertNotVisible:
    id: "error_message"
    timeout: 5000
```

---

### `assertTrue`
Assert a JavaScript expression is truthy.

```yaml
- assertTrue: ${output.userCount > 0}
- assertTrue:
    condition: ${output.status === "active"}
    label: "User status should be active"             # Error message on failure
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `condition` | string/JS | JavaScript expression to evaluate |
| `label` | string | Custom error message on failure |

---

### `extendedWaitUntil`
Wait until a condition is met with extended timeout.

```yaml
- extendedWaitUntil:
    visible: "Loaded Content"
    timeout: 30000
- extendedWaitUntil:
    notVisible: "Loading Spinner"
    timeout: 15000
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `visible` | string/map | Wait until element is visible |
| `notVisible` | string/map | Wait until element is not visible |
| `timeout` | integer | Timeout in ms |

---

### `waitForAnimationToEnd`
Wait for animations to complete.

```yaml
- waitForAnimationToEnd                             # Default timeout
- waitForAnimationToEnd:
    timeout: 5000
```

---

### `assertScreenshot`
Compare current screen against a baseline screenshot (visual regression).

```yaml
- assertScreenshot                                  # Compare full screen
- assertScreenshot:
    label: "home_screen"
    tolerance: 0.01                                 # Tolerance threshold (0-1)
```

> Available since CLI 2.2.0+. Baseline screenshots stored in `.maestro/screenshots/`.

---

## Scroll Commands

### `scroll`
Scroll in a direction.

```yaml
- scroll
- scroll:
    direction: DOWN                                 # UP, DOWN, LEFT, RIGHT
```

---

### `scrollUntilVisible`
Scroll until a specific element becomes visible.

```yaml
- scrollUntilVisible:
    element:
      text: "Footer Item"
    direction: DOWN
    timeout: 10000
    speed: 40                                       # Scroll speed (default 40)
    visibilityPercentage: 100                       # Required visibility % (0-100)
    centerElement: true                             # Center element after visible
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `element` | map | Element selector to find |
| `direction` | string | `UP`, `DOWN`, `LEFT`, `RIGHT`. Default `DOWN` |
| `timeout` | integer | Max time to scroll. Default `10000` |
| `speed` | integer | Scroll speed. Default `40` |
| `visibilityPercentage` | integer | Required visibility percentage. Default `100` |
| `centerElement` | boolean | Center element after it becomes visible |

> **Note**: `optional: true` does NOT work on `scrollUntilVisible`. Wrap in `runFlow` with `optional` if needed.

---

## Flow Control Commands

### `runFlow`
Execute another flow file or inline commands.

```yaml
# Run a file
- runFlow: login.yaml

# Run with environment variables
- runFlow:
    file: login.yaml
    env:
      EMAIL: "test@example.com"
      PASSWORD: "secret"

# Inline subflow with label
- runFlow:
    label: "Sort alphabetically"
    commands:
      - tapOn:
          id: sort_icon
      - tapOn: A-Z

# Inline with env
- runFlow:
    env:
      PARAM: "value"
    commands:
      - tapOn: ${PARAM}
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `file` | string | Path to flow file (relative) |
| `label` | string | Description shown in reports |
| `env` | map | Key-value environment variables |
| `commands` | list | Inline commands (alternative to `file`) |

> **Critical**: Subflows do NOT inherit parent's `env`. Always pass explicitly.

---

### `repeat`
Repeat a block of commands.

```yaml
# Repeat N times
- repeat:
    times: 5
    commands:
      - swipe:
          direction: UP

# Repeat while condition
- repeat:
    while:
      notVisible: "End of List"
    commands:
      - scroll

# Combined (safe loop)
- repeat:
    times: 10
    while:
      notVisible: "Target"
    commands:
      - swipe:
          direction: DOWN
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `times` | integer | Max repetitions |
| `while` | map | Condition: `visible`, `notVisible`, `true` (JS) |
| `commands` | list | Commands to repeat |

---

### `retry`
Retry a command or block on failure.

```yaml
- retry:
    maxRetries: 3                                   # 0-3 retries
    commands:
      - tapOn:
          id: "unstable_button"
      - assertVisible: "Success"
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `maxRetries` | integer | 0 to 3. Default `1` |
| `commands` | list | Commands to retry |

---

## Device Control Commands

### `pressKey`
Press a hardware/system key.

```yaml
- pressKey: Enter
- pressKey: Back                                    # Android only
- pressKey: Home
- pressKey: VolumeUp
- pressKey: VolumeDown
- pressKey: Power
- pressKey: Search
- pressKey: Menu                                    # Android only
- pressKey: Tab
- pressKey: Escape
- pressKey: Space
- pressKey: DpadUp                                  # Android TV
- pressKey: DpadDown
- pressKey: DpadLeft
- pressKey: DpadRight
- pressKey: DpadCenter
```

**Supported keys:** `Enter`, `Back`, `Home`, `VolumeUp`, `VolumeDown`, `VolumeMute`, `Power`, `Search`, `Menu`, `Tab`, `Escape`, `Space`, `Delete`, `DpadUp`, `DpadDown`, `DpadLeft`, `DpadRight`, `DpadCenter`, `BrightnessUp`, `BrightnessDown`.

---

### `setLocation`
Set GPS location.

```yaml
- setLocation:
    latitude: 40.7128
    longitude: -74.0060
```

---

### `setAirplaneMode`
Toggle airplane mode.

```yaml
- setAirplaneMode: true
- setAirplaneMode: false
```

---

### `setOrientation`
Set screen orientation.

```yaml
- setOrientation: LANDSCAPE
- setOrientation: PORTRAIT
```

---

### `setPermissions`
Override app permissions.

```yaml
- setPermissions:
    all: allow
    camera: deny
    location: unset
```

---

### `setClipboard`
Set clipboard content.

```yaml
- setClipboard: "Text to copy"
```

---

### `takeScreenshot`
Capture a screenshot.

```yaml
- takeScreenshot: checkout_page                     # Named screenshot
- takeScreenshot                                   # Auto-named
- takeScreenshot:
    label: "payment_screen"
    optional: true
```

Screenshots saved to `~/.maestro/tests/` by default.

---

### `startRecording` / `stopRecording`
Record test execution as video.

```yaml
- startRecording: my_test
# ... test commands ...
- stopRecording
```

> Max 2 minutes. Saved to `~/.maestro/tests/`. Use `--local` flag for local rendering.

---

### `addMedia`
Add media to device gallery.

```yaml
- addMedia:
    - "path/to/photo1.jpg"
    - "path/to/photo2.png"
```

---

### `travel`
Simulate travel between GPS coordinates.

```yaml
- travel:
    points:
      - latitude: 40.7128
        longitude: -74.0060
      - latitude: 40.7580
        longitude: -73.9855
    speed: 50                                         # km/h
```

---

### `reboot`
Reboot the device (Android only).

```yaml
- reboot
```

---

### `mockNetwork`
Mock network conditions.

```yaml
- mockNetwork:
    url: https://api.example.com/data
    headers:
      Content-Type: application/json
    body: |
      { "status": "ok", "data": [] }
    statusCode: 200
    method: GET
```

---

## AI-Powered Commands

### `assertWithAI`
Use AI to assert a condition described in natural language.

```yaml
- assertWithAI: "The login form is visible with email and password fields"
```

### `assertNoDefectsWithAI`
Use AI to scan for visual defects.

```yaml
- assertNoDefectsWithAI: "No crashes, error messages, or visual glitches"
```

### `extractTextWithAI`
Extract text from the screen using AI.

```yaml
- extractTextWithAI: "Extract all visible prices on the screen"
```

---

## Text Extraction Commands

### `copyTextFrom`
Copy text from an element into `maestro.copiedText`.

```yaml
- copyTextFrom:
    id: "price_label"
# Later access via JavaScript:
- evalScript: ${output.price = maestro.copiedText}
```

---

## Selector System

### Core Selectors

| Selector | Type | Example | Notes |
|----------|------|---------|-------|
| `text` | string | `text: "Submit"` | **Regex by default!** Escape special chars |
| `id` | string | `id: "login_btn"` | Most stable. Maps to testID/resource-id/accessibilityIdentifier |
| `index` | integer | `index: 2` | Zero-based. Third matching element |
| `point` | string | `point: "50%,50%"` | Relative % or absolute pixels. Avoid when possible |
| `css` | string | `css: "button.primary"` | Web testing only |

### Relational Selectors

```yaml
- tapOn:
    text: "Submit"
    below: "Form Header"                            # Element below "Form Header"
- tapOn:
    id: "settings"
    above: "Logout"                                 # Element above "Logout"
- tapOn:
    text: "Item"
    leftOf: "Price"                                 # Element to the left
- tapOn:
    text: "Item"
    rightOf: "Checkbox"                             # Element to the right
- tapOn:
    containsChild: "Label"                          # Parent containing child
- tapOn:
    childOf:
      id: "list_container"                          # Direct child of parent
- tapOn:
    containsDescendants: "Text"                     # Ancestor at any depth
```

### State Selectors

```yaml
- tapOn:
    text: "Submit"
    enabled: true
- tapOn:
    id: "agree_checkbox"
    checked: true
- tapOn:
    id: "search_field"
    focused: true
- tapOn:
    id: "option_a"
    selected: true
```

### Element Trait Selectors (iOS)

```yaml
- tapOn:
    traits: button                                  # iOS accessibility traits
```

### Dimension Matchers

```yaml
- tapOn:
    width: 200
    height: 50
```

### Combining Selectors

Selectors can be combined for precise targeting:

```yaml
- tapOn:
    text: "Buy"
    enabled: true
    below: "Product Name"
    index: 0
```

---

## Command Parameters

These parameters can be applied to most commands:

| Parameter | Type | Description |
|-----------|------|-------------|
| `optional` | boolean | Skip command if element not found. Default `false` |
| `label` | string | Custom description in test reports |
| `timeout` | integer | Override default timeout (ms) |
| `when` | map | Conditional execution (see Conditions below) |

---

## Conditions

### `when` Block

Execute a command only if a condition is met:

```yaml
- tapOn:
    id: "accept_cookies"
    when:
      visible: "Accept Cookies"                     # Only tap if visible

- tapOn:
    id: "ios_only_button"
    when:
      platform: ios                                 # Only on iOS

- tapOn:
    id: "android_only"
    when:
      platform: android

- runFlow:
    file: premium_flow.yaml
    when:
      true: ${output.isPremium === true}            # JavaScript condition

- tapOn:
    id: "banner_close"
    when:
      notVisible: "Main Content"                    # Only if NOT visible (inverted)
```

| Condition | Description |
|-----------|-------------|
| `visible` | Execute if element is visible |
| `notVisible` | Execute if element is NOT visible |
| `platform: ios` | Execute only on iOS |
| `platform: android` | Execute only on Android |
| `true` | Execute if JS expression is truthy |

**Multiple conditions in a single `when` use AND logic.**

### Flow-Level Conditions

```yaml
appId: com.example.app
onFlowStart:
  - runFlow: setup.yaml                             # Run before every flow
onFlowComplete:
  - runFlow: teardown.yaml                          # Run after every flow
```

### Flow Configuration

```yaml
appId: com.example.app
tags:
  - smoke
  - critical
properties:
  jiraIssue: PROJ-123                               # Custom JUnit metadata
---
- launchApp
```
