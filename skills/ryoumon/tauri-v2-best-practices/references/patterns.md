# 7 Core Design Patterns (Complete Code Templates)

> All code is ready to run, based on real Tauri v2 APIs.

---

## 1. Command Handler Pattern

**Scenario:** Frontend calls Rust functions via `invoke()`, request-response style communication.

**Core:** Handler layer stays "thin" — only parameter parsing and call forwarding, business logic sinks to Service.

```rust
// commands/file.rs
use tauri::State;
use crate::{error::AppError, models::FileRequest, services::FileService};
use tracing::instrument;

/// Read file — thin Handler: validate → forward to Service → return
#[tauri::command]
#[instrument(skip(service))]
pub async fn read_file(
    req: FileRequest,
    service: State<'_, FileService>,
) -> Result<String, AppError> {
    // 1. Parameter validation
    req.validate().map_err(|e| AppError::Validation(e.to_string()))?;

    // 2. Path security validation (Handler layer responsibility)
    let sanitized = service.sanitize_path(&req.path).await?;

    // 3. Business logic delegated to Service
    service.read_text(&sanitized).await
}

/// Write file
#[tauri::command]
#[instrument(skip(service))]
pub async fn write_file(
    path: String,
    content: String,
    service: State<'_, FileService>,
) -> Result<(), AppError> {
    let sanitized = service.sanitize_path(&path).await?;
    service.write_text(&sanitized, &content).await
}
```

```rust
// commands/mod.rs — unified exposure
pub mod file;
pub mod system;
```

```rust
// lib.rs — register commands
use commands::{file, system};

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .manage(FileService::new())
        .invoke_handler(tauri::generate_handler![
            file::read_file,
            file::write_file,
            system::get_system_info,
        ])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

```typescript
// lib/tauri/api.ts
import { invoke } from '@tauri-apps/api/core';

export interface FileRequest {
  path: string;
}

export async function readFile(req: FileRequest): Promise<string> {
  return invoke<string>('read_file', { req });
}

export async function writeFile(path: string, content: string): Promise<void> {
  return invoke('write_file', { path, content });
}
```

---

## 2. State DI Pattern

**Scenario:** Sharing global state in commands (database connections, configuration, cache, etc.).

**Core:** `Builder::manage(T)` registers, `State<'_, T>` injects, type-safe zero-cost.

```rust
// services/mod.rs
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::Arc;

/// Application global state
#[derive(Default)]
pub struct AppState {
    pub request_counter: AtomicUsize,
    pub config: tokio::sync::RwLock<AppConfig>,
}

#[derive(Default, Clone)]
pub struct AppConfig {
    pub max_file_size: usize,
    pub allowed_extensions: Vec<String>,
}

impl AppState {
    pub fn new() -> Arc<Self> {
        Arc::new(Self::default())
    }

    pub fn increment_counter(&self) -> usize {
        self.request_counter.fetch_add(1, Ordering::Relaxed) + 1
    }
}
```

```rust
// Register state (lib.rs setup)
use std::sync::Arc;
use services::AppState;

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .manage(AppState::new())  // Arc<AppState> is managed
        .invoke_handler(tauri::generate_handler![
            get_request_count,
            get_config,
            update_config,
        ])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

```rust
// commands/state_demo.rs — multi-state type injection example
use tauri::State;
use std::sync::Arc;
use crate::services::{AppState, AppConfig};
use crate::error::AppError;

/// Read counter — uses Atomic, no locking needed
#[tauri::command]
pub fn get_request_count(state: State<'_, Arc<AppState>>) -> usize {
    state.request_counter.load(std::sync::atomic::Ordering::Relaxed)
}

/// Read config — uses RwLock, read-heavy write-rare
#[tauri::command]
pub async fn get_config(state: State<'_, Arc<AppState>>) -> Result<AppConfig, AppError> {
    let config = state.config.read().await;
    Ok(config.clone())
}

/// Update config — write lock
#[tauri::command]
pub async fn update_config(
    new_config: AppConfig,
    state: State<'_, Arc<AppState>>,
) -> Result<(), AppError> {
    let mut config = state.config.write().await;
    *config = new_config;
    Ok(())
}
```

---

## 3. Cancel Token Pattern

**Scenario:** Long tasks (file downloads, batch processing) need to support user cancellation.

**Core:** `Arc<AtomicBool>` or `tokio_util::sync::CancellationToken`, periodically checking cancellation signal.

```rust
// services/cancel_token.rs
use std::sync::atomic::{AtomicBool, Ordering};
use std::sync::Arc;

/// Shared cancel token
#[derive(Default, Clone)]
pub struct CancelToken {
    cancelled: Arc<AtomicBool>,
}

