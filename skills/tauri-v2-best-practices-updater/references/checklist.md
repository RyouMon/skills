# Tauri v2 Coding Standards Post-Update Quality Checklist

> This checklist is used in Phase 5 to verify the quality of the updated AGENTS.md standards.
> Each check item includes: check content, verification method, pass criteria, and failure handling.
> All check items must pass before the final standards file can be output.

---

## A. Code Example Checks

Verify that all code examples are syntactically correct and can be compiled and run under the latest stable version of Tauri v2.

### A.1 Rust Code Examples

| # | Check Item | Verification Method | Pass Criteria |
|---|------------|---------------------|---------------|
| A.1.1 | All Rust code blocks are tagged with `rust` language marker | Regex match code block header | No omissions, no incorrect tags (e.g., `rs` should be `rust`) |
| A.1.2 | `#[tauri::command]` function parameter types are valid | Compare with `tauri::command` macro requirements | Async commands have no `&str`, `&Path`, or other borrowed types; sync command parameter types are valid |
| A.1.3 | Used crates and types exist and can be imported | Compare with docs.rs or Cargo.toml | No spelling errors; used crates exist in the ecosystem |
| A.1.4 | `T` in `State<'_, T>` implements `Send + Sync + 'static` | Rust type system knowledge | `T` conforms to Tauri State constraints |
| A.1.5 | No `std::sync::MutexGuard` held across await points in async functions | Code review | Use `tokio::sync::Mutex` or narrow the lock scope |
| A.1.6 | `T` in `tauri::ipc::Channel<T>` implements `Serialize` | Compare with Tauri v2 API | Channel payload type is serializable |
| A.1.7 | `tauri::ipc::Response` is used correctly | Compare with API documentation | Use `Response::new(data)` when returning binary data |
| A.1.8 | `Builder::default()` chain method calls exist | Compare with tauri::Builder API | No calls to deprecated methods |
| A.1.9 | Types injected via `manage()` are not double-wrapped with `Arc::new()` | Code review | Use `manage(T)` directly, not `manage(Arc::new(T))` (unless T itself requires Arc) |
| A.1.10 | Capability JSON schema reference and field names are correct | Compare with tauri v2 capability format | `$schema`, `identifier`, `windows`, `permissions` field names are correct |
| A.1.11 | `thiserror` derive macro and manual `Serialize` implementation have no conflicts | Code review | `#[derive(Debug, Error)]` and manual `impl Serialize` are correctly separated |
| A.1.12 | `tracing` macro usage syntax is correct | Compare with tracing documentation | `#[instrument]` attribute, `info!()` / `warn!()` / `error!()` macro syntax is correct |
| A.1.13 | `spawn_blocking` return value is correctly handled | Code review | Correctly unwrap `JoinHandle` result after `.await?` |
| A.1.14 | `CancellationToken` or `Arc<AtomicBool>` Ordering is used correctly | Code review | Cancellation check uses `Ordering::Relaxed` or `Ordering::SeqCst` (depending on scenario) |
| A.1.15 | `#[serde(rename_all = "camelCase")]` annotation is present | Code review | All front-end/back-end shared structs have this annotation |
| A.1.16 | Macro call syntax for i18n crates such as `rust-i18n`, `fluent`, `icu` is correct | Compare with each crate's documentation | `t!()` macro, `fluent::FluentBundle` API calls are correct |

**Failure Handling**:
- Compilation errors (A.1.2 ~ A.1.4, A.1.6 ~ A.1.9, A.1.11, A.1.13 ~ A.1.16) → **Must fix**, return to Phase 4 to fix code examples
- Spelling or tagging errors (A.1.1, A.1.5, A.1.10, A.1.12) → **Must fix**, return to Phase 4 to fix
- If unable to verify (crate does not exist in current environment) → Search docs.rs to confirm; if still uncertain, mark as `[To Be Verified]`

### A.2 TypeScript Code Examples

