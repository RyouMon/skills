# Tauri v2 Coding Standards Update Workflow

> This document defines the complete update workflow from Phase 0 to Phase 5, for AI Agent to execute stage by stage.
> Each phase includes: objectives, inputs, outputs, specific execution steps, decision points, and rollback strategies.

---

## Phase 0: Load Existing Standards

**Objective**: Fully read and parse the existing standards file, establishing an update baseline.

### Execution Steps

1. **Locate the standards file**
   - Search in the following priority order:
     1. User-specified path
     2. `AGENTS.md` in the current project root directory
     3. `docs/AGENTS.md` in the current project root directory
     4. `AGENTS.md` in the skill references directory
   - If not found, terminate the workflow and prompt the user to run the `tauri-v2-best-practices` initialization skill first

2. **Extract metadata**
   Extract from the file header and overall structure:
   ```yaml
   metadata:
     last_updated: "YYYY-MM-DD"  # Extracted from comments or changelog
     total_dimensions: 12        # Actual dimension count
     total_principles: 0         # To be counted
     total_anti_patterns: 0      # To be counted
     total_code_examples: 0      # To be counted
     tauri_version_reference: "" # Referenced Tauri version
     source_links: []            # Collect all external links
   ```

3. **Structure parsing**
   Read the directory structure and extract the dimension list:
   - Traverse all `## N. Dimension Name` headings
   - Record `### N.M Principle Title` under each dimension
   - Record all code block language tags and line counts
   - Collect all external URLs (`https://...`)

4. **Establish change tracking table**
   ```markdown
   | Dimension | Principle Count | Change Status | Search Count | Notes |
   |-----------|-----------------|---------------|--------------|-------|
   | 1. Architecture | 4 | Pending Review | 0 | |
   | 2. Security | 5 | Pending Review | 0 | |
   | ... | ... | ... | ... | ... |
   ```

### Output

- `metadata` object
- `dimension_map`: mapping of dimension → principle list
- `change_tracking_table`: change tracking table (initial status all "Pending Review")

### Decision Points

- File corrupted or unparseable → Prompt user to manually check and terminate the workflow
- Version number below v2 (e.g., v1 content found) → Mark as "Major Upgrade", increase Phase 2 search depth

---

## Phase 1: Change Scanning

**Objective**: Perform a quick search for each dimension to identify potentially outdated content and narrow the scope for in-depth research.

### Execution Steps

For each dimension (12 total), execute the following:

#### 1.1 Execute Quick Search (5 searches per dimension)

Use the first 5 keywords listed in the "12-Dimension Search Keyword Quick Reference" in SKILL.md:

```
Search 1: "tauri v2 {dimension} best practice 2025"
Search 2: "tauri v2 {dimension} {specific technical point} 2025"  (select the most relevant from the keyword list)
Search 3: "site:tauri.app {dimension} {key API}"
Search 4: "repo:tauri-apps/tauri {dimension} breaking change 2025"
Search 5: "repo:tauri-apps/plugins-workspace {dimension} new feature 2025"
```

#### 1.2 Change Identification Rules

Compare search results with existing standards content, checking for the following change signals:

| Signal | Criteria | Tag |
|--------|----------|-----|
| API Change | Search results show core API signature/name differs from the standard | **API Change** |
| New Best Practice | New pattern/new crate/new plugin not covered by the standard | **Addition Candidate** |
| Official Recommendation Change | tauri.app documentation recommendation differs from the standard | **Recommendation Change** |
| Deprecation Warning | Search results mention deprecation/legacy/removed | **Deprecation Candidate** |
| Version Incompatibility | Code in the standard cannot compile with the new Tauri version | **Incompatible** |
| Security Patch | CVE or patch related to the standard is found | **Security Update** |

#### 1.3 Dimension Grading

Based on the number of change signals, mark each dimension as:

| Level | Signal Count | Meaning | Phase 2 Processing |
|-------|-------------|---------|--------------------|
| 🟢 Stable | 0 signals | Content may still be valid | Skip in-depth research |
| 🟡 Needs Attention | 1-2 signals | Minor changes possible | Standard in-depth research (15 searches) |
| 🔴 Needs Update | 3+ signals | Clearly needs updating | Enhanced in-depth research (20+ searches) |
| 🆕 New Dimension | New area discovered | New dimension not covered by the standard | In-depth research + new dimension design |

### Output

- Updated `change_tracking_table`, with an added `Change Level` column per row
- `phase1_findings.md` for each dimension (temporary notes recording specific signals discovered and source links)
- `phase1_summary`: list of dimensions requiring in-depth research

### Decision Points

- All dimensions are 🟢 Stable → Proceed directly to Phase 5 (verify existing standards are still valid), skipping Phases 2-4
- New dimension discovered (e.g., a major new functional area added to Tauri) → Add "new dimension research" task in Phase 2

### Rollback Strategy

- Search tool unavailable → Mark all dimensions as 🟡 Needs Attention and proceed to Phase 2
- Empty search results → Retry with alternative keywords once; if still empty, mark as 🟢 Stable

---