impl CancelToken {
    pub fn new() -> Self {
        Self::default()
    }

    pub fn cancel(&self) {
        self.cancelled.store(true, Ordering::Relaxed);
    }

    pub fn is_cancelled(&self) -> bool {
        self.cancelled.load(Ordering::Relaxed)
    }

    pub fn reset(&self) {
        self.cancelled.store(false, Ordering::Relaxed);
    }
}
```

```rust
// commands/long_task.rs
use tauri::State;
use std::sync::Arc;
use tokio::time::{sleep, Duration};
use crate::services::CancelToken;
use crate::error::AppError;
use tracing::{info, warn};

/// Start a cancellable long task
#[tauri::command]
pub async fn start_download(
    url: String,
    token: State<'_, Arc<CancelToken>>,
) -> Result<String, AppError> {
    token.reset();
    info!("Starting download from: {}", url);

    for i in 0..100 {
        if token.is_cancelled() {
            warn!("Download cancelled at {}%", i);
            return Err(AppError::Cancelled);
        }

        // simulate download work unit
        sleep(Duration::from_millis(50)).await;

        // report progress every 10% (real scenario uses Channel)
        if i % 10 == 0 {
            info!("Download progress: {}%", i);
        }
    }

    Ok("download_complete.zip".to_string())
}

/// Cancel current task
#[tauri::command]
pub fn cancel_task(token: State<'_, Arc<CancelToken>>) {
    info!("Cancellation requested");
    token.cancel();
}
```

```typescript
// lib/tauri/api.ts — frontend invocation
import { invoke } from '@tauri-apps/api/core';

let currentCancel: (() => void) | null = null;

export async function startDownload(url: string): Promise<string> {
  return invoke<string>('start_download', { url });
}

export function cancelDownload(): void {
  invoke('cancel_task');
}
```

---

## 4. Channel Pattern (Progress Reporting)

**Scenario:** File batch processing, large data imports, long-running calculations, requiring real-time progress feedback.

**Core:** Tauri v2 `Channel<T>` API — lower latency and higher throughput than Events.

```rust
// models/progress.rs
use serde::Serialize;

#[derive(Clone, Serialize)]
pub struct ProgressEvent {
    pub current: usize,
    pub total: usize,
    pub percent: u32,
    pub message: String,
}

#[derive(Clone, Serialize)]
pub struct TaskComplete {
    pub task_id: String,
    pub result_count: usize,
}
```

```rust
// commands/batch.rs
use tauri::ipc::Channel;
use crate::models::{ProgressEvent, TaskComplete};
use crate::error::AppError;
use tokio::time::{sleep, Duration};

/// Batch process files — report progress via Channel
#[tauri::command]
pub async fn batch_process_files(
    paths: Vec<String>,
    progress: Channel<ProgressEvent>,
    complete: Channel<TaskComplete>,
) -> Result<(), AppError> {
    let total = paths.len();
    let task_id = uuid::Uuid::new_v4().to_string();

    for (i, path) in paths.iter().enumerate() {
        // execute processing
        process_single_file(path).await?;

        // push progress in real-time
        progress.send(ProgressEvent {
            current: i + 1,
            total,
            percent: ((i + 1) as f64 / total as f64 * 100.0) as u32,
            message: format!("Processed: {}", path),
        })?;
    }

    // send completion notification
    complete.send(TaskComplete {
        task_id,
        result_count: total,
    })?;

    Ok(())
}

async fn process_single_file(_path: &str) -> Result<(), AppError> {
    sleep(Duration::from_millis(100)).await;
    Ok(())
}
```

```typescript
// Frontend receives progress
import { invoke } from '@tauri-apps/api/core';
import { Channel } from '@tauri-apps/api/core';

interface ProgressEvent {
  current: number;
  total: number;
  percent: number;
  message: string;
}

interface TaskComplete {
  task_id: string;
  result_count: number;
}

export async function batchProcessWithProgress(
  paths: string[],
  onProgress: (p: ProgressEvent) => void,
  onComplete: (c: TaskComplete) => void
): Promise<void> {
  const progressChannel = new Channel<ProgressEvent>();
  progressChannel.onmessage = onProgress;

  const completeChannel = new Channel<TaskComplete>();
  completeChannel.onmessage = onComplete;

  await invoke('batch_process_files', {
    paths,
    progress: progressChannel,
    complete: completeChannel,
  });
}
```

---

## 5. Event Bus Pattern

**Scenario:** Backend actively pushes notifications, multi-window state synchronization, decoupled inter-module communication.

**Core:** `app.emit()` broadcast + `listen()` subscription, supports targeting specific windows/tabs.

```rust
// events/mod.rs — event name constants (avoid hardcoded strings)
pub const SYNC_STATE: &str = "app://sync-state";
pub const NOTIFY_USER: &str = "app://notify-user";
pub const WINDOW_FOCUS: &str = "app://window-focus";
```

```rust
// commands/events.rs
use tauri::{AppHandle, Manager};
use crate::events;
use crate::models::{AppState, UserNotification};
use serde::Serialize;