| # | Check Item | Verification Method | Pass Criteria |
|---|------------|---------------------|---------------|
| A.2.1 | All TS code blocks are tagged with `typescript` language marker | Regex match | No omissions |
| A.2.2 | `invoke()` call parameters match Rust command signature | Front-end/back-end code comparison | Function names (camelCase ↔ snake_case), parameter names are consistent |
| A.2.3 | `listen()` return value `unlisten` function is properly cleaned up | Code review | Called in React `useEffect` cleanup, Vue `onUnmounted`, etc. |
| A.2.4 | `convertFileSrc()` is used in the correct scenario | Code review | Only used to convert file paths returned by Rust into front-end loadable URLs |
| A.2.5 | `@tauri-apps/api/*` import paths are correct | Compare with Tauri v2 npm package structure | `@tauri-apps/api/core`, `@tauri-apps/api/event`, etc. paths are correct |
| A.2.6 | `mockIPC()` usage syntax conforms to Tauri v2 mocks API | Compare with `@tauri-apps/api/mocks` documentation | `mockIPC((cmd, payload) => ...)` callback signature is correct |

### A.3 JSON / TOML / YAML / CSS Configuration Examples

| # | Check Item | Verification Method | Pass Criteria |
|---|------------|---------------------|---------------|
| A.3.1 | `tauri.conf.json` field names are consistent with v2 schema | Compare with `tauri.conf.json` schema | No v1 legacy fields (e.g., `tauri > allowlist`) |
| A.3.2 | `capabilities/*.json` permission identifier format is correct | Compare with Tauri v2 permission documentation | Format is `{plugin}:{action}` or `{plugin}:allow-{action}` |
| A.3.3 | `Cargo.toml` `[profile.release]` field names are correct | Compare with Cargo documentation | `opt-level`, `lto`, `codegen-units`, `strip`, `panic` fields are correct |
| A.3.4 | CSS media query syntax is correct | Code review | `@media (prefers-reduced-motion: reduce)` and similar syntax are correct |
| A.3.5 | HTML semantic elements and ARIA attribute syntax is correct | Compare with HTML standards and ARIA specification | `aria-live`, `aria-describedby`, `role` and other attribute values are valid |

---

## B. Source Reference Checks

Verify that all external links are valid, sources are authoritative, and information is not outdated.

### B.1 Link Validity

| # | Check Item | Verification Method | Pass Criteria |
|---|------------|---------------------|---------------|
| B.1.1 | All `https://` links return 200 status code | Visit one by one (HEAD request) | No 404/403/500 errors |
| B.1.2 | `site:tauri.app` documentation links point to the latest version | Manual check | Not v1 documentation paths |
| B.1.3 | `docs.rs` links point to the correct crate and version | Manual check | Crate name and version number are correct |
| B.1.4 | GitHub links point to the correct repo and file path | Manual check | Not deleted files or moved paths |

**Failure Handling**:
- Link 404 → Try searching for the new URL; if not found, remove the link or change to a text reference
- Link points to an old version → Update to the latest version link
- Link temporarily inaccessible (403/timeout) → Mark as `[Link To Be Verified]`, does not block release

### B.2 Source Authority

| # | Check Item | Verification Method | Pass Criteria |
|---|------------|---------------------|---------------|
| B.2.1 | Core architecture principles have tauri.app official documentation references | Check evidence for changed principles | Each MODIFY/ADD architecture principle has at least one official source |
| B.2.2 | Security-related principles reference official security documentation or security bulletins | Check security dimension references | No purely community-sourced security claims |
| B.2.3 | API usage examples reference docs.rs or official documentation | Check API calls in code examples | Used APIs have official documentation support |
| B.2.4 | Third-party crate recommendations have clear rationale and alternative comparisons | Check crate recommendations | Recommendation rationale is clear, not personal preference |

### B.3 Timeliness

| # | Check Item | Verification Method | Pass Criteria |
|---|------------|---------------------|---------------|
| B.3.1 | All external source publication dates are after 2024-01 | Check source dates | Prefer sources from 2024-2025 |
| B.3.2 | Referenced APIs are not marked deprecated in the latest stable Tauri v2 | Search docs.rs or source code | No deprecated API references |
| B.3.3 | Tauri v1 content is no longer referenced (except for comparison) | Full-text search for "v1", "allowlist", "tauri::Builder::tauri" and other v1 characteristics | No v1 remnants in the main text; comparison scenarios are clearly marked as v1 |

---

## C. Completeness Checks

Verify that the standards content is comprehensive, with no missing dimensions or principles.

### C.1 Dimension Coverage

