# Maestro Platform-Specific Guide

Detailed guide for iOS, Android, React Native, and Flutter integration with Maestro.

## Table of Contents

1. [iOS](#ios)
2. [Android](#android)
3. [React Native](#react-native)
4. [Flutter](#flutter)
5. [Cross-Platform Testing](#cross-platform-testing)
6. [Other Frameworks](#other-frameworks)
7. [WebView Testing](#webview-testing)

---

## iOS

### Setup Requirements

- macOS (required — iOS testing only works on macOS)
- Latest Xcode + Xcode Command Line Tools
- Java 17+
- iOS Simulator (physical devices NOT supported locally, use Maestro Cloud)

```bash
# Install Xcode Command Line Tools
xcode-select --install

# Accept license
sudo xcodebuild -license accept

# Verify setup
xcrun simctl list devices
```

### Selectors for iOS

| Maestro Selector | iOS Property |
|-----------------|-------------|
| `id` | `accessibilityIdentifier` |
| `text` | `accessibilityLabel` |
| `traits` | `accessibilityTraits` |

```swift
// SwiftUI
Button("Login") {
    // action
}
.accessibilityIdentifier("login_button")

// UIKit
let button = UIButton()
button.accessibilityIdentifier = "login_button"
button.accessibilityLabel = "Login"
```

### iOS-Specific Commands

```yaml
# Clear keychain (iOS only)
- clearKeychain

# Handle permission dialogs
- launchApp:
    permissions:
      all: allow
      notifications: unset

# Swipe-to-go-back gesture
- swipe:
    direction: RIGHT

# Multi-app journeys via system breadcrumbs
- tapOn:
    id: "breadcrumb"
```

### iOS Simulator Tips

```bash
# List available simulators
xcrun simctl list devices

# Boot a simulator
xcrun simctl boot "iPhone 15"

# Install app
xcrun simctl install booted MyApp.app

# Run tests
maestro test flows/
```

### Common iOS Issues

| Issue | Solution |
|-------|----------|
| macOS Sequoia + Xcode 16.2 hang | Update to latest Maestro CLI |
| Elements not found | Set `accessible={false}` on parent containers (React Native) |
| WebView IDs not recognized | Use text-based selectors |
| Keychain persists | Use `clearKeychain` command |

---

## Android

### Setup Requirements

- Android Studio + Android SDK
- ADB (Android Debug Bridge)
- Java 17+
- Emulator (API 29-34) or physical device

```bash
# Verify ADB
adb devices

# Start emulator
emulator -avd Pixel_7_API_33

# Or use Maestro's start-device
maestro start-device
```

### Selectors for Android

| Maestro Selector | Android Property |
|-----------------|-----------------|
| `id` | `resource-id` (e.g., `com.app:id/button`) |
| `text` | `text` attribute |
| `contentDescription` | Handled via text matching |

```kotlin
// Kotlin/XML
<Button
    android:id="@+id/login_button"
    android:contentDescription="Login button"
    ... />

// Compose
Button(
    modifier = Modifier.testTag("login_button")
) {
    Text("Login")
}
```

### Android-Specific Commands

```yaml
# Hardware back button
- pressKey: Back
# Or shorthand:
- back

# Deep links with auto-verify
- openLink:
    link: https://example.com/product/123
    autoVerify: true

# Reboot device (Android only)
- reboot
```

### Physical Device Testing

1. Enable Developer Options on device
2. Enable USB Debugging
3. Authorize the computer
4. Verify with `adb devices`

```bash
# Should show device as "device" (not "unauthorized")
adb devices
# List of devices attached
# ABC123DEF    device
```

### Common Android Issues

| Issue | Solution |
|-------|----------|
| Unicode in `inputText` | Not supported — use ASCII only |
| Emulator slow on CI | Use `macos-latest` runner, enable KVM |
| App link disambiguation | Use `autoVerify: true` in `openLink` |
| Permission denied | Grant via `launchApp.permissions` |

---

## React Native

### Setup

**Zero library dependencies** — Maestro works out of the box with React Native.

### Adding testIDs

```jsx
// Basic testID
<TextInput
    testID="email_input"
    placeholder="Email"
/>

// Native components support testID directly
<Button
    testID="submit_button"
    title="Submit"
    onPress={handleSubmit}
/>
```

### iOS Accessibility Nesting Fix

On iOS, if a parent has `accessible={true}`, children become invisible to Maestro:

```jsx
// BAD — Children hidden from Maestro on iOS
<View accessible={true}>
    <Text>Title</Text>
    <Button title="Tap me" />
</View>

// GOOD — Only leaf nodes accessible
<View accessible={false}>
    <Text accessible={true}>Title</Text>
    <TouchableOpacity
        testID="tap_button"
        accessible={true}
    >
        <Text>Tap me</Text>
    </TouchableOpacity>
</View>
```

### Expo Support

```yaml
# For Expo Go development
- openLink: exp://127.0.0.1:8081

# For EAS development builds (recommended for testing)
- launchApp: com.yourcompany.yourapp
```

### React Native Testing Patterns

```yaml
# Example: Complete login flow for React Native app
appId: com.example.rnapp
---
- launchApp:
    clearState: true
    permissions: { all: allow }

# Handle optional onboarding
- tapOn:
    text: "Skip"
    optional: true

# Login screen
- tapOn:
    id: "email_input"
- inputText: "test@example.com"
- tapOn:
    id: "password_input"
- inputText: "password123"
- hideKeyboard

# Submit
- tapOn:
    id: "login_button"

# Verify
- assertVisible:
    id: "home_screen"
```

---

## Flutter

### Setup

**No pubspec.yaml changes needed.** Maestro works via the Semantics tree.

### Adding Identifiers

```dart
// RECOMMENDED (Flutter 3.19+): Use Semantics identifier
Semantics(
    identifier: 'login_button',
    child: ElevatedButton(
        onPressed: () {},
        child: Text('Login'),
    ),
)

// NOT RECOMMENDED: Flutter Keys are NOT exposed to accessibility
// NEVER do this for Maestro testing:
// ElevatedButton(
//     key: ValueKey('login_button'),  // Won't work!
//     ...
// )
```

### Semantics Setup

For Flutter Web testing, call `ensureSemantics()`:

```dart
// In main.dart or app initialization
import 'package:flutter/rendering.dart';

void main() {
    RendererBinding.instance.ensureSemantics();
    runApp(MyApp());
}
```

### Flutter Testing Pattern

```yaml
appId: com.example.flutterapp
---
- launchApp:
    clearState: true

- tapOn:
    id: "email_field"
- inputText: "user@example.com"

- tapOn:
    id: "password_field"
- inputText: "password123"

- tapOn:
    id: "login_button"

- assertVisible:
    id: "dashboard_screen"
```

### Flutter Desktop Limitations

- Flutter Desktop is **not supported** by Maestro
- Use mobile emulators/simulators instead
- Flutter Web requires `ensureSemantics()`

---

## Cross-Platform Testing

### Writing Platform-Agnostic Flows

```yaml
# Use environment variables for cross-platform app IDs
appId: ${APP_ID}
---
- launchApp:
    clearState: true

# Platform-conditional steps
- tapOn:
    id: "ios_specific_button"
    when:
      platform: ios

- tapOn:
    id: "android_specific_button"
    when:
      platform: android

# Platform-agnostic step
- tapOn:
    id: "submit_button"
```

### Platform Detection in JavaScript

```javascript
// scripts/platform.js
if (maestro.platform === 'ios') {
    output.submitButton = 'submit_ios'
} else {
    output.submitButton = 'submit_android'
}
```

```yaml
- runScript: scripts/platform.js
- tapOn:
    id: ${output.submitButton}
```

### Conditional Subflows

```yaml
# Parent flow
- runFlow:
    file: ios/setup.yaml
    when:
      platform: ios

- runFlow:
    file: android/setup.yaml
    when:
      platform: android

# Common steps for both
- runFlow: common/login.yaml
```

### Environment Variable Approach

```bash
# .env
MAESTRO_IOS_APP_ID=com.example.ios
MAESTRO_ANDROID_APP_ID=com.example.android
```

```yaml
# Use in flow
appId: ${APP_ID}  # Set via CLI -e APP_ID=com.example.ios
```

---

## Other Frameworks

### NativeScript, Ionic, Capacitor

Maestro works with any framework that exposes native accessibility attributes:

1. Set proper `testID` / `accessibilityIdentifier` / `resource-id`
2. Ensure accessibility labels are set
3. Test via standard Maestro selectors

### WebView Testing

Maestro handles the **native layer**, not the WebView DOM content. For WebView-heavy apps:

1. **Use text-based selectors** — Maestro can read visible text
2. **Use JavaScript evaluation** — Execute JS in the WebView context
3. **Consider hybrid approach** — Maestro for native shells, Appium/Selenium for WebView content
4. **Test IDs in HTML** — If you control the WebView content, add test attributes

```yaml
# WebView text selector
- tapOn: "Web Button Text"

# If accessible labels are set
- tapOn:
    id: "webview_submit"
```

> **Note**: For apps that are primarily WebView-based, Appium may be a better fit than Maestro.

---

## Framework Quick Reference

| Framework | Identifier Method | Notes |
|-----------|------------------|-------|
| React Native | `testID` prop | Set `accessible={false}` on parent Views for iOS |
| Flutter | `Semantics(identifier:)` | Never use Flutter Keys. Use `ensureSemantics()` for Web |
| SwiftUI | `.accessibilityIdentifier()` | Native iOS support |
| UIKit | `accessibilityIdentifier` | Native iOS support |
| Android XML | `android:id` | Use as `id` selector |
| Jetpack Compose | `Modifier.testTag()` | Native Android support |
| Ionic/Capacitor | Standard web `id` | Limited WebView support |