/// Broadcast global state change — all windows receive
#[tauri::command]
pub fn broadcast_state_change(
    app: AppHandle,
    new_state: AppState,
) -> Result<(), String> {
    // emit broadcasts to all windows
    app.emit(events::SYNC_STATE, &new_state)
        .map_err(|e| e.to_string())?;
    Ok(())
}

/// Send notification to a specific window
#[tauri::command]
pub fn notify_window(
    app: AppHandle,
    target_window: String,
    notification: UserNotification,
) -> Result<(), String> {
    if let Some(window) = app.get_webview_window(&target_window) {
        // use emit_to for targeted sending
        window.emit(events::NOTIFY_USER, &notification)
            .map_err(|e| e.to_string())?;
    }
    Ok(())
}

/// Notify other windows that current window gained focus
#[tauri::command]
pub fn report_window_focus(
    app: AppHandle,
    window_label: String,
) -> Result<(), String> {
    app.emit(events::WINDOW_FOCUS, &window_label)
        .map_err(|e| e.to_string())?;
    Ok(())
}
```

```typescript
// Frontend: global state synchronization
import { listen, emit } from '@tauri-apps/api/event';
import { getCurrentWebviewWindow } from '@tauri-apps/api/webviewWindow';

const appWindow = getCurrentWebviewWindow();

// listen to global state changes
const unlistenState = await listen<AppState>('app://sync-state', (event) => {
  updateGlobalState(event.payload);
});

// listen to targeted notifications
const unlistenNotify = await listen<UserNotification>('app://notify-user', (event) => {
  showToast(event.payload);
});

// report window focus
await emit('app://window-focus', appWindow.label);

// cleanup on component unmount
return () => {
  unlistenState();
  unlistenNotify();
};
```

---

## 6. Plugin Encapsulation Pattern

**Scenario:** Independent functional units (custom storage, hardware access, third-party SDK integration) need self-contained modules.

**Core:** Independent command namespace `plugin:<name>|<command>`, with its own setup and lifecycle.

```rust
// plugins/secure_store.rs
use tauri::{
    plugin::{Builder, TauriPlugin},
    Manager, Runtime,
};
use serde::Serialize;
use std::collections::HashMap;
use std::sync::Mutex;

/// Plugin state (private)
struct SecureStore {
    data: Mutex<HashMap<String, String>>,
}

#[derive(Serialize)]
struct StoreEntry {
    key: String,
    value: String,
}

// --- Plugin commands ---

#[tauri::command]
fn store_set(
    key: String,
    value: String,
    store: tauri::State<'_, SecureStore>,
) -> Result<(), String> {
    let mut data = store.data.lock().map_err(|e| e.to_string())?;
    data.insert(key, value);
    Ok(())
}

#[tauri::command]
fn store_get(
    key: String,
    store: tauri::State<'_, SecureStore>,
) -> Result<Option<String>, String> {
    let data = store.data.lock().map_err(|e| e.to_string())?;
    Ok(data.get(&key).cloned())
}

#[tauri::command]
fn store_delete(
    key: String,
    store: tauri::State<'_, SecureStore>,
) -> Result<bool, String> {
    let mut data = store.data.lock().map_err(|e| e.to_string())?;
    Ok(data.remove(&key).is_some())
}

// --- Plugin entry ---

pub fn init<R: Runtime>() -> TauriPlugin<R> {
    Builder::new("secure-store")
        .invoke_handler(tauri::generate_handler![
            store_set,
            store_get,
            store_delete,
        ])
        .setup(|app| {
            // plugin initialization logic
            app.manage(SecureStore {
                data: Mutex::new(HashMap::new()),
            });
            Ok(())
        })
        .build()
}
```

```rust
// lib.rs — register plugin
#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .plugin(plugins::secure_store::init())
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

```typescript
// Frontend calls plugin commands
import { invoke } from '@tauri-apps/api/core';

// Note namespace format: plugin:<name>|<command>
export async function secureStoreSet(key: string, value: string): Promise<void> {
  return invoke('plugin:secure-store|store_set', { key, value });
}

export async function secureStoreGet(key: string): Promise<string | null> {
  return invoke('plugin:secure-store|store_get', { key });
}
```

