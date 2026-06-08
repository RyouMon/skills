# Maestro Best Practices & Design Patterns

Comprehensive guide for organizing, writing, and maintaining Maestro test suites.

## Table of Contents

1. [Test Organization Patterns](#test-organization-patterns)
2. [Naming Conventions](#naming-conventions)
3. [Page Object Model (POM)](#page-object-model-pom)
4. [Reusable Patterns](#reusable-patterns)
5. [Data-Driven Testing](#data-driven-testing)
6. [Anti-Patterns](#anti-patterns)
7. [React Native Specifics](#react-native-specifics)
8. [Flutter Specifics](#flutter-specifics)

---

## Test Organization Patterns

### Pattern A: User Journeys (Goal-Driven Apps)

Best for: E-commerce, fintech, food delivery (DoorDash, Uber, Banking)

Goal: Success = user reaches the end of a funnel (e.g., Order Confirmed)

```
flows/
├── config.yaml
├── journeys/
│   ├── new_user/
│   │   ├── register_and_purchase.yaml
│   │   └── onboarding_complete.yaml
│   └── returning_user/
│       ├── reorder_previous.yaml
│       └── update_payment.yaml
└── common/
    ├── login.yaml
    ├── setup.yaml
    └── teardown.yaml
```

### Pattern B: Feature Tests (Habit-Driven Apps)

Best for: Social media, messaging, content apps (Instagram, TikTok, Chat)

Goal: Each feature works independently (post, like, message, search)

```
flows/
├── config.yaml
├── auth/
│   ├── login.yaml
│   ├── logout.yaml
│   └── forgot_password.yaml
├── feed/
│   ├── scroll_feed.yaml
│   ├── like_post.yaml
│   └── create_post.yaml
├── profile/
│   ├── view_profile.yaml
│   ├── edit_profile.yaml
│   └── change_avatar.yaml
└── common/
    ├── setup.yaml
    └── login_subflow.yaml
```

### Pattern C: Screen-Based Organization

Best for: Apps with clear screen-by-screen navigation

```
flows/
├── config.yaml
├── screens/
│   ├── splash_screen.yaml
│   ├── login_screen.yaml
│   ├── home_screen/
│   │   ├── tab_discovery.yaml
│   │   └── tab_profile.yaml
│   └── checkout_screen/
│       ├── shipping.yaml
│       └── payment.yaml
└── common/
    └── navigation.yaml
```

---

## Naming Conventions

### Flow Files

```
# Descriptive, snake_case, action-focused
login_with_email.yaml
complete_checkout_guest.yaml
add_item_to_cart.yaml
deep_link_from_notification.yaml

# BAD: vague, too short
test1.yaml
flow.yaml
new.yaml
```

### Subflows (Reusable Components)

```
# Prefix with action: do_*, verify_*, setup_*
do_login.yaml
verify_home_screen_loaded.yaml
setup_clean_state.yaml

# Or prefix with underscore for "private" subflows
_login.yaml
_setup.yaml
```

### Tags

Use consistent tag categories:

```yaml
# Priority tags
tags: [critical, smoke, regression]

# Feature tags
tags: [auth, checkout, profile, search]

# Execution tags
tags: [slow, fast, parallel, serial]

# Status tags
tags: [stable, flaky, wip, skip]

# Platform tags
tags: [ios, android, cross_platform]
```

Tag utility/subflows with `skip` to prevent standalone execution:

```yaml
# common/login.yaml — only meant to be run via runFlow
appId: com.example.app
tags:
  - skip
---
- tapOn: Email
- inputText: ${EMAIL}
```

---

## Page Object Model (POM)

Centralize selectors in JavaScript files. When the UI changes, update selectors in one place.

### Basic POM

```javascript
// scripts/selectors.js
output.selectors = {
    auth: {
        emailField:     'email_input',
        passwordField:  'password_input',
        loginButton:    'login_button',
        errorMessage:   'login_error',
        welcomeHeader:  'welcome_text'
    },
    checkout: {
        cartIcon:       'cart_icon',
        checkoutButton: 'checkout_btn',
        cardNumber:     'card_number_input',
        payButton:      'pay_now_btn',
        successMessage: 'order_confirmed'
    }
}
```

```yaml
# flows/auth/login.yaml
appId: com.example.app
---
- runScript: scripts/selectors.js
- tapOn:
    id: ${output.selectors.auth.emailField}
- inputText: ${EMAIL}
- tapOn:
    id: ${output.selectors.auth.passwordField}
- inputText: ${PASSWORD}
- tapOn:
    id: ${output.selectors.auth.loginButton}
- assertVisible:
    id: ${output.selectors.auth.welcomeHeader}
```

### Cross-Platform POM

```javascript
// scripts/selectors.js
if (maestro.platform === 'ios') {
    output.selectors = {
        loginButton: 'login_button_ios',
        menuIcon:    'menu_icon_ios'
    }
} else {
    output.selectors = {
        loginButton: 'login_button_android',
        menuIcon:    'menu_icon_android'
    }
}
```

### Platform-Specific Selector Loading

```yaml
# In flow file
- runScript: scripts/selectors.js

# Platform-specific element
- tapOn:
    id: ${output.selectors.loginButton}
```

---

## Reusable Patterns

### Login Reuse Pattern

```yaml
# common/login_subflow.yaml
appId: com.example.app
tags:
  - skip
---
- tapOn:
    id: "email_field"
- inputText: ${EMAIL}
- tapOn:
    id: "password_field"
- inputText: ${PASSWORD}
- tapOn:
    id: "login_button"
- assertVisible:
    id: "home_screen"
```

```yaml
# flows/checkout/purchase.yaml
appId: com.example.app
---
- launchApp
- runFlow:
    file: common/login_subflow.yaml
    env:
      EMAIL: ${TEST_USER_EMAIL}
      PASSWORD: ${TEST_USER_PASSWORD}
- tapOn:
    id: "add_to_cart"
# ... rest of checkout flow
```

### Wait and Retry Pattern

```yaml
# For flaky elements
- retry:
    maxRetries: 3
    commands:
      - tapOn:
          id: "unstable_network_button"
      - assertVisible: "Success Message"

# For slow-loading content
- extendedWaitUntil:
    visible:
      id: "dynamic_content"
    timeout: 30000
```

### Scroll and Find Pattern

```yaml
# Scroll until element is visible, then tap
- scrollUntilVisible:
    element:
      id: "target_item"
    direction: DOWN
    timeout: 15000
- tapOn:
    id: "target_item"
```

### Swipe Pattern for Carousels

```yaml
# Swipe carousel left 3 times
- repeat:
    times: 3
    commands:
      - swipe:
          from:
            id: "carousel"
          direction: LEFT
```

### Conditional Popup/Ad Handling

```yaml
# Dismiss optional popup if present
- tapOn:
    id: "cookie_accept"
    optional: true

- tapOn:
    id: "promo_close"
    optional: true

- tapOn:
    id: "rate_app_later"
    optional: true
```

---

## Data-Driven Testing

### Using JavaScript for Dynamic Data

```javascript
// scripts/faker.js — Generate test data
const emails = [
    'test1@example.com',
    'test2@example.com',
    'test3@example.com'
]
output.testData = {
    email: emails[Math.floor(Math.random() * emails.length)],
    timestamp: Date.now(),
    randomId: Math.random().toString(36).substring(7)
}
```

```yaml
- runScript: scripts/faker.js
- tapOn: Email
- inputText: ${output.testData.email}
```

### Using HTTP for API-Driven Tests

```javascript
// scripts/api.js
const response = http.get('https://api.example.com/test-users')
const users = response.json()
output.testUser = users[0]
```

```yaml
- runScript: scripts/api.js
- tapOn: Email
- inputText: ${output.testUser.email}
```

---

## Anti-Patterns

### 1. Coordinate-Based Tapping (Overusing `point`)

```yaml
# BAD — Breaks on different screen sizes
- tapOn: "50%,50%"

# GOOD — Use stable selectors
- tapOn:
    id: "submit_button"
```

### 2. Hard-Coded Sleeps

```yaml
# BAD — Wastes time, still may fail
- launchApp
- sleep: 5000                                       # Don't do this!

# GOOD — Maestro auto-waits, or use extendedWaitUntil
- launchApp
- extendedWaitUntil:
    visible: "Home Screen"
    timeout: 10000
```

### 3. Monolithic Flows

```yaml
# BAD — 500 lines in one file
# GOOD — Break into subflows
- runFlow: login.yaml
- runFlow: navigate_to_cart.yaml
- runFlow: checkout.yaml
```

### 4. Duplicate Setup Code

```yaml
# BAD — Copy-paste login in every flow
# GOOD — Use onFlowStart hooks or runFlow
```

### 5. Unnecessary Platform Duplication

```yaml
# BAD — Two separate files for iOS/Android
# GOOD — Single flow with platform conditionals
- tapOn:
    id: "submit"
    when:
      platform: ios
- tapOn:
    id: "submit_android"
    when:
      platform: android
```

### 6. Missing `optional` on One-Time Elements

```yaml
# BAD — Tutorial only shows on first run, fails on reruns
- tapOn: "Skip Tutorial"

# GOOD — Mark as optional
- tapOn:
    text: "Skip Tutorial"
    optional: true
```

---

## React Native Specifics

### Setting Up testID

```jsx
// React Native component
function LoginScreen() {
    return (
        <View>
            <TextInput
                testID="email_input"
                accessibilityLabel="Email Address"
                placeholder="Enter email"
            />
            <TextInput
                testID="password_input"
                secureTextEntry
                placeholder="Enter password"
            />
            <Button
                testID="login_button"
                title="Login"
                onPress={handleLogin}
            />
        </View>
    )
}
```

### iOS Nesting Fix

If elements aren't found on iOS, set `accessible={false}` on parent containers:

```jsx
// BAD — Parent accessible=true swallows children on iOS
<View accessible={true}>
    <Text>Title</Text>
    <Button title="Tap" />
</View>

// GOOD — Only leaf nodes are accessible
<View accessible={false}>
    <Text accessible={true}>Title</Text>
    <Button title="Tap" accessible={true} />
</View>
```

### Expo Support

Maestro works with both Expo Go and development builds:

```yaml
# For Expo Go
- openLink: exp://127.0.0.1:8081

# For EAS development build
- launchApp: com.yourapp.dev
```

---

## Flutter Specifics

### Setting Up Accessibility Identifiers

```dart
// Flutter — Use Semantics widget (recommended since Flutter 3.19+)
Semantics(
    identifier: 'login_button',
    child: ElevatedButton(
        onPressed: () {},
        child: Text('Login'),
    ),
)
```

```dart
// Alternative — Use keyName (less reliable)
ElevatedButton(
    key: Key('login_button'),
    onPressed: () {},
    child: Text('Login'),
)
```

> **Important**: Never use Flutter Keys directly (`ValueKey`, `GlobalKey`). They are NOT exposed to the accessibility layer. Use `Semantics(identifier:)` instead.

### Flutter Web

Requires calling `ensureSemantics()` before running tests:

```dart
// In your app initialization
RendererBinding.instance.ensureSemantics();
```

---

## Environment Variable Best Practices

```bash
# .env file (gitignored)
MAESTRO_APP_ID=com.example.app
MAESTRO_BASE_URL=https://staging.example.com
MAESTRO_TIMEOUT=10000
MAESTRO_TEST_USER_EMAIL=test@example.com
MAESTRO_TEST_USER_PASSWORD=TestPass123
```

Maestro auto-reads environment variables with `MAESTRO_` prefix (stripping the prefix):

```yaml
# In flow — reference without MAESTRO_ prefix
- inputText: ${APP_ID}
- inputText: ${TEST_USER_EMAIL}
```

CLI override with `-e` flag:

```bash
maestro test -e ENV=staging -e TIMEOUT=20000 flows/
```

## CI/CD Tag Strategy

```yaml
# config.yaml
includeTags:
  - smoke
excludeTags:
  - skip
  - wip
  - slow

# PR checks — fast feedback
# CLI: maestro test --include-tags smoke flows/

# Nightly build — full regression
# CLI: maestro test --include-tags regression --exclude-tags skip flows/

# Feature branch — relevant tests
# CLI: maestro test --include-tags checkout flows/
```
