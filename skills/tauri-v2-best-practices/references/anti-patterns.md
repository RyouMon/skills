# 24 Anti-Patterns Checklist

> Check item by item when reviewing code. Each item includes: anti-pattern description, consequences, correct approach.

---

## Architecture & Modules

### 1. Writing Business Logic in `main.rs`

- **Consequences**: Mobile build fails. Tauri v2 requires the mobile entry point in `lib.rs`; `main.rs` logic cannot be called by mobile.
- **Correct Approach**: All business logic goes in `lib.rs`, `main.rs` is only one line forwarding: `fn main() { my_app_lib::run(); }`

### 2. Command Handlers Containing Business Logic

- **Consequences**: Bloated Handler, difficult to test, mixed responsibilities.
- **Correct Approach**: Handler layer stays "thin" — only parameter parsing, permission checking, calling Service, returning results. Business logic sinks to `services/`.

### 3. Manually Wrapping State with `Arc::new()`

- **Consequences**: Redundant code, violates Tauri conventions. `.manage(T)` already handles `Arc` wrapping automatically.
- **Correct Approach**: Use `.manage(T)` directly, `State<'_, T>` auto-injects. Only use explicit `Arc` when `Clone` is needed.

---

## Async & Concurrency

### 4. async Commands Using `&str` / `&Path` Parameters

- **Consequences**: Compilation error. Async commands cannot guarantee reference lifetimes across await points.
- **Correct Approach**: Use `String`, `PathBuf`, and other owned types.

```rust
// ❌ Wrong
pub async fn bad(name: &str) -> String { ... }
// ✅ Correct
pub async fn good(name: String) -> String { ... }
```

### 5. Holding `std::sync::MutexGuard` Across `.await` Points

- **Consequences**: `Future` does not implement `Send`, cannot be scheduled across threads, compilation error or runtime deadlock.
- **Correct Approach**: Reduce lock scope, release before await; or switch to `tokio::sync::Mutex`.

```rust
// ❌ Wrong — lock crosses await
let guard = mutex.lock().unwrap();
let result = some_async_fn().await;  // compilation error!
drop(guard);

// ✅ Correct — reduce lock scope
{
    let guard = mutex.lock().unwrap();
    let data = guard.clone();
}
let result = some_async_fn().await;

// ✅ Correct — use tokio::sync::Mutex
let guard = tokio_mutex.lock().await;
let result = some_async_fn().await;  // allowed
```

### 6. Executing CPU-Intensive Calculations Directly in async Commands

- **Consequences**: Blocks async runtime thread, UI freezes, unresponsive.
- **Correct Approach**: Use `tokio::task::spawn_blocking` to move CPU-intensive tasks out of the async runtime.

```rust
// ✅ Correct
pub async fn compress(path: String) -> Result<(), AppError> {
    tokio::task::spawn_blocking(move || heavy_work(&path))
        .await
        .map_err(|e| AppError::Internal(e.to_string()))?
}
```

### 7. panic in async Commands

- **Consequences**: Frontend `await` hangs forever, no timeout, no rejection, UI deadlocks.
- **Correct Approach**: Always return `Result<T, AppError>`, use `?` to propagate errors.

---

## IPC Communication

### 8. Transferring Large File Content via IPC

- **Consequences**: Huge JSON serialization overhead, memory spikes, may cause application crash.
- **Correct Approach**: Rust writes to temp file → returns file path → frontend loads with `convertFileSrc(path)`.

```typescript
// ✅ Correct — transfer path, not file content
const path = await invoke<string>('save_temp_image', { imageData });
const src = convertFileSrc(path);  // asset:// protocol
imgElement.src = src;
```

### 9. Using Event System for Streaming Data

- **Consequences**: Event system has high latency and low throughput, not suitable for high-frequency data push.
- **Correct Approach**: Use `Channel<T>` API, designed for streaming data.

### 10. Dynamically Generating Event Names

- **Consequences**: Difficult to track, prone to naming conflicts, security auditing difficulty.
- **Correct Approach**: Define event name constants in modules, use static strings.

```rust
// ✅ Correct
pub const PROGRESS_EVENT: &str = "app://download-progress";
app.emit(PROGRESS_EVENT, payload)?;
```

---

## Security

### 11. Capability Using Wildcard Permissions

- **Consequences**: Attacker gains full filesystem, network, and shell access.
- **Correct Approach**: Precisely declare each required permission and scope, principle of least privilege.

```json
// ❌ Wrong — wildcards
"permissions": ["fs:*", "shell:*"]

// ✅ Correct — precise declaration
"permissions": [
  { "identifier": "fs:allow-read-text-file", "allow": [{ "path": "$APPDATA/**" }] }
]
```

### 12. CSP Set to `null` in Production

- **Consequences**: Completely loses XSS protection, frontend scripts can execute arbitrary code.
- **Correct Approach**: Explicitly configure minimally necessary CSP policy.

### 13. Trusting Paths Passed from Frontend

- **Consequences**: Path traversal attacks, attacker can read sensitive files like `/etc/passwd`.
- **Correct Approach**: Normalize paths and validate with `canonicalize` within the allowed base directory.