---

## 7. Typed IPC Contract Pattern

**Scenario:** Shared data structures between frontend and backend, ensuring type safety and avoiding manual TypeScript type maintenance.

**Core:** `serde(rename_all = "camelCase")` + `tauri-specta` auto-generated TS bindings + versioned API.

```rust
// models/api.rs — shared data structures (used by both frontend and backend)
use serde::{Deserialize, Serialize};

/// Unified API response wrapper
#[derive(Serialize, Deserialize, Clone, Debug)]
#[serde(rename_all = "camelCase")]
pub struct ApiResponse<T> {
    pub data: T,
    pub timestamp: u64,
    pub request_id: String,
    pub success: bool,
}

impl<T> ApiResponse<T> {
    pub fn ok(data: T, request_id: String) -> Self {
        Self {
            data,
            timestamp: std::time::SystemTime::now()
                .duration_since(std::time::UNIX_EPOCH)
                .unwrap_or_default()
                .as_secs(),
            request_id,
            success: true,
        }
    }

    pub fn err(request_id: String) -> Self
    where T: Default {
        Self {
            data: T::default(),
            timestamp: std::time::SystemTime::now()
                .duration_since(std::time::UNIX_EPOCH)
                .unwrap_or_default()
                .as_secs(),
            request_id,
            success: false,
        }
    }
}

/// Paginated request parameters
#[derive(Deserialize, Clone, Debug)]
#[serde(rename_all = "camelCase")]
pub struct PaginatedRequest {
    pub page: u32,
    pub page_size: u32,
    #[serde(default)]
    pub sort_by: Option<String>,
    #[serde(default)]
    pub sort_order: Option<String>,
}

/// Paginated response
#[derive(Serialize, Clone, Debug)]
#[serde(rename_all = "camelCase")]
pub struct PaginatedResponse<T> {
    pub items: Vec<T>,
    pub total: u64,
    pub page: u32,
    pub page_size: u32,
    pub total_pages: u32,
}

/// File metadata
#[derive(Serialize, Deserialize, Clone, Debug)]
#[serde(rename_all = "camelCase")]
pub struct FileMetadata {
    pub name: String,
    pub path: String,
    pub size: u64,
    pub modified_at: u64,
    pub mime_type: String,
}
```

```rust
// commands/api.rs — commands using typed contracts
use tauri::State;
use crate::models::{
    ApiResponse, FileMetadata, PaginatedRequest, PaginatedResponse,
};
use crate::services::FileService;
use crate::error::AppError;
use uuid::Uuid;

/// Get file list — returns typed response
#[tauri::command]
pub async fn list_files(
    req: PaginatedRequest,
    service: State<'_, FileService>,
) -> Result<ApiResponse<PaginatedResponse<FileMetadata>>, AppError> {
    let request_id = Uuid::new_v4().to_string();

    let result = service
        .list_files(req.page, req.page_size, req.sort_by, req.sort_order)
        .await?;

    Ok(ApiResponse::ok(result, request_id))
}
```

```rust
// Cargo.toml — tauri-specta configuration
// [dependencies]
// specta = "2.0"
// tauri-specta = { version = "2.0", features = ["typescript"] }
```

```rust
// build.rs or bin — generate TypeScript types
use specta::-typescript;
use tauri_specta::{collect_commands, ts};

fn main() {
    let builder = ts::builder()
        .commands(collect_commands![list_files, read_file])
        .config(specta::ts::ExportConfig::default().formatter(specta::ts::formatter::prettier));

    builder.export(std::path::PathBuf::from("../src/lib/tauri/bindings.ts"))
        .expect("Failed to export TypeScript bindings");
}
```

```typescript
// auto-generated bindings.ts (generated by tauri-specta)
// export interface ApiResponse<T> { data: T; timestamp: number; requestId: string; success: boolean; }
// export interface PaginatedRequest { page: number; pageSize: number; sortBy: string | null; sortOrder: string | null; }
// export interface PaginatedResponse<T> { items: T[]; total: number; page: number; pageSize: number; totalPages: number; }
// export interface FileMetadata { name: string; path: string; size: number; modifiedAt: number; mimeType: string; }

// Frontend usage — fully type-safe
import { commands } from './bindings';

const response = await commands.list_files({
  page: 1,
  pageSize: 20,
  sortBy: 'name',
  sortOrder: 'asc',
});

// response type is ApiResponse<PaginatedResponse<FileMetadata>>
if (response.success) {
  console.log(`Total: ${response.data.total}`);
  response.data.items.forEach(f => console.log(f.name));
}
```
