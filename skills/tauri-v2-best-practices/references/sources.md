# Research Source Index

> Source tracking for all principles in this document. Categorized by topic with citation markers.

---

## Tauri Official Documentation

| Citation Marker | Source | URL |
|-----------------|--------|-----|
| [TAURI-ARCH] | Tauri v2 Architecture Guide | https://v2.tauri.app/concept/architecture/ |
| [TAURI-IPC] | Tauri v2 Inter-Process Communication | https://v2.tauri.app/concept/inter-process-communication/ |
| [TAURI-CMD] | Tauri v2 Commands | https://v2.tauri.app/develop/calling-rust/ |
| [TAURI-STATE] | Tauri v2 State Management | https://v2.tauri.app/develop/state-management/ |
| [TAURI-PLUGIN] | Tauri v2 Plugin Development | https://v2.tauri.app/develop/plugins/ |
| [TAURI-SECURITY] | Tauri v2 Security | https://v2.tauri.app/security/ |
| [TAURI-CSP] | Tauri v2 CSP Configuration | https://v2.tauri.app/security/csp/ |
| [TAURI-CAP] | Tauri v2 Capabilities | https://v2.tauri.app/security/capabilities/ |
| [TAURI-SCOPE] | Tauri v2 Scope Permissions | https://v2.tauri.app/security/scope/ |
| [TAURI-BUILD] | Tauri v2 Build Configuration | https://v2.tauri.app/reference/config/#build |
| [TAURI-EVENT] | Tauri v2 Events | https://v2.tauri.app/develop/calling-frontend-rust/ |
| [TAURI-CHANNEL] | Tauri v2 Channels | https://v2.tauri.app/develop/calling-rust/#channels |
| [TAURI-TEST] | Tauri v2 Testing | https://v2.tauri.app/develop/tests/ |
| [TAURI-DISTRIBUTE] | Tauri v2 Distribution | https://v2.tauri.app/distribute/ |
| [TAURI-UPDATE] | Tauri v2 Updater | https://v2.tauri.app/plugin/updater/ |
| [TAURI-STORE] | Tauri Plugin Store | https://v2.tauri.app/plugin/store/ |
| [TAURI-OS] | Tauri Plugin OS | https://v2.tauri.app/plugin/os/ |
| [TAURI-DIALOG] | Tauri Plugin Dialog | https://v2.tauri.app/plugin/dialog/ |
| [TAURI-FS] | Tauri Plugin FS | https://v2.tauri.app/plugin/file-system/ |

---

## Rust Ecosystem

| Citation Marker | Source | URL |
|-----------------|--------|-----|
| [RUST-ASYNC] | Asynchronous Programming in Rust | https://rust-lang.github.io/async-book/ |
| [RUST-SEND-SYNC] | The Rustonomicon - Send and Sync | https://doc.rust-lang.org/nomicon/send-and-sync.html |
| [THISERROR] | thiserror crate | https://docs.rs/thiserror/latest/ |
| [TRACING] | tracing crate | https://docs.rs/tracing/latest/ |
| [SPECTA] | specta - Type generation | https://docs.rs/specta/latest/ |
| [TOKIO] | Tokio runtime | https://docs.rs/tokio/latest/ |
| [DUNCE] | dunce - Windows path handling | https://docs.rs/dunce/latest/ |
| [SERDE] | serde - Serialization framework | https://serde.rs/ |

---

## Security References

| Citation Marker | Source | URL |
|-----------------|--------|-----|
| [OWASP-PATH] | OWASP Path Traversal | https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html |
| [CSP-GUIDE] | Mozilla CSP Guide | https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP |
| [W3C-CSP] | W3C Content Security Policy | https://www.w3.org/TR/CSP3/ |

---

## Accessibility (a11y)

| Citation Marker | Source | URL |
|-----------------|--------|-----|
| [WCAG21] | WCAG 2.1 Specification | https://www.w3.org/TR/WCAG21/ |
| [WCAG22] | WCAG 2.2 Specification | https://www.w3.org/TR/WCAG22/ |
| [ARIA-LIVE] | MDN ARIA Live Regions | https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions |
| [FOCUS-VISIBLE] | MDN :focus-visible | https://developer.mozilla.org/en-US/docs/Web/CSS/:focus-visible |
| [AXE-CORE] | axe-core testing library | https://github.com/dequelabs/axe-core |
| [A11Y-PROJECT] | The A11Y Project | https://www.a11yproject.com/ |

---

## Internationalization (i18n)

| Citation Marker | Source | URL |
|-----------------|--------|-----|
| [ICU4X] | ICU4X Unicode Library | https://icu4x.unicode.org/ |
| [RUST-I18N] | rust-i18n crate | https://github.com/longbridge/rust-i18n |
| [FLUENT] | Project Fluent (Mozilla) | https://projectfluent.org/ |
| [I18NEXT] | i18next framework | https://www.i18next.com/ |
| [BCP47] | BCP 47 Language Tags | https://tools.ietf.org/html/bcp47 |
| [INTL-API] | MDN Intl API | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl |

---

## Community & Production Practices

| Citation Marker | Source | URL |
|-----------------|--------|-----|
| [TAURI-DISCORD] | Tauri Official Discord | https://discord.gg/tauri |
| [TAURI-GITHUB] | Tauri GitHub Discussions | https://github.com/tauri-apps/tauri/discussions |
| [ZUSTAND] | Zustand state management | https://github.com/pmndrs/zustand |
| [VITEST] | Vitest testing framework | https://vitest.dev/ |
| [PLAYWRIGHT] | Playwright E2E testing | https://playwright.dev/ |
| [CROWDIN] | Crowdin localization platform | https://crowdin.com/ |

---

## Chapter-to-Source Mapping

| Chapter | Primary Source Markers |
|---------|----------------------|
| 1. Architecture Principles | [TAURI-ARCH] |
| 2. Security Principles | [TAURI-SECURITY], [TAURI-CSP], [TAURI-CAP], [OWASP-PATH] |
| 3. Rust Backend Principles | [TAURI-CMD], [THISERROR], [TRACING], [SERDE] |
| 4. Frontend Integration Principles | [TAURI-CMD], [ZUSTAND] |
| 5. IPC Communication Principles | [TAURI-IPC], [TAURI-EVENT], [TAURI-CHANNEL] |
| 6. State Management Principles | [TAURI-STATE], [TOKIO] |
| 7. Error Handling Principles | [THISERROR], [TAURI-CMD] |
| 8. Async & Performance Principles | [TOKIO], [RUST-ASYNC] |
| 9. Testing Principles | [TAURI-TEST], [VITEST], [PLAYWRIGHT] |
| 10. Build & Deployment Principles | [TAURI-DISTRIBUTE], [TAURI-UPDATE] |
| 11. Accessibility Principles | [WCAG21], [WCAG22], [ARIA-LIVE], [AXE-CORE] |
| 12. Internationalization Principles | [I18NEXT], [RUST-I18N], [ICU4X], [BCP47], [INTL-API] |

---

*This document is compiled based on Tauri v2 official documentation, community best practices, and production experience. It is recommended to review periodically with framework version updates.*
