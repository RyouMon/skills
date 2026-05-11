---
name: tauri-v2-best-practices
description: >
  Tauri v2 desktop application development coding standards and best practices.
  Covers 12 dimensions: Rust backend architecture, frontend integration, IPC
  communication, state management, security model, error handling, async
  performance, accessibility (a11y), internationalization (i18n), testing,
  build/deployment. Provides 60+ actionable principles, 7 design patterns with
  runnable code templates, and 24 anti-patterns checklist. Activate when working
  on Tauri v2 projects: generating command handlers, designing API contracts,
  organizing module structure, implementing IPC, managing app state, handling
  error boundaries, configuring security permissions, writing accessible UI,
  implementing multi-language support, optimizing performance, setting up
  testing strategies, building cross-platform apps. Not for: pure frontend web
  projects, non-Tauri Rust projects, Tauri v1 legacy maintenance.
---

# Tauri v2 Best Practices

## Quick Decision: When to Load Which Reference

| Scenario | Load | Contents |
|----------|------|----------|
| New project / module design | **This file** Architecture Cheatsheet | Directory structure, layering rules |
| Full 60+ coding principles | `references/guidelines.md` | 12-dimension complete guide |
| Implement a design pattern | `references/patterns.md` | 7 patterns with runnable templates |
| Code review / avoiding pitfalls | `references/anti-patterns.md` | 24 anti-patterns checklist |
| Verify principle sources | `references/sources.md` | Official docs citation index |
| Choose IPC primitive | **This file** IPC Selection Table | Commands / Events / Channel |
| Error handling / security / a11y / i18n | `references/guidelines.md` | Detailed rules per dimension |

**Core principle:** Read SKILL.md for global context; load references on demand. Do NOT read all references at once.

---

## Architecture Cheatsheet

### Directory Structure (Mandatory)

```
src-tauri/src/
├── main.rs              # Pass-through only: my_app_lib::run();
├── lib.rs               # Entry, Builder, command registration, plugins
├── commands/            # Thin Handler layer
│   ├── mod.rs           # Module re-exports
│   ├── system.rs
│   └── file.rs
├── services/            # Business logic layer (Service)
├── models/              # DTOs, shared data structures
└── error.rs             # Global error enum
```

**Iron rule:** `lib.rs` carries all logic; `main.rs` is pass-through only — this is a Tauri v2 mobile build requirement.

### Layering: Handler -> Service -> Repository

- **Handler** (`commands/`): Param parsing -> permission check -> call Service -> return `Result<T, AppError>`
- **Service** (`services/`): Core business logic, pure Rust, no Tauri dependency
- **Repository**: Data access isolation (database, filesystem, network)

### Naming Conventions

| Item | Rust Side | Frontend | Note |
|------|-----------|----------|------|
| Command fn | `snake_case` | `camelCase` auto-mapped | Tauri auto-converts |
| DTO fields | `snake_case` | `camelCase` | Must add `#[serde(rename_all = "camelCase")]` |
| Event names | `kebab-case` | `kebab-case` | Only `a-z0-9` + `-/_:` |
| Error enum | `PascalCase` | `kind` field | Custom `Serialize` impl |

---

## Coding Principles Essentials (Top 20%)

> Full 60+ principles in `references/guidelines.md`

### Rust Backend

1. **Async command params must be owned** — Use `String` / `PathBuf`, never `&str` / `&Path`
2. **Never panic** — Panic in async causes frontend `await` to hang forever; always return `Result`
3. **Use `thiserror` + manual `Serialize`** — Errors must cross IPC boundary via serialization
4. **CPU-intensive work via `spawn_blocking`** — Never run heavy compute directly in async command
5. **Use `tracing` for logging** — Supports spans and async context propagation; avoid `println!`
6. **Path traversal defense** — All frontend-provided paths: `dunce::canonicalize` then prefix-check
7. **Validate input at Rust boundary** — Never trust frontend data; validate at command entry

### Frontend Integration

8. **Wrap `invoke()` in typed API layer** — No raw `invoke()` in components; use domain API functions
9. **Always clean up event listeners** — Store `unlisten` fn; call on component unmount
10. **Use auto type generation** — `tauri-specta` or `tauri-typegen` for Rust->TypeScript bindings

### IPC Communication

11. **Choose the right primitive** (see table below)
12. **Large files: write to disk + `convertFileSrc`** — Never transfer large data through IPC
13. **Binary data: use `tauri::ipc::Response`** — Avoids JSON serialization overhead
14. **Progress reporting: use Channel API** — For operations > 1 second

### Security

15. **Default deny all permissions** — Each window's Capability in `capabilities/*.json`
16. **Never wildcard permissions** — Explicitly scope filesystem paths with `allow`
17. **CSP must be explicit in production** — Never set `csp: null`

### Error Handling

18. **All commands return `Result<T, AppError>`** — Structured errors, not strings
19. **Map errors at command boundary** — Service uses native errors; Handler maps to serializable
20. **Don't leak internals to frontend** — No stack traces, connection strings, or internal details

### IPC Primitive Selection