## Phase 2: In-Depth Research

**Objective**: Conduct comprehensive in-depth research on dimensions marked as 🟡/🔴/🆕 in Phase 1, gathering sufficient evidence to support standards updates.

### Execution Steps

#### 2.1 Determine Research Scope

Read the list of dimensions requiring in-depth research from Phase 1 output. For each dimension:

| Level | Minimum Searches | Target Source Count |
|-------|-----------------|---------------------|
| 🟡 Needs Attention | 15 | 8+ independent sources |
| 🔴 Needs Update | 20 | 12+ independent sources |
| 🆕 New Dimension | 25 | 15+ independent sources |

#### 2.2 Search Strategy (Layered Search Method)

Allocate searches for each dimension across the following layers:

**Layer 1: Official Documentation (30%)**
```
site:tauri.app {dimension} guide
site:tauri.app {dimension} configuration
site:tauri.app {dimension} API reference
```

**Layer 2: API Documentation and Source Code (25%)**
```
site:docs.rs/tauri {dimension} {key type/function}
site:github.com/tauri-apps/tauri {dimension} (browse recent commits/releases)
repo:tauri-apps/tauri {dimension} "breaking change" OR "deprecated"
```

**Layer 3: Official Plugins and Ecosystem (20%)**
```
site:github.com/tauri-apps/plugins-workspace {dimension}
repo:tauri-apps/plugins-workspace {dimension} latest release
{dimension} tauri plugin official 2025
```

**Layer 4: Community Practices and Tutorials (15%)**
```
{dimension} tauri v2 tutorial 2025
{dimension} tauri v2 production best practice
site:dev.to tauri {dimension}
site:reddit.com/r/tauri {dimension}
```

**Layer 5: Issues and Discussions (10%)**
```
repo:tauri-apps/tauri is:issue {dimension} label:bug OR label:enhancement
repo:tauri-apps/tauri {dimension} "how to" OR "best way"
stackoverflow tauri v2 {dimension}
```

#### 2.3 New Dimension Discovery

If Phase 1 marked 🆕 new dimensions, additionally execute:

1. **Domain definition search**: `tauri v2 new feature area 2025` / `tauri v2 ecosystem emerging pattern`
2. **Competitor comparison**: `electron {new dimension} best practice` / `flutter desktop {new dimension}` (to understand general desktop application patterns)
3. **Official roadmap**: `site:tauri.app roadmap` / `repo:tauri-apps/tauri milestone`

#### 2.4 Information Extraction Template

Extract the following fields from each source:

```yaml
source:
  url: "https://..."
  type: "official_doc" | "api_doc" | "github_issue" | "community_blog" | "stack_overflow"
  date: "YYYY-MM-DD"  # Publication/last update time
  author: "..."       # Author or organization
  authority: "high" | "medium" | "low"  # Credibility: official=high, community=medium, individual=low
  relevance: "direct" | "indirect"      # Direct relevance to the dimension
  key_findings:
    - "Specific content discovered"
    - "Comparison differences with existing standards"
  related_principle: "X.Y"  # Related existing principle number, leave empty if none
```

#### 2.5 Evidence Organization

After research on each dimension is complete, generate `phase2_evidence_{dimension}.md`:

```markdown
# Dimension X: {Dimension Name} — In-Depth Research Evidence

## Change Evidence Summary
| ID | Type | Evidence Summary | Source | Authority |
|----|------|------------------|--------|-----------|
| E1 | API Change | ... | [Link] | high |

## Addition Candidate Principles
1. ... (evidence support)

## Deprecation Candidate Principles
1. ... (evidence support)

## Recommended Changes
1. ... (evidence support)
```

### Output

- `phase2_evidence_{dimension}.md` for each researched dimension
- `phase2_summary.md`: summary of evidence across all dimensions
- New dimension proposal (if applicable): `new_dimension_proposal.md`

### Decision Points

- Insufficient evidence (< 3 independent sources) → Mark as `[To Be Verified]`, do not directly change the standard, only record in the changelog
- Evidence conflict (different sources give contradictory recommendations) → Record in the Phase 3 conflict resolution list

---

## Phase 3: Cross-Validation

**Objective**: Compare Phase 2 findings with existing standards, mark all changes, and resolve conflicts.

### Execution Steps

#### 3.1 Change Classification

Classify each finding into one of the following four categories:

| Category | Condition | Tag |
|----------|-----------|-----|
| **Add Principle** | Does not exist in the standard, new pattern supported by >=2 authoritative sources | `ADD` |
| **Modify Principle** | Existing principle content needs modification, with clear evidence | `MODIFY` |
| **Deprecate Principle** | Existing principle is no longer applicable, API is deprecated or has a safer alternative | `DEPRECATE` |
| **Keep Unchanged** | No reliable evidence indicates a change is needed | `KEEP` |

#### 3.2 Conflict Resolution

When different sources give contradictory recommendations, adjudicate according to the following priority:

1. **Official documentation > Official blog > Core maintainer statements > Community tutorials > Stack Overflow answers**
2. **Recent sources (2025) > Older sources (2024)**
3. **Multiple independent sources in agreement > Single source**
4. **Verified with code example > Pure text description**

