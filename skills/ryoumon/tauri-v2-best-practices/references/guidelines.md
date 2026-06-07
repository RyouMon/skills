# Tauri Project AGENTS.md Programming Principles

> This document compiles core programming principles, design patterns, and best practices for the Tauri v2 framework. It serves as a decision-making reference for AI Agents when generating, reviewing, and refactoring code in Tauri projects.
> Based on official documentation, community practices, and production experience from 2024-2026, covering 12 dimensions: Security, Architecture, IPC, State Management, Error Handling, Performance, Testing, Build, Accessibility, and Internationalization.

---

## Table of Contents

- [1. Architecture Principles](#1-architecture-principles)
- [2. Security Principles](#2-security-principles)
- [3. Rust Backend Principles](#3-rust-backend-principles)
- [4. Frontend Integration Principles](#4-frontend-integration-principles)
- [5. IPC Communication Principles](#5-ipc-communication-principles)
- [6. State Management Principles](#6-state-management-principles)
- [7. Error Handling Principles](#7-error-handling-principles)
- [8. Async & Performance Principles](#8-async--performance-principles)
- [9. Testing Principles](#9-testing-principles)
- [10. Build & Deployment Principles](#10-build--deployment-principles)
- [11. Accessibility Principles](#11-accessibility-principles)
- [12. Internationalization Principles](#12-internationalization-principles)
- [13. Core Design Patterns Cheatsheet](#13-core-design-patterns-cheatsheet)
- [14. Anti-Patterns Checklist](#14-anti-patterns-checklist)

---

## 1. Architecture Principles

### 1.1 Directory Structure: lib.rs Carries Logic, main.rs Only Forwards

Tauri v2 requires `lib.rs` to contain all application logic (commands, state, plugin registration), while `main.rs` is only responsible for forwarding calls. This is a mandatory requirement for mobile builds.

```rust
// main.rs — minimal entry point
fn main() {
    my_app_lib::run();
}

// lib.rs — application core
#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![...])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### 1.2 Commands Modularized by Functional Domain

Split command handlers by domain into `commands/` submodules, uniformly exposed in `mod.rs`, and registered in `lib.rs`.

```
src-tauri/src/
├── main.rs              # forwarding only
├── lib.rs               # entry, Builder config, command registration
├── commands/            # command handlers
│   ├── mod.rs
│   ├── system.rs        # system commands
│   ├── file.rs          # file operations
│   └── config.rs        # configuration management
├── services/            # business logic layer (optional)
├── models/              # data models
└── error.rs             # global error types
```

### 1.3 Layered Architecture: Handler → Service → Repository

The command handler layer should remain thin — responsible for parameter parsing, permission checking, and response wrapping; business logic sinks to the Service layer; data access is isolated to the Repository layer.

```rust
// commands/file.rs — thin Handler layer
#[tauri::command]
pub async fn read_file(
    path: String,
    state: State<'_, AppState>,
) -> Result<String, AppError> {
    let sanitized = validate_path(&path)?;
    state.file_service.read(&sanitized).await   // business logic in Service
}
```

### 1.4 Plugins as Modules: Functionality Encapsulated via Plugin System

Each independent functional unit should be encapsulated as a plugin, registered using the Builder pattern, with its own command namespace.

```rust
// custom plugin registration
Builder::default()
    .plugin(tauri_plugin_store::Builder::new().build())
    .plugin(my_feature_plugin::init())
```

---

## 2. Security Principles

### 2.1 Default Deny: Principle of Least Privilege

Tauri v2 denies all permissions by default. Each window's Capability must be explicitly declared in `capabilities/*.json`.

```json
// capabilities/main.json
{
  "$schema": "../gen/schemas/desktop-schema.json",
  "identifier": "main-capability",
  "windows": ["main"],
  "permissions": [
    "core:default",
    {
      "identifier": "fs:allow-read-text-file",
      "allow": [{ "path": "$APPDATA/**" }]
    }
  ]
}
```

**Key Rules:**
- Never use wildcards to grant unrestricted permissions
- Filesystem permissions must include `allow` scope to restrict paths
- `deny` rules always take precedence over `allow` rules

### 2.2 CSP Must Be Explicitly Configured

Content Security Policy is only enabled when explicitly configured. Production environments must set CSP; never set it to `null`.

```json
{
  "app": {
    "security": {
      "csp": {
        "default-src": "'self' customprotocol: asset:",
        "connect-src": "ipc: http://ipc.localhost",
        "style-src": "'self' 'unsafe-inline'",
        "img-src": "'self' asset: http://asset.localhost blob: data:"
      }
    }
  }
}
```

### 2.3 Path Traversal Defense

All file paths from the frontend must be normalized and verified to be within the allowed base directory.

```rust
use dunce::canonicalize;

fn sanitize_path(base: &Path, user_input: &str) -> Result<PathBuf, AppError> {
    let full = base.join(user_input);
    let canonical = canonicalize(&full).map_err(|_| AppError::InvalidPath)?;
    if !canonical.starts_with(base) {
        return Err(AppError::PathTraversal);
    }
    Ok(canonical)
}
```

### 2.4 Input Validation at the Rust Boundary

Never trust data passed from the frontend. All input must be validated at the command entry point.

```rust
#[derive(serde::Deserialize, Validate)]
pub struct FileRequest {
    #[validate(length(min = 1, max = 255))]
    path: String,
}

#[tauri::command]
pub async fn read_file(req: FileRequest) -> Result<String, AppError> {
    req.validate().map_err(|e| AppError::Validation(e.to_string()))?;
    // ...
}
```

### 2.5 Use Isolation Pattern for Untrusted Frontend Code

If the application uses many third-party frontend dependencies or handles sensitive data, enable the Isolation Pattern to add a validation layer for IPC calls.

---

## 3. Rust Backend Principles

### 3.1 Command Naming and Serde Conventions

- Rust backend uses `snake_case` for parameter names, frontend automatically maps to `camelCase`
- Data transfer structs must be annotated with `#[serde(rename_all = "camelCase")]`
- All return types (including errors) must implement `serde::Serialize`

```rust
#[derive(serde::Serialize, serde::Deserialize)]
#[serde(rename_all = "camelCase")]
pub struct UserConfig {
    pub theme_mode: String,
    pub auto_save: bool,
}
```

### 3.2 Define Structured Errors Using thiserror

All error types are defined using `thiserror`, with manual `Serialize` implementation to support cross-IPC transmission.

```rust
use thiserror::Error;
use serde::Serialize;

#[derive(Debug, Error)]
pub enum AppError {
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),
    #[error("Permission denied: {0}")]
    Permission(String),
    #[error("Validation failed: {0}")]
    Validation(String),
}

#[derive(Serialize)]
#[serde(tag = "kind", content = "message")]
enum ErrorKind {
    Io(String),
    Permission(String),
    Validation(String),
}

impl Serialize for AppError {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where S: serde::ser::Serializer {
        let kind = match self {
            Self::Io(_) => ErrorKind::Io(self.to_string()),
            Self::Permission(_) => ErrorKind::Permission(self.to_string()),
            Self::Validation(_) => ErrorKind::Validation(self.to_string()),
        };
        kind.serialize(serializer)
    }
}
```

### 3.3 async Command Parameters Must Use Owned Types

Async commands cannot use borrowed types (`&str`, `&Path`); must use `String`, `PathBuf`, and other owned types.

```rust
// ✅ Correct
#[tauri::command]
pub async fn greet(name: String) -> String {
    format!("Hello, {name}")
}

// ❌ Wrong — async commands cannot use &str
#[tauri::command]
pub async fn greet_bad(name: &str) -> String { ... }
```

### 3.4 Avoid panic in async Commands

A panic in an async command causes the frontend `await` to hang forever with no timeout or rejection. Use `catch_unwind` guards or always return `Result`.

### 3.5 Use tracing Instead of println!

Use the `tracing` crate for structured logging, supporting spans and cross-async context tracking.

```rust
use tracing::{info, warn, error, instrument};

#[tauri::command]
#[instrument(skip(state))]
pub async fn process(state: State<'_, AppState>) -> Result<(), AppError> {
    info!("Starting processing");
    // ...
}
```

---

## 4. Frontend Integration Principles

### 4.1 Wrap invoke Calls as a Type-Safe API Layer

Do not call `invoke()` directly in components; instead wrap as domain API functions.

```typescript
// lib/tauri/api.ts — centralized API layer
import { invoke } from '@tauri-apps/api/core';

export async function readFile(path: string): Promise<string> {
    return invoke<string>('read_file', { path });
}

export async function getSystemInfo(): Promise<SystemInfo> {
    return invoke<SystemInfo>('get_system_info');
}
```

### 4.2 Event Listeners Must Be Cleaned Up

All `unlisten` functions returned by `listen()` and `once()` must be called when the component unmounts.

```typescript
import { listen } from '@tauri-apps/api/event';
import { useEffect } from 'react';

useEffect(() => {
    let unlisten: (() => void) | undefined;

    listen('download-progress', (event) => {
        setProgress(event.payload);
    }).then(fn => { unlisten = fn; });

    return () => {
        unlisten?.();  // must clean up
    };
}, []);
```

### 4.3 Use Automatic Type Generation Tools

Use `tauri-specta` or `tauri-typegen` to automatically generate TypeScript type bindings from Rust commands, avoiding manual type maintenance.

### 4.4 Recommend Zustand for Frontend State Management

In Tauri desktop applications, Zustand is the preferred lightweight state management solution; multi-window state is synchronized via event-driven updates.

---

## 5. IPC Communication Principles

### 5.1 Choose the Right IPC Primitive

| Scenario | Recommended Primitive | Description |
|----------|----------------------|-------------|
| Frontend requests backend to perform an action | **Commands** | Request-response, type-safe, preferred for 90% of scenarios |
| Backend pushes notifications/state changes | **Events** | Publish-subscribe, broadcast or targeted |
| Streaming data / large files / real-time progress | **Channel** | Low latency and high throughput, new in v2 |
| Inter-window frontend communication | **Events** | Via Rust relay |

### 5.2 Large File Transfer Does Not Use IPC

Transferring large files via `invoke` causes severe JSON serialization overhead. Correct approach: Rust writes to a local file then passes the path; frontend uses `convertFileSrc` to load.

```rust
// Rust: write file and return path
#[tauri::command]
pub async fn process_image(data: Vec<u8>) -> Result<String, AppError> {
    let path = temp_dir().join("image.png");
    fs::write(&path, &data).await?;
    Ok(path.to_string_lossy().to_string())
}
```

```typescript
// Frontend: use convertFileSrc to load directly
import { convertFileSrc } from '@tauri-apps/api/core';
const src = convertFileSrc(path);  // asset:// protocol URL
```

### 5.3 Binary Data Uses tauri::ipc::Response

When returning binary data, use `tauri::ipc::Response` instead of JSON serialization.

```rust
#[tauri::command]
pub fn get_binary() -> Result<tauri::ipc::Response, AppError> {
    let data = std::fs::read("file.bin")?;
    Ok(tauri::ipc::Response::new(data))
}
```

### 5.4 Event Naming Convention

Event names only allow: alphanumeric characters + `-` / `/` / `:` / `_`. Avoid dynamically generating event names.

### 5.5 Progress Reporting Uses Channel API

For long tasks exceeding 1 second, use Channel to report progress, avoiding the high latency of the event system.

```rust
use tauri::ipc::Channel;

#[derive(Serialize)]
pub struct ProgressEvent {
    pub current: usize,
    pub total: usize,
    pub percent: u32,
}

#[tauri::command]
pub async fn batch_process(
    paths: Vec<String>,
    channel: Channel<ProgressEvent>,
) -> Result<(), AppError> {
    for (i, path) in paths.iter().enumerate() {
        process(path).await?;
        channel.send(ProgressEvent {
            current: i + 1,
            total: paths.len(),
            percent: ((i + 1) as f64 / paths.len() as f64 * 100.0) as u32,
        })?;
    }
    Ok(())
}
```

---

## 6. State Management Principles

### 6.1 Use State<'_, T> as Dependency Injection

Tauri's `State<T>` system is a type-safe dependency injection container, automatically handling `Send + Sync + 'static` constraints.

```rust
// register state
Builder::default()
    .manage(Mutex::new(AppState::default()))
    .invoke_handler(...)

// inject in commands
#[tauri::command]
pub fn get_count(state: State<'_, Mutex<AppState>>) -> u32 {
    state.lock().unwrap().counter
}
```

### 6.2 Interior Mutability Selection

| Type | Applicable Scenario | Notes |
|------|-------------------|-------|
| `std::sync::Mutex<T>` | Mixed read/write, short critical sections | Do not hold the lock across `.await` points |
| `tokio::sync::Mutex<T>` | Need to hold the lock between awaits | Slightly worse performance than std Mutex |
| `RwLock<T>` | Read-heavy, write-rare | Risk of writer starvation |
| `AtomicBool/Usize` | Simple counters/flags | Lock-free, highest performance |

### 6.3 Use AppHandle for Cross-Thread State Access

`AppHandle` implements `Clone + Send + Sync` and can be safely passed to any thread to access state.

```rust
.setup(|app| {
    let handle = app.handle().clone();
    std::thread::spawn(move || {
        let state = handle.state::<AppState>();
        // ...
    });
    Ok(())
})
```

### 6.4 Persistent State Uses Store Plugin

For settings that need to survive application restarts, use `tauri-plugin-store`.

```rust
use tauri_plugin_store::{Store, StoreBuilder};

Builder::default()
    .plugin(tauri_plugin_store::Builder::default().build())
```

---

## 7. Error Handling Principles

### 7.1 All Commands Return Result<T, AppError>

Commands never panic; always return structured errors. The frontend gets type-safe error objects via `.catch()`.

```typescript
// Frontend: structured error handling
type AppError = { kind: 'io' | 'permission' | 'validation'; message: string };

invoke('read_file', { path })
    .then(data => setContent(data))
    .catch((err: AppError) => {
        switch (err.kind) {
            case 'io': /* ... */ break;
            case 'permission': /* ... */ break;
        }
    });
```

### 7.2 Error Conversion at Command Boundaries

The Service layer uses native Rust error types; the Handler layer uses `map_err()` to convert to serializable `AppError`.

### 7.3 Do Not Leak Internal Error Details to the Frontend

Internal errors (such as database connection strings, stack traces) should not be directly exposed to the frontend; they should be mapped to user-friendly error messages.

---

## 8. Async & Performance Principles

### 8.1 CPU-Intensive Tasks Use spawn_blocking

Do not execute CPU-intensive calculations directly in async commands; use `tokio::task::spawn_blocking`.

```rust
#[tauri::command]
pub async fn compress_file(path: String) -> Result<(), AppError> {
    tokio::task::spawn_blocking(move || {
        heavy_compression(&path)
    })
    .await
    .map_err(|e| AppError::Internal(e.to_string()))?
}
```

### 8.2 Long Tasks Support Cancellation

Operations lasting several seconds should support cancellation using `Arc<AtomicBool>` or `tokio_util::sync::CancellationToken`.

```rust
use std::sync::atomic::{AtomicBool, Ordering};

#[tauri::command]
pub async fn long_task(
    cancel: State<'_, Arc<AtomicBool>>,
) -> Result<(), AppError> {
    for i in 0..100 {
        if cancel.load(Ordering::Relaxed) {
            return Err(AppError::Cancelled);
        }
        // ...
    }
    Ok(())
}
```

### 8.3 Compilation Optimization Configuration

Configure compilation optimizations in `Cargo.toml`'s `[profile.release]` to reduce binary size.

```toml
[profile.release]
opt-level = "s"          # optimize for size
lto = true               # link-time optimization
codegen-units = 1        # single codegen unit
strip = true             # strip symbol table
panic = "abort"          # no unwinding
```

### 8.4 Resource Management: RAII and Drop

Ensure all long-lived resources (file handles, network connections, event listeners) implement proper cleanup. Channel and event listeners may cause memory leaks.

---

## 9. Testing Principles

### 9.1 Three-Layer Testing Model

| Level | Type | Tool | Responsibility |
|-------|------|------|---------------|
| 1 | Rust unit tests | `cargo test` | Pure functions, Service layer logic |
| 2 | Frontend integration tests | Vitest + `mockIPC()` | Frontend components, API layer |
| 3 | E2E tests | WebDriverIO / Playwright | Complete user flows |

### 9.2 Windows Test Isolation

Windows `cargo test` may fail due to manifest issues in the `tao` library. Use `#[cfg(not(test))]` to isolate Tauri runtime dependencies.

```rust
#[cfg(not(test))]
mod tauri_integration;   // compile only in non-test environments

#[cfg(test)]
mod tests {              // pure unit tests
    use super::*;

    #[test]
    fn test_core_logic() {
        assert_eq!(process_data("input"), "expected");
    }
}
```

### 9.3 Frontend Mocking

Use `mockIPC()` and `clearMocks()` from `@tauri-apps/api/mocks` for frontend testing.

```typescript
import { mockIPC, clearMocks } from '@tauri-apps/api/mocks';
import { afterEach } from 'vitest';

afterEach(() => clearMocks());

test('file read', () => {
    mockIPC((cmd, payload) => {
        if (cmd === 'read_file') return 'mocked content';
    });
    // ...
});
```

---

## 10. Build & Deployment Principles

### 10.1 Use Platform-Specific Configuration Overrides

Tauri supports `tauri.{platform}.conf.json` automatic merging for platform-differentiated configuration.

```
src-tauri/
├── tauri.conf.json           # base configuration
├── tauri.linux.conf.json     # Linux overrides
├── tauri.windows.conf.json   # Windows overrides
└── tauri.macos.conf.json     # macOS overrides
```

### 10.2 CI/CD Multi-Platform Signing

A complete release pipeline requires:
- **macOS**: Developer ID certificate + notarization
- **Windows**: Authenticode signing certificate
- **Tauri Updates**: ed25519 key pair to sign update packages

### 10.3 Secure Auto-Updates

The public key is embedded in `tauri.conf.json`; the private key is injected via the `TAURI_SIGNING_PRIVATE_KEY` environment variable at build time — never commit it to version control.

---

## 11. Accessibility Principles

> Tauri application accessibility relies on the underlying system WebView (Windows: WebView2, macOS: WKWebView, Linux: WebKitGTK). WCAG 2.1/2.2 standards apply to desktop software. Semantic HTML takes priority over ARIA; keyboard navigation must be complete; screen reader support is a core requirement.

### 11.1 Semantic HTML Takes Priority Over ARIA

Use native HTML semantic elements (`<button>`, `<nav>`, `<main>`, `<dialog>`) instead of simulating with `<div>`. Native elements have built-in keyboard focus management, screen reader role announcement, and Enter/Space activation support.

```html
<!-- Correct: semantic elements -->
<button @click="submit">Submit</button>
<nav aria-label="Main navigation">...</nav>
<main>...</main>

<!-- Wrong: DIV simulation -->
<div class="btn" @click="submit">Submit</div>
```

### 11.2 Complete Keyboard Navigation Coverage

All interactive elements must be accessible via `Tab`/`Shift+Tab`, following standard keyboard interaction conventions:

| Key | Behavior |
|-----|----------|
| `Tab` / `Shift+Tab` | Focus forward/backward |
| `Enter` / `Space` | Activate buttons/links/menu items |
| `Escape` | Close dialogs/popups/menus |
| `↑↓←→` | Navigate within lists, menus, tabs |
| `Home` / `End` | Jump to first/last item in list |

### 11.3 Focus Indicator Visible (3:1 Contrast)

Keyboard focus indicators must always be visible, meeting WCAG non-text contrast requirements (3:1). Use the `:focus-visible` pseudo-class to render focus rings only during keyboard navigation.

```css
/* Recommended: focus-visible only shows on keyboard */
:focus-visible {
  outline: 2px solid #0066ff;
  outline-offset: 2px;
}

/* Wrong: removing focus ring */
:focus { outline: none; }  /* serious anti-pattern! */
```

### 11.4 Screen Reader Dynamic Content Notification

Use ARIA Live Regions to notify screen reader users of dynamic content updates. Live Regions must be created in the initial HTML (not dynamically injected), otherwise screen readers may miss the initial announcement.

```html
<!-- Pre-place Live Region in the app root template -->
<div id="announcement" aria-live="polite" aria-atomic="true" class="sr-only"></div>
```

```typescript
// Update content to trigger screen reader announcement
function announce(message: string): void {
  const el = document.getElementById('announcement');
  if (el) el.textContent = message;
}

// Usage: operation completion notification
await invoke('save_file');
announce('File saved successfully');  // screen reader reads aloud
```

**Live Region Level Selection:**
- `aria-live="polite"` — default, non-interrupting announcement (save success, list update)
- `aria-live="assertive"` — urgent interrupting announcement (save failed, data loss warning), only use when truly urgent

### 11.5 Support System Accessibility Preferences

Respond to operating system accessibility settings, providing corresponding CSS media query adaptations:

```css
/* Dark/Light mode */
@media (prefers-color-scheme: dark) { /* dark styles */ }

/* Reduced motion */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation: none !important;
    transition: none !important;
  }
}

/* High contrast mode */
@media (prefers-contrast: more) { /* high contrast styles */ }
@media (forced-colors: active) {
  /* Windows high contrast: use system color keywords */
  color: CanvasText;
  background: Canvas;
  border: 1px solid ButtonBorder;
}
```

### 11.6 Dialog Focus Management

Modal dialogs must implement Focus Trap: when opened, focus moves to the first interactive element inside the dialog; when closed, focus returns to the triggering element.

```typescript
// Dialog open: move focus in
useEffect(() => {
  if (isOpen) {
    const firstFocusable = dialogRef.current?.querySelector(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    (firstFocusable as HTMLElement)?.focus();
  }
}, [isOpen]);

// Dialog close: restore focus
const closeAndRestoreFocus = () => {
  setIsOpen(false);
  triggerRef.current?.focus();  // focus back to trigger button
};

// ESC to close
dialogRef.current?.addEventListener('keydown', (e) => {
  if (e.key === 'Escape') closeAndRestoreFocus();
});
```

### 11.7 WCAG Contrast Compliance

All text content must meet WCAG 2.1 Level AA contrast requirements:

| Text Type | Minimum Contrast |
|-----------|-----------------|
| Normal text (< 18pt or < 14pt bold) | **4.5:1** |
| Large text (>= 18pt or >= 14pt bold) | **3:1** |
| UI components/graphics (focus rings, icons) | **3:1** |

Use online tools (e.g., WebAIM Contrast Checker) to verify. Color alone must not be the only carrier of information; it must be accompanied by icons/text/shapes.

### 11.8 Form Accessibility

Every form input must have a programmatically associated label; error hints must be associated with the input field via `aria-describedby`.

```html
<!-- Correct: label associated with input via for -->
<label for="email">Email address</label>
<input id="email" type="email" aria-describedby="email-error" />
<span id="email-error" role="alert">Please enter a valid email address</span>

<!-- Correct: using aria-label when no visible label -->
<input type="search" aria-label="Search content" placeholder="Search..." />

<!-- Wrong: no label association -->
<span>Email</span>
<input type="email" />  <!-- screen reader cannot associate label -->
```

### 11.9 Automated Accessibility Testing

Integrate automated a11y testing tools in CI, covering approximately 57% of WCAG issues:

```typescript
// Vitest + axe-core example
import { test, expect } from 'vitest';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

test('page has no a11y violations', async () => {
  const results = await axe(document.body);
  expect(results).toHaveNoViolations();
});
```

**Recommended Tool Stack:** `axe-core` (automated detection) + `pa11y` (CLI scanning) + manual screen reader testing (NVDA / VoiceOver).

### 11.10 Page Zoom Support

Tauri supports page zoom via `Ctrl/Cmd + +/-` (20%-1000%) by default. Ensure the application functions correctly within this range and layouts are not broken.

```json
// tauri.conf.json — enable zoom hotkeys
{
  "app": {
    "windows": [{
      "zoomHotkeysEnabled": true
    }]
  }
}
```

---

## 12. Internationalization Principles

> Tauri core does not provide built-in internationalization; it must be implemented via a frontend i18n framework + Rust i18n crate combination. The recommended "frontend-led" model: UI translations are managed by the frontend; the Rust backend only returns structured error codes.

### 12.1 Frontend i18n Framework Selection

| Frontend Framework | Recommended Library | Integration |
|--------------------|---------------------|-------------|
| React | `react-i18next` | `useTranslation` hook |
| Vue 3 | `vue-i18n` | `useI18n` composable |
| Svelte | `svelte-i18n` | `$t` store |

```typescript
// React example: useTranslation hook
import { useTranslation } from 'react-i18next';

function App() {
  const { t, i18n } = useTranslation('common');
  return <h1>{t('welcome.title')}</h1>;
}
```

### 12.2 Rust Backend i18n Solution Selection

| Scenario | Recommended Crate | Rationale |
|----------|-------------------|-----------|
| Simple translation (key-value) | `rust-i18n` | Compile-time embedding, `t!` macro, supports YAML/JSON/TOML |
| Complex translation (plural/gender/selectors) | `fluent` | Mozilla design, FTL format, translator-friendly |
| Number/date/currency formatting | `icu` (ICU4X) | Unicode official, CLDR data, covers hundreds of locales |
| System locale detection | `tauri-plugin-os` | Official Tauri plugin, cross-platform BCP-47 tags |

```rust
// rust-i18n example
rust_i18n::i18n!("locales", fallback = "en");

#[tauri::command]
pub fn get_status() -> String {
    t!("status.ready")  // automatically returns based on current locale
}
```

### 12.3 Translation Resource Directory Structure

Split translation files by functional domain and language, with i18next namespace lazy loading:

```
locales/
├── en/
│   ├── common.json      # general text
│   ├── menu.json        # menu/tray
│   ├── settings.json    # settings panel
│   └── errors.json      # error messages
├── zh-CN/
│   ├── common.json
│   ├── menu.json
│   ├── settings.json
│   └── errors.json
└── ar/
    ├── common.json
    ├── menu.json
    ├── settings.json
    └── errors.json
```

```typescript
// i18next lazy loading configuration
import resourcesToBackend from 'i18next-resources-to-backend';

i18next
  .use(resourcesToBackend((lang, ns) =>
    import(`./locales/${lang}/${ns}.json`)
  ))
  .init({
    partialBundledLanguages: true,
    ns: ['common', 'menu', 'settings', 'errors'],
    defaultNS: 'common',
    fallbackLng: 'en',
  });
```

### 12.4 System Language Detection Priority

Language selection follows the following priority (from high to low):

1. **User-saved language preference** (read from `tauri-plugin-store`)
2. **System locale** (via `tauri-plugin-os` `locale()` API)
3. **Default fallback language** (e.g., `'en'`)

```typescript
import { locale } from '@tauri-apps/plugin-os';
import { load } from '@tauri-apps/plugin-store';

async function initLanguage(): Promise<void> {
  const store = await load('settings.json');
  const savedLang = await store.get<string>('language');
  const systemLang = await locale();  // e.g., "zh-CN"

  const targetLang = savedLang || systemLang || 'en';
  await i18n.changeLanguage(targetLang);
}
```

```rust
// Rust backend: get system locale
use tauri_plugin_os::locale;

#[tauri::command]
pub fn get_system_locale() -> Option<String> {
    locale()  // returns BCP-47 tags like "zh-CN", "en-US", etc.
}
```

### 12.5 Dynamic Language Switching (No Restart Required)

Runtime language switching via i18next's `languageChanged` event, synchronously updating UI direction, tray menu, and system menu.

```typescript
const rtlLanguages = ['ar', 'he', 'fa', 'ur'];

i18n.on('languageChanged', async (lng) => {
  // 1. Update document direction (RTL support)
  document.documentElement.dir = rtlLanguages.includes(lng) ? 'rtl' : 'ltr';
  document.documentElement.lang = lng;

  // 2. Notify Rust backend to sync locale
  await invoke('set_locale', { locale: lng });

  // 3. Rebuild system tray menu (new language)
  await rebuildTrayMenu();

  // 4. Rebuild application menu (new language)
  await buildAppMenu();
});
```

```rust
// Rust backend sync locale
#[tauri::command]
pub fn set_locale(locale: String) {
    rust_i18n::set_locale(&locale);
}
```

### 12.6 RTL (Right-to-Left) Language Support

Arabic, Hebrew, and other RTL languages require automatic document direction switching; use CSS logical properties instead of physical properties:

```css
/* Recommended: CSS logical properties (auto-adapt to LTR/RTL) */
.element {
  margin-inline-start: 1rem;   /* replaces margin-left */
  margin-inline-end: 1rem;     /* replaces margin-right */
  padding-inline-start: 0.5rem; /* replaces padding-left */
  text-align: start;            /* replaces text-align: left */
}

/* Layout direction aware */
.flex-row {
  flex-direction: row;  /* automatically reverses visual order in RTL */
}
```

| Physical Property (Avoid) | Logical Property (Recommended) |
|---------------------------|-------------------------------|
| `margin-left` | `margin-inline-start` |
| `margin-right` | `margin-inline-end` |
| `padding-left` | `padding-inline-start` |
| `padding-right` | `padding-inline-end` |
| `text-align: left` | `text-align: start` |
| `text-align: right` | `text-align: end` |
| `border-left` | `border-inline-start` |
| `border-right` | `border-inline-end` |
| `left` | `inset-inline-start` |
| `right` | `inset-inline-end` |

### 12.7 Plurals and Gender: Use ICU MessageFormat

Different languages have different numbers of plural forms; plural logic must be defined in translation files, not hardcoded.

| Language | Plural Forms | Description |
|----------|-------------|-------------|
| English/German | 2 (one, other) | one is only used for n=1 |
| French | 2 (one, other) | one is used for n=0,1 |
| Russian/Ukrainian | 4 (one, few, many, other) | complex rules |
| Arabic | 6 (zero, one, two, few, many, other) | most complex |
| Chinese/Japanese/Korean | 1 (other) | no grammatical plurals |

```yaml
# locales/en/common.yaml (fluent FTL format)
unread-count:
  { $count } unread message{ $count ->
    [one] ""
    *[other] s
  }
```

```typescript
// Frontend uses Intl API to handle plurals (browser native)
const formatter = new Intl.PluralRules('ar');
formatter.select(0);  // "zero"
formatter.select(1);  // "one"
formatter.select(2);  // "two"
```

### 12.8 Frontend-Backend Translation Responsibility Separation

Recommend the "frontend-led" model, avoiding duplicate translation definitions:

| Responsibility | Frontend | Rust Backend |
|----------------|----------|--------------|
| UI text translation | ✅ Lead | ❌ Not handled |
| Error message display | ✅ Map error codes to translation keys | ✅ Only return structured error codes |
| System tray/menu | ✅ Translate at frontend build time | ❌ |
| Logs/debug info | ❌ | ✅ Rust original text (not translated) |
| Number/date/currency formatting | ✅ `Intl.NumberFormat` / `Intl.DateTimeFormat` | ✅ `icu` / `num-format` |

```typescript
// Frontend: map error codes to translations
const errorMap: Record<string, string> = {
  'ERR_FILE_NOT_FOUND': t('errors.fileNotFound'),
  'ERR_PERMISSION_DENIED': t('errors.permissionDenied'),
};

try {
  await invoke('read_file', { path });
} catch (err: AppError) {
  showToast(errorMap[err.code] || t('errors.unknown'));
}
```

```rust
// Backend: only return structured error codes, no translation handling
#[tauri::command]
pub async fn read_file(path: String) -> Result<String, AppErrorCode> {
    if !path.exists() {
        return Err(AppErrorCode::FileNotFound);
    }
    // ...
}
```

### 12.9 Continuous Localization Workflow

Integrate the translation process into CI/CD to achieve "code commit means translation":

```yaml
# .github/workflows/i18n.yml
name: Localization

on:
  push:
    branches: [main]
    paths: ['locales/en/**']  # trigger only when English source files change

jobs:
  sync-translations:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Upload source strings to Crowdin
        run: crowdin upload sources

      - name: Download latest translations
        run: crowdin download

      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v6
        with:
          title: 'chore(i18n): sync translations from Crowdin'
          branch: 'auto/i18n-sync'
```

### 12.10 Number and Date Formatting

Frontend should prioritize the browser native `Intl` API; Rust backend uses `icu` or `num-format`:

```typescript
// Frontend: Intl API (browser native, auto-adapts to current locale)
// Number formatting
new Intl.NumberFormat('de-DE').format(1234567.89);  // "1.234.567,89"

// Currency formatting
new Intl.NumberFormat('ja-JP', { style: 'currency', currency: 'JPY' }).format(1234);
// "￥1,234"

// Date-time formatting
new Intl.DateTimeFormat('zh-CN', {
  dateStyle: 'full',
  timeStyle: 'short'
}).format(new Date());
// "Wednesday, January 15, 2025 at 2:30 PM"
```

```rust
// Backend: ICU4X formatting
use num_format::{Locale, ToFormattedString};

let formatted = 1_000_000.to_formatted_string(&Locale::de); // "1.000.000"
```

---

## 13. Core Design Patterns Cheatsheet

### 13.1 Command Handler Pattern

```rust
// commands/file.rs
use crate::{error::AppError, models::FileRequest};

#[tauri::command]
pub async fn read_file(req: FileRequest) -> Result<String, AppError> {
    // validate → business logic → return
}
```

### 13.2 Dependency Injection via State

```rust
// register → inject → use
Builder::default().manage(FileService::new());

#[tauri::command]
pub async fn read(path: String, service: State<'_, FileService>) -> Result<String, AppError> {
    service.read(&path).await
}
```

### 13.3 Cancel Token Pattern

```rust
// use Arc<AtomicBool> or CancellationToken for cancellable operations
#[tauri::command]
pub async fn cancellable_task(cancel: State<'_, CancelToken>) -> Result<(), AppError> {
    while !cancel.is_cancelled() {
        // work unit + periodic check
    }
}
```

### 13.4 Progress Reporting Pattern

```rust
#[tauri::command]
pub async fn with_progress(paths: Vec<String>, channel: Channel<Progress>) -> Result<(), AppError> {
    for (i, path) in paths.iter().enumerate() {
        process(path).await?;
        channel.send(Progress { current: i + 1, total: paths.len() })?;
    }
    Ok(())
}
```

### 13.5 Event Bus Pattern

```rust
// backend global broadcast
app.emit("state-changed", &new_state)?;

// inter-window frontend communication (via Rust relay)
listen("state-changed", handler);  // all windows receive
```

### 13.6 Plugin Encapsulation Pattern

```rust
// plugin defines independent command namespace: plugin:<name>|<command>
pub fn init<R: Runtime>() -> TauriPlugin<R> {
    Builder::new("my-plugin")
        .invoke_handler(generate_handler![command1, command2])
        .setup(|app| { /* initialization */ Ok(()) })
        .build()
}
```

### 13.7 Typed IPC Contract Pattern

```rust
// Rust side defines shared types
#[derive(Serialize, Deserialize)]
#[serde(rename_all = "camelCase")]
pub struct ApiResponse<T> {
    pub data: T,
    pub timestamp: u64,
}

// TypeScript side gets corresponding types via auto-generation tools
// interface ApiResponse<T> { data: T; timestamp: number; }
```

---

## 14. Anti-Patterns Checklist

| # | Anti-Pattern | Consequence | Correct Approach |
|---|-------------|-------------|-----------------|
| 1 | Writing business logic in `main.rs` | Mobile build fails | All logic goes in `lib.rs` |
| 2 | async commands using `&str` parameters | Compilation error | Use `String` |
| 3 | Holding `std::sync::MutexGuard` across `.await` points | Future is not Send | Reduce lock scope or use `tokio::sync::Mutex` |
| 4 | Executing CPU-intensive calculations directly in async commands | UI freezes | Use `spawn_blocking` |
| 5 | Transferring large file content via IPC | JSON serialization overhead, memory issues | Write to file then use `convertFileSrc` |
| 6 | panic in async commands | Frontend await hangs forever | Always return `Result` |
| 7 | Using `String` as error type | Frontend cannot distinguish error types | Use structured error enums |
| 8 | Event listeners not cleaned up | Memory leaks | Store `unlisten` and call on unmount |
| 9 | Manually wrapping State with `Arc::new()` | Redundant, violates conventions | Use `manage(T)` directly, State handles it automatically |
| 10 | Capability uses wildcard permissions | Security vulnerability | Precisely declare each required permission and scope |
| 11 | CSP set to `null` in production | Loses XSS protection | Explicitly configure minimal necessary CSP |
| 12 | Trusting paths passed from frontend | Path traversal attacks | Normalize and verify path prefix |
| 13 | Error messages leaking internal details | Security risk | Map to user-friendly messages |
| 14 | Windows tests not isolating Tauri dependencies | `STATUS_ENTRYPOINT_NOT_FOUND` | Use `#[cfg(not(test))]` |
| 15 | Using event system for streaming data | High latency, low throughput | Use Channel API |
| 16 | Removing focus ring (`outline: none`) | Keyboard users cannot locate focus | Use `:focus-visible` to keep focus visible |
| 17 | Using `<div>` to simulate buttons/links | Loses keyboard and screen reader support | Use semantic HTML elements |
| 18 | Dynamically injecting ARIA Live Region | Screen readers miss initial announcement | Pre-place Live Region in initial HTML |
| 19 | Color alone as information carrier | Color-blind users cannot distinguish | Accompany with icons/text/shapes |
| 20 | Hardcoding UI text (not extracting translation keys) | Cannot support multiple languages | All user-visible text uses `t('key')` |
| 21 | Rust backend returning translated error messages | Duplicate and inconsistent translations | Backend only returns error codes; frontend maps translations |
| 22 | Using physical CSS properties (`margin-left`) | RTL language layout breaks | Use logical properties (`margin-inline-start`) |
| 23 | Hardcoding plural logic in code | Different languages have different plural rules | Use ICU MessageFormat defined in translation files |
| 24 | Ignoring `prefers-reduced-motion` | Triggers discomfort for vestibular disorder users | Detect and disable animations |

---

## Appendix: Key Terms

| Term | Description |
|------|-------------|
| **Capability** | Tauri v2 permission system capability configuration, binds permissions to specific windows |
| **Command** | Rust backend functions exposed to frontend invocation, annotated with `#[tauri::command]` |
| **Event** | Tauri's publish-subscribe communication mechanism, supports bidirectional frontend/backend sending |
| **Channel** | Tauri v2's new streaming data transmission API, replaces event system for high-throughput scenarios |
| **State** | Tauri's dependency injection system, injects global state into commands by type |
| **AppHandle** | Application handle, can be safely used across threads for emitting events and accessing state |
| **Scope** | Permission scope, restricts the specific resource range a command can operate on (e.g., file paths) |
| **Isolation** | IPC security mode, validates all IPC calls via a sandbox iframe |

---

*This document is compiled based on Tauri v2 official documentation, community best practices, and production experience. It is recommended to review periodically with framework version updates.*