```rust
fn sanitize_path(base: &Path, user_input: &str) -> Result<PathBuf, AppError> {
    let canonical = dunce::canonicalize(base.join(user_input))?;
    if !canonical.starts_with(base) {
        return Err(AppError::PathTraversal);
    }
    Ok(canonical)
}
```

### 14. Error Messages Leaking Internal Details to Frontend

- **Consequences**: Exposes database structure, internal paths, tech stack information, increasing attack surface.
- **Correct Approach**: Map internal errors to user-friendly messages; logs retain detailed information.

```rust
// ❌ Wrong — leaks internal info
Err(AppError::Db(format!("Connection to {} failed: {:?}", conn_string, err)))

// ✅ Correct — user-friendly + internal log
tracing::error!("DB connection failed: {:?}", err);
Err(AppError::Internal("Service temporarily unavailable".into()))
```

---

## Error Handling

### 15. Using `String` as Error Type

- **Consequences**: Frontend cannot distinguish error types, can only do fuzzy handling.
- **Correct Approach**: Use structured error enums with custom `Serialize` implementation for cross-IPC transmission.

```rust
// ❌ Wrong
pub async fn cmd() -> Result<String, String> { ... }

// ✅ Correct
#[derive(Debug, Error)]
pub enum AppError { ... }
pub async fn cmd() -> Result<String, AppError> { ... }
```

### 16. Event Listeners Not Cleaned Up

- **Consequences**: Memory leaks, duplicate listeners cause logic errors.
- **Correct Approach**: Store `unlisten` and call on component unmount.

```typescript
useEffect(() => {
    let unlisten: (() => void) | undefined;
    listen('event', handler).then(fn => { unlisten = fn; });
    return () => { unlisten?.(); };
}, []);
```

---

## Frontend Integration

### 17. Calling `invoke()` Directly in Components

- **Consequences**: IPC calls scattered everywhere, type-unsafe, difficult to maintain and test.
- **Correct Approach**: Encapsulate as a domain API layer; components call type-safe wrapper functions.

### 18. Improper Frontend State Management (Prop Drilling / Global Variables)

- **Consequences**: Multi-window state inconsistency, code difficult to maintain.
- **Correct Approach**: Use lightweight state management like Zustand; multi-window sync via Events.

---

## Testing

### 19. Windows Tests Not Isolating Tauri Dependencies

- **Consequences**: `STATUS_ENTRYPOINT_NOT_FOUND` error, `cargo test` fails.
- **Correct Approach**: Use `#[cfg(not(test))]` to isolate Tauri runtime dependencies; unit tests stay pure logic.

```rust
#[cfg(not(test))]
mod tauri_integration;

#[cfg(test)]
mod tests {
    #[test]
    fn pure_logic_test() { ... }
}
```

---

## Accessibility

### 20. Removing Focus Ring (`outline: none`)

- **Consequences**: Keyboard users completely unable to locate current focus, violates WCAG.
- **Correct Approach**: Use `:focus-visible` to show focus ring only during keyboard navigation.

```css
:focus-visible {
  outline: 2px solid #0066ff;
  outline-offset: 2px;
}
```

### 21. Using `<div>` to Simulate Buttons/Links

- **Consequences**: Loses keyboard focus management, Enter/Space activation, screen reader role announcement.
- **Correct Approach**: Use semantic HTML elements; reset default styles when custom styling is needed.

### 22. Dynamically Injecting ARIA Live Region

- **Consequences**: Screen readers may miss initial announcement, dynamic content notification is unreliable.
- **Correct Approach**: Pre-place Live Region in the initial HTML of the app root template; only update content at runtime.

```html
<div id="announcement" aria-live="polite" aria-atomic="true" class="sr-only"></div>
```

### 23. Color Alone as Information Carrier

- **Consequences**: Color-blind/low-vision users cannot distinguish states (e.g., red/green success/failure).
- **Correct Approach**: Color must accompany icons/text/shapes to convey information.

```html
<!-- ✅ Correct — icon + color + text -->
<span class="text-red-500">
  <ErrorIcon /> Save failed
</span>
```

### 24. Ignoring `prefers-reduced-motion`

- **Consequences**: Triggers discomfort for users with vestibular disorders (dizziness, nausea).
- **Correct Approach**: Detect media query and disable animations.

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation: none !important;
    transition: none !important;
  }
}
```

---

## Quick Checklist

```
□ main.rs only has forwarding code           □ async parameters are all owned types
□ Handler layer is thin, logic is in Service  □ no std MutexGuard held across await points
□ no wildcard permissions                     □ async commands return Result, no panic
□ CSP is explicitly configured                □ large files use convertFileSrc, not IPC
□ paths validated by canonicalize             □ event listeners have unlisten cleanup
□ errors don't leak internal details          □ use Channel instead of Events for streaming data
□ use thiserror + Serialize                   □ focus ring visible (:focus-visible)
□ Windows tests use cfg(not(test))            □ semantic HTML takes priority over ARIA
□ Live Region pre-placed in HTML              □ color accompanied by icons/text
□ prefers-reduced-motion handled              □ frontend invoke encapsulated as API layer
```