| Scenario | Primitive | Direction | Pattern |
|----------|-----------|-----------|---------|
| Frontend calls backend API | **Commands** | FE -> BE | Request-response, typed |
| Backend pushes notification | **Events** | BE -> FE | Pub-sub, broadcast or targeted |
| Stream data / large payloads | **Channel** | BE -> FE | Low-latency streaming |
| Cross-window communication | **Events** | FE <-> FE | Via Rust relay |

---

## Gotchas (Most Common Traps)

1. **Using `&str` in async commands** -> Compile error; use `String`
2. **Holding `std::sync::MutexGuard` across `.await`** -> `Future` not `Send`; use `tokio::sync::Mutex` or narrow lock scope
3. **CPU work in async without `spawn_blocking`** -> UI freeze
4. **Transferring large files via IPC `invoke`** -> JSON serialization kills performance; use `convertFileSrc`
5. **Panic in async command** -> Frontend `await` hangs forever with no timeout
6. **Using `String` as error type** -> Frontend can't distinguish error kinds; use structured enums
7. **Event listeners without cleanup** -> Memory leak; always call `unlisten()`
8. **Manual `Arc::new()` for State** -> Redundant; `manage(T)` auto-wraps
9. **Wildcard permissions in Capability** -> Security hole; always scope paths
10. **`csp: null` in production** -> Disables XSS protection
11. **Trusting frontend file paths** -> Path traversal attack; always canonicalize + prefix-check
12. **`outline: none` removing focus ring** -> Keyboard users lost; use `:focus-visible`
13. **Using `<div role="button">`** -> Loses keyboard + screen reader support; use semantic HTML
14. **Hardcoding UI text** -> Blocks i18n; always use `t('key')`
15. **Rust backend returning translated errors** -> Duplicated translations; return error codes only
16. **Physical CSS props (`margin-left`)** -> Breaks RTL; use logical props (`margin-inline-start`)
17. **Hardcoding plural logic in code** -> Languages have different plural rules; use ICU MessageFormat

---

## Code Templates

### lib.rs: Builder Setup (Mandatory Pattern)

```rust
#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .plugin(tauri_plugin_store::Builder::new().build())
        .manage(AppState::default())
        .invoke_handler(tauri::generate_handler![
            commands::system::get_info,
            commands::file::read,
            commands::file::write,
        ])
        .setup(|app| {
            // Init logic: tray, shortcuts, event listeners
            let handle = app.handle().clone();
            Ok(())
        })
        .run(tauri::generate_context!())
        .expect("error running app");
}
```

### Thin Command Handler + Structured Error

```rust
// error.rs
use thiserror::Error;
use serde::Serialize;

#[derive(Debug, Error)]
pub enum AppError {
    #[error("io error: {0}")]
    Io(#[from] std::io::Error),
    #[error("permission denied: {0}")]
    Permission(String),
    #[error("validation failed: {0}")]
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
    fn serialize<S: serde::Serializer>(&self, s: S) -> Result<S::Ok, S::Error> {
        let kind = match self {
            Self::Io(_) => ErrorKind::Io(self.to_string()),
            Self::Permission(_) => ErrorKind::Permission(self.to_string()),
            Self::Validation(_) => ErrorKind::Validation(self.to_string()),
        };
        kind.serialize(s)
    }
}

// commands/file.rs
use crate::{error::AppError, models::FileRequest, services::file_service};

#[tauri::command]
pub async fn read_file(req: FileRequest) -> Result<String, AppError> {
    req.validate().map_err(|e| AppError::Validation(e.to_string()))?;
    let sanitized = validate_path(&req.path)?;
    file_service::read(&sanitized).await
}
```

### Frontend: Typed API Layer + Event Listener Cleanup

```typescript
// lib/tauri/api.ts
import { invoke } from '@tauri-apps/api/core';

export async function readFile(path: string): Promise<string> {
    return invoke<string>('read_file', { path });
}

// Component.tsx
import { listen } from '@tauri-apps/api/event';
import { useEffect } from 'react';

useEffect(() => {
    let unlisten: (() => void) | undefined;
    listen('download-progress', (e) => setProgress(e.payload))
        .then(fn => { unlisten = fn; });
    return () => { unlisten?.(); };  // Must cleanup
}, []);
```

### Capability Configuration Template

```json
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

### Accessibility: Live Region Announcement

```typescript
// Pre-create in app root HTML:
// <div id="a11y-live" aria-live="polite" aria-atomic="true" class="sr-only"></div>

function announce(message: string): void {
    const el = document.getElementById('a11y-live');
    if (el) el.textContent = message;
}

await invoke('save_file');
announce('File saved successfully');  // Screen reader announces
```

### Internationalization: Language Switch

```typescript
const rtlLanguages = ['ar', 'he', 'fa', 'ur'];

i18n.on('languageChanged', async (lng) => {
    document.documentElement.dir = rtlLanguages.includes(lng) ? 'rtl' : 'ltr';
    document.documentElement.lang = lng;
    await invoke('set_locale', { locale: lng });
    await rebuildTrayMenu();
});
```

---

## Security Checklist

Before any command touches filesystem, network, or shell:

- [ ] Capability JSON explicitly grants permission with scoped paths
- [ ] Input validated at command entry (length, format, path traversal)
- [ ] Paths canonicalized with `dunce::canonicalize` + prefix-checked
- [ ] Errors don't leak internal details (no stack traces, no connection strings)
- [ ] CSP configured in production (never `null`)