Conflict resolution record format:
```markdown
### Conflict X: {Brief Description}
- Viewpoint A: ... (source)
- Viewpoint B: ... (source)
- **Ruling**: Adopt viewpoint X, reasoning: ...
```

#### 3.3 Generate Change List

```markdown
# Phase 3 Change List

## Dimension 1: Architecture (3 changes)
### MODIFY 1.1
- Principle: lib.rs / main.rs architecture
- Change: Add mobile-specific directory structure recommendations
- Evidence: E3, E7
- Tag: **[Change: Added Mobile Directory]**

## Dimension 5: IPC (2 additions)
### ADD 5.6
- New principle: WebRTC / WebTransport usage in Tauri v2
- Evidence: E12, E15, E18 (3 sources)
- Tag: **[New]**
```

#### 3.4 Anti-Pattern Sync Check

Traverse the anti-pattern list to ensure:
- Each `ADD/MODIFY` principle has a corresponding anti-pattern entry (or is already covered)
- Each `DEPRECATE` principle's anti-pattern entry is also marked as deprecated
- New anti-pattern numbers continue from the existing maximum number

### Output

- `phase3_change_list.md`: complete change list, including classification, tags, and evidence references
- `phase3_conflict_resolution.md`: conflict resolution records (if any)
- Updated `change_tracking_table` (final status)

### Decision Points

- Total changes is 0 → Skip Phase 4, proceed to Phase 5 to verify existing standards
- More than 30 changes → Recommend splitting into multiple updates, process high-priority changes first

---

## Phase 4: Generate Updates

**Objective**: Based on the Phase 3 change list, output the updated complete AGENTS.md standards file.

### Execution Steps

#### 4.1 Edit Strategy

Apply changes in the following order:

1. **Structural changes**: Add/remove dimensions → Adjust table of contents
2. **Principle number adjustment**: Re-number sequentially after inserting new principles
3. **Content changes**: Update principle content according to the `MODIFY` list
4. **New principles**: Append new principles according to the `ADD` list
5. **Deprecation tags**: Mark deprecated content according to the `DEPRECATE` list
6. **Anti-pattern updates**: Sync update the anti-pattern list
7. **Glossary updates**: Supplement new terms
8. **Changelog append**: Append this update record at the end of the document

#### 4.2 Edit Specifications

**When modifying existing principles**:
- Preserve the original structure (heading levels, code block positions, table formats)
- Add `**[Change: Old→New]**` tag at the changed location
- If the change is large (exceeds 50% of the original content), add a `> **Note**: This principle was updated on YYYY-MM-DD, main changes: ...` callout at the beginning of the principle

**When adding new principles**:
- Follow the style of existing standards (tone, format, code example density)
- Include: title, description, code examples (correct vs. incorrect comparison), table (if applicable)
- Add `**[New]**` tag after the title
- Ensure logical connection with adjacent principles

**When deprecating principles**:
- Do not delete principle content (maintain backward compatibility reference)
- Add `**[Deprecated: v2.x]**` tag after the title
- Add `> **Warning**: This principle is deprecated, alternative: [link to new principle]` at the beginning of the principle
- Retain but annotate `// Deprecated, do not use in new projects` in the code example

#### 4.3 Completeness Check (Phase 4 Internal Check)

Before generating output and proceeding to Phase 5, self-check the following:

- [ ] All 12 dimension titles are correct and numbers are continuous
- [ ] Principle numbers within each dimension are continuous, with no gaps
- [ ] All `ADD/MODIFY` tagged principles have code examples
- [ ] Anti-pattern list numbers are continuous and correspond to principle references
- [ ] Glossary contains all newly introduced terms
- [ ] Changelog has been appended

### Output

- `AGENTS.md` (updated complete standards file)
- `phase4_edit_log.md`: edit log (for tracing the reason for each change)

### Rollback Strategy

- Discovered missing changes during editing → Return to Phase 3 to supplement
- New code example cannot be verified → Add `**[To Be Verified]**` tag

---

## Phase 5: Quality Check

**Objective**: Verify the quality of the updated standards item by item.

**The complete checklist is in `references/checklist.md`, only the workflow is described here**:

1. Read `references/checklist.md`
2. Check item by item according to the five categories (code examples, source references, completeness, consistency, progressive disclosure)
3. Record `Pass/Fail/N/A` for each check item
4. Output the final AGENTS.md after all items pass
5. If there are any failures, return to Phase 4 to fix and re-check

### Quality Check Execution Rules

| Check Category | Must All Pass | Allow Warnings |
|----------------|---------------|----------------|
| Code Examples | ✅ Yes | No |
| Source References | ✅ Yes | Link failures can be marked as historical references |
| Completeness | ✅ Yes | No |
| Consistency | ✅ Yes | No |
| Progressive Disclosure | ✅ Yes | No |

### Final Output

- `AGENTS.md` that has passed all checks
- `update_report.md`: update report summary (change statistics, key update points description)