| # | Check Item | Verification Method | Pass Criteria |
|---|------------|---------------------|---------------|
| C.1.1 | All 12 core dimensions are present | Check table of contents and body | Architecture, Security, Rust Backend, Front-end Integration, IPC Communication, State Management, Error Handling, Async Performance, Testing, Build & Deployment, Accessibility, Internationalization |
| C.1.2 | Core design pattern quick reference table exists | Check section | Section 13 exists, containing common pattern code examples |
| C.1.3 | Anti-pattern list exists | Check section | Section 14 exists, with continuous numbering |
| C.1.4 | Glossary exists | Check section | Glossary at the end of the document exists, containing core term definitions |

### C.2 Principle Coverage

| # | Check Item | Verification Method | Pass Criteria |
|---|------------|---------------------|---------------|
| C.2.1 | Each dimension has at least 3 principles | Count per dimension | No empty-shell dimensions |
| C.2.2 | All principles have code examples or configuration examples | Check per principle | Pure text principles do not exceed 20% of the total |
| C.2.3 | All principles have a clear "why" (motivation explanation) | Readability check | Each principle explains its value, not just a list |
| C.2.4 | New principles have corresponding entries in the anti-pattern list | Anti-pattern cross-check | Each ADD principle corresponds to at least one anti-pattern |

### C.3 Anti-Pattern Coverage

| # | Check Item | Verification Method | Pass Criteria |
|---|------------|---------------------|---------------|
| C.3.1 | Anti-pattern numbers are continuous with no gaps | Check anti-pattern table | Numbers from 1 to N, no omissions |
| C.3.2 | Each anti-pattern has: name, consequence, correct approach | Check table row by row | All three columns are non-empty |
| C.3.3 | Anti-patterns correspond one-to-one with principles | Cross-reference check | Each anti-pattern can be traced back to a specific principle number |
| C.3.4 | Anti-patterns for deprecated principles are marked | Check anti-pattern table | Anti-pattern rows for DEPRECATE principles have deprecation tags |

### C.4 Cross-References

| # | Check Item | Verification Method | Pass Criteria |
|---|------------|---------------------|---------------|
| C.4.1 | Principle numbers referenced in the body are accurate | Full-text search for `X.Y` format references | No references to non-existent numbers |
| C.4.2 | Cross-dimension references are clearly marked | Check cross-dimension references | E.g., when IPC principles reference security principles, marked as "see 2.X" |
| C.4.3 | Table of contents links match body headings | Click table of contents links one by one | TOC anchors match headings |

---

## D. Consistency Checks

Verify uniform terminology, formatting, and style within the standards.

### D.1 Terminology Consistency

| # | Check Item | Verification Method | Pass Criteria |
|---|------------|---------------------|---------------|
| D.1.1 | Core terms are unified throughout the document (see glossary) | Full-text keyword search | "Capability" not mixed with "ability/permission/function"; "Channel" not mixed with "channel/conduit" |
| D.1.2 | API names are exactly consistent with official documentation (case-sensitive) | Compare with docs.rs | `State<'_, T>` not `state<'_, T>`; `convertFileSrc` not `convert_file_src` |
| D.1.3 | Chinese-English mixed format is uniform | Visual check | Consistent spacing around English terms (e.g., `State<'_, T>` not `State < '_, T >`) |

### D.2 Format Consistency

| # | Check Item | Verification Method | Pass Criteria |
|---|------------|---------------------|---------------|
| D.2.1 | Heading levels are uniform | Structure check | `#` → File title; `##` → Dimension; `###` → Principle; `####` → Sub-item |
| D.2.2 | Code block tags are uniform | Regex check | Rust=`rust`, TypeScript=`typescript`, JSON=`json`, TOML=`toml`, YAML=`yaml`, CSS=`css`, HTML=`html` |
| D.2.3 | Table format is uniform | Visual check | Table header `---` separator, columns aligned, no empty lines interrupting |
| D.2.4 | Correct/incorrect example tags are uniform | Full-text search | Correct uses `// ✅ Correct`, incorrect uses `// ❌ Incorrect` |
| D.2.5 | Change tag format is uniform | Regex check | `**[New]**`, `**[Change: Old→New]**`, `**[Deprecated: v2.x]**` format is correct |

### D.3 Style Consistency

