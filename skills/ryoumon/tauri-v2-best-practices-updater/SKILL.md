---
name: tauri-v2-best-practices-updater
description: >
  Automatically re-research and update the Tauri v2 coding standards skill.
  Activate when user says "update coding standards", "refresh AGENTS.md",
  "re-research Tauri best practices", "sync latest standards", or similar.
  Executes deep research across 12 dimensions (architecture, security,
  Rust backend, frontend integration, IPC, state management, error handling,
  async performance, testing, build/deploy, accessibility, i18n) to identify
  API changes, new best practices, deprecated features, and new dimensions.
  Outputs updated complete standards files. Not for: initial creation (use
  tauri-v2-best-practices skill instead), non-Tauri projects.
metadata:
  last_updated: "2026-05-11"
  version: "1.0.0"
---

# Tauri v2 Coding Standards Auto-Updater Skill

## 1. Trigger Recognition

Activate this skill when user input matches any of these patterns:

| Pattern | Examples |
|---------|----------|
| update / refresh / sync + standards | "update coding standards", "refresh AGENTS.md" |
| re-research / regenerate + Tauri | "re-research Tauri best practices" |
| upgrade / update + skill | "upgrade tauri skill" |
| check + outdated / latest | "check if standards are outdated" |

**Do NOT activate** (route to other skills):
- Initial standards creation -> use `tauri-v2-best-practices` skill
- Non-Tauri project -> use corresponding tech stack skill
- Single snippet edit -> edit directly, skip update workflow

## 2. Update Workflow Overview

Execute a **5-phase** update process. Load the corresponding reference at each phase:

| Phase | Load | Task |
|-------|------|------|
| Phase 0 | `references/workflow.md` section Phase 0 | Load existing standards, extract metadata |
| Phase 1 | `references/workflow.md` section Phase 1 | Change scan: 5 searches/dimension |
| Phase 2 | `references/workflow.md` section Phase 2 | Deep dive: >=15 searches/flagged dimension |
| Phase 3 | `references/workflow.md` section Phase 3 | Cross-verify: mark new/changed/deprecated |
| Phase 4 | `references/workflow.md` section Phase 4 | Generate updated standards |
| Phase 5 | `references/checklist.md` | Quality gate: verify all checks |

**Flow control:**
- If Phase 1 determines no dimension has changes, skip to Phase 5 (verification only)
- If Phase 3 finds >30 changes, recommend splitting the update into multiple sessions
- Phase 5 any check failure -> return to Phase 4, iterate until all pass

## 3. Change Markup Convention

After Phase 3, annotate all changes in the output:

| Mark | Meaning | Example |
|------|---------|---------|
| `**[NEW]**` | Newly added principle | `Always use Channel for progress **[NEW]**` |
| `**[CHG: old -> new]**` | Changed principle | `Use spawn_blocking **[CHG: spawn -> spawn_blocking]**` |
| `**[DEP]**` | Deprecated principle | `v1 allowlist config **[DEP: use capabilities instead]**` |
| `**[TBD]**` | Uncertain, needs verification | `Mobile plugin API **[TBD]**` |

Append a **Change Log** section at the end of the updated standards:

```markdown
## Change Log

### 2026-05-11
- Added: 3 new principles in Security (CVE-2026-XXXX mitigation)
- Changed: IPC primitive selection table (Channel now preferred for files >10MB)
- Deprecated: `tauri::api` path (removed in v2.5)
- New dimension: WebAssembly integration (Phase 2 finding)
```

## 4. Output Format

After Phase 5 passes all checks, save the updated standards to:

```
/mnt/agents/output/AGENTS.md          # Full standards (for human reference)
```

Or if updating skill files:
```
/mnt/agents/output/tauri-v2-best-practices/references/guidelines.md   # Updated principles
/mnt/agents/output/tauri-v2-best-practices/references/patterns.md      # Updated patterns
/mnt/agents/output/tauri-v2-best-practices/references/anti-patterns.md  # Updated anti-patterns
/mnt/agents/output/tauri-v2-best-practices/references/sources.md       # Updated sources
```

Top marker:
```markdown
<!-- Last updated: YYYY-MM-DD | Dimensions: X | Principles: Y | Sources: Z -->
```