| # | Check Item | Verification Method | Pass Criteria |
|---|------------|---------------------|---------------|
| D.3.1 | Tone is consistent (technical documentation style, not colloquial) | Paragraph reading | No subjective expressions like "I think", "maybe", etc. |
| D.3.2 | Principle description structure is consistent | Sampling check | Each principle: Title → Description → Code Example/Table → (Optional) Key rules list |
| D.3.3 | Code example length is appropriate | Sampling check | Single code block does not exceed 80 lines; if exceeded, split into multiple blocks or reference external files |
| D.3.4 | Comment language is uniform | Full-text search | Code comments use English (consistent with the document language) |

---

## E. Progressive Disclosure Checks

Verify that SKILL.md does not bloat excessively, and detailed content remains in the references/ directory.

### E.1 SKILL.md Size

| # | Check Item | Verification Method | Pass Criteria |
|---|------------|---------------------|---------------|
| E.1.1 | SKILL.md total line count does not exceed 500 lines | `wc -l SKILL.md` | `lines <= 500` |
| E.1.2 | SKILL.md does not contain lengthy content beyond the complete search keyword list | Content review | Search keywords are placed in tables, no redundant explanations |
| E.1.3 | SKILL.md references detailed content via pointers rather than inline expansion | Content review | "See references/workflow.md" instead of expanded descriptions |

### E.2 references/ Directory Structure

| # | Check Item | Verification Method | Pass Criteria |
|---|------------|---------------------|---------------|
| E.2.1 | `references/workflow.md` exists and content is complete | File check | Contains complete Phase 0-5 workflow |
| E.2.2 | `references/checklist.md` exists and content is complete | File check | Contains A-E five categories of check items |
| E.2.3 | No files under references/ directory duplicate content with SKILL.md | Text comparison | SKILL.md is only a summary; detailed content is in references/ |

### E.3 Content Layering is Reasonable

| # | Check Item | Verification Method | Pass Criteria |
|---|------------|---------------------|---------------|
| E.3.1 | SKILL.md contains only: trigger conditions, workflow overview, quick checks, change tag specifications, output format, keyword quick reference | Content review | No specific execution steps, no detailed check items, no code examples |
| E.3.2 | workflow.md contains: specific steps for each Phase, decision points, input/output, rollback strategies | Content review | Execution details are not in SKILL.md |
| E.3.3 | checklist.md contains: item-by-item checklists, verification methods, pass criteria, failure handling | Content review | Check criteria are not in SKILL.md or workflow.md |

---

## F. Final Confirmation Checklist

After all checks pass, execute the following final confirmations:

| # | Confirmation Item | Action |
|---|-------------------|--------|
| F.1 | Changelog has been appended | Check that the changelog at the end of the document contains the update date and all change summaries |
| F.2 | Version number/date has been updated | Check that `last_updated` or equivalent marker in the file header has been updated |
| F.3 | SKILL.md line count is compliant | Execute `wc -l SKILL.md` to confirm <= 500 lines |
| F.4 | No TODO/FIXME remnants | Full-text search for `TODO`, `FIXME`, `XXX` to confirm all are cleaned up |
| F.5 | No `[To Be Verified]` leftovers (unless intentionally retained) | Full-text search for `[To Be Verified]` to confirm each has a clear rationale |
| F.6 | No Markdown syntax errors | Check table closure, code block closure, and link format through rendering preview |

---

## Check Execution Record Template

Fill in when executing checks:

```markdown
## Check Execution Record

| Category | Total Items | Pass | Fail | N/A | Notes |
|----------|-------------|------|------|-----|-------|
| A. Rust Code | 16 | | | | |
| A. TypeScript Code | 6 | | | | |
| A. Config Examples | 5 | | | | |
| B. Link Validity | 4 | | | | |
| B. Source Authority | 4 | | | | |
| B. Timeliness | 3 | | | | |
| C. Dimension Coverage | 4 | | | | |
| C. Principle Coverage | 4 | | | | |
| C. Anti-Pattern Coverage | 4 | | | | |
| C. Cross-References | 3 | | | | |
| D. Terminology Consistency | 3 | | | | |
| D. Format Consistency | 5 | | | | |
| D. Style Consistency | 4 | | | | |
| E. SKILL.md Size | 3 | | | | |
| E. references/ Structure | 3 | | | | |
| E. Content Layering | 3 | | | | |
| F. Final Confirmation | 6 | | | | |

**Total**: __/__ Pass
**Status**: ⬜ Pass / ⬜ Needs Fix
**Fix Items**: (List items that need to return to Phase 4 for correction)
```
