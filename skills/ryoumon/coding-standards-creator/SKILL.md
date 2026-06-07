---
name: coding-standards-creator
description: >
  Automatically research any technology stack and generate a complete coding standards skill
  (best-practices skill). Activate when user says "specify coding standards for xxx",
  "create best practices for xxx", "generate development standards for xxx", or similar.
  Covers research, standards creation, and skill packaging. Applies to any technology stack:
  frontend frameworks (React, Vue, Svelte), backend frameworks (Django, Spring, Gin),
  cross-platform frameworks (Flutter, React Native, Tauri), programming languages (Go, Rust, Python),
  toolchains (Docker, K8s, CI/CD), etc. Not for: updating existing skills (use corresponding updater skill),
  non-coding standards (documentation writing, design specs).
license: MIT
---

# Coding Standards Creator

A meta-skill that automatically researches any technology stack and generates a complete coding standards skill.

**Output Language Rule: ALL generated content MUST be in English. This includes SKILL.md, all reference files, code examples, comments, and documentation. No exceptions.**

## Workflow

Execute the following 6 stages. Do not skip stages.

### Stage 1: Parse User Requirements

Extract from user input:

1. **Target technology stack name** — Normalize naming (e.g. "React Native with Expo" → `expo-react-native-best-practices`)
2. **Skill name** — `{tech-stack}-best-practices` format
3. **Special requirements** — User-specified constraints (e.g. specific framework combinations, team size, etc.)

### Stage 2: Deep Research

Load `references/research-workflow.md` and execute research by dimension.

Core requirements:
- Cover at least 10 dimensions (Architecture, Coding Style, Type System, State Management, Components/Modules, Testing, Performance, Security, Error Handling, Tooling, Dependency Management, i18n/a11y)
- Perform 25+ searches covering official docs, GitHub, community best practices
- Record all sources

Research depth requirements:
- Find at least 3 authoritative sources per core dimension
- Every recommended tool/library must include the latest stable version number
- Every architectural decision must include comparisons (e.g. state management options comparison)
- Collect at least 5 complete code examples for patterns.md

### Stage 3: Design Standards Structure

Load `references/output-template.md` and design based on research:

1. Directory structure specification
2. Coding principles (5-10 per dimension, 40+ total)
3. Gotchas (10+ items)
4. Code templates (5+ complete runnable examples)
5. Anti-patterns list (10+ items with symptoms/consequences/fixes)
6. Reference file list

### Stage 4: Generate Skill Files

Generate in this order:

1. `references/sources.md` — All citations (Tier 1-3 sources prioritized)
2. `references/guidelines.md` — Complete coding principles (all dimensions, each with rationale + implementation + code example + anti-pattern reference)
3. `references/architecture.md` — Directory structure details, layered patterns, module dependency rules
4. `references/patterns.md` — 5+ design pattern code templates
5. `references/anti-patterns.md` — 10+ anti-patterns (symptoms/consequences/fixes/correct principle reference)
6. Additional dimension references as needed (testing.md, performance.md, security-and-errors.md, tooling.md, etc.)
7. `SKILL.md` — Main file (core principles精华 + Gotchas + quick decision table + reference index)

**SKILL.md Design Requirements** (under 500 lines):
- Must include "Quick Decision Table": guides when to load which reference
- Must include "Architecture Cheat Sheet": directory structure + naming convention quick reference
- Must include "Essential Principles": brief version of 20% core rules
- Must include "Gotchas": most common pitfalls
- Must include 2-3 key code templates
- Must include reference file index

### Stage 5: Validate

- [ ] SKILL.md is under 500 lines
- [ ] SKILL.md contains quick decision table, architecture cheat sheet, Gotchas
- [ ] All reference files are readable and well-structured
- [ ] guidelines.md contains 40+ actionable principles, each with rationale + code example
- [ ] anti-patterns.md contains 10+ anti-patterns with cross-references to principles
- [ ] patterns.md contains 5+ complete code templates
- [ ] sources.md contains all citations
- [ ] Code examples are syntactically correct
- [ ] No broken internal references
- [ ] Principles are actionable ("Use X instead of Y", not "Consider using X")
- [ ] All tools/frameworks have version numbers
- [ ] **ALL content is in English**

### Stage 6: Package

Run the packaging script:

```bash
python3 /app/.agents/skills/skill-creator-swarm/scripts/package_skill.py /mnt/agents/output/{skill-name} /mnt/agents/output/
```

Deliver the `.skill` file to the user with installation instructions.

## Research Dimensions Quick Reference

Must-cover dimensions (adjust per technology stack, at least 10):

| # | Dimension | Content | Reference File |
|---|-----------|---------|---------------|
| 1 | Architecture | Directory structure, layered patterns, module organization | `references/architecture.md` |
| 2 | Coding Style | Naming conventions, formatting tools, lint rules | `references/guidelines.md` |
| 3 | Type System | Type safety, interface design, generic usage | `references/guidelines.md` |
| 4 | State Management | State design, data flow, persistence | `references/guidelines.md` |
| 5 | Components/Modules | Component patterns, reuse strategies, interface design | `references/guidelines.md` + `references/patterns.md` |
| 6 | Testing | Unit tests, integration tests, E2E, mocking strategies | `references/testing.md` |
| 7 | Performance | Rendering optimization, memory management, async strategies | `references/performance.md` |
| 8 | Security | Input validation, access control, data protection | `references/security-and-errors.md` |
| 9 | Error Handling | Error types, recovery strategies, logging | `references/security-and-errors.md` |
| 10 | Tooling | Build tools, dev environment, CI/CD | `references/tooling.md` |
| 11 | Dependency Management | Package selection, versioning strategy, update workflow | `references/guidelines.md` or `references/tooling.md` |
| 12 | i18n/a11y | Internationalization, accessibility, localization (if applicable) | `references/guidelines.md` |

**Required reference files**: `guidelines.md`, `architecture.md`, `anti-patterns.md`, `sources.md`
**Recommended reference files**: 8+ files, one per core dimension

## Naming Conventions

File naming for generated skills:

| Item | Convention |
|------|-----------|
| Skill directory | `{tech-stack}-best-practices`, lowercase, hyphen-separated |
| Skill name | Same as directory name |
| Reference files | Lowercase, hyphen-separated, `.md` suffix |
| Code examples | Language-annotated, standard file naming |

## Source Evaluation Criteria

| Tier | Source Type | Weight |
|------|-------------|--------|
| 1 | Official documentation | Highest |
| 2 | GitHub release notes / RFCs | High |
| 3 | Core maintainer blogs / talks | Medium-High |
| 4 | Established community blogs | Medium |
| 5 | Individual developer blogs | Low — cross-verify with official docs |

## Key Constraints

1. **Do not assume** — Each technology stack's uniqueness must be confirmed through research
2. **Official first** — Official recommendations > community practices > personal experience
3. **Actionable specifics** — Principles must be "Use X instead of Y", not "Consider using X"
4. **Version labels** — All tools/frameworks must include version numbers or time ranges
5. **Distinguish stable vs experimental** — Clearly separate stable recommendations from experimental features
6. **English only** — ALL output must be in English, including code comments and documentation

## Gotchas

- A technology stack's official name may differ from user input (e.g. "RN" → "React Native"). Confirm official naming through search.
- Some stacks have multiple mainstream usage patterns (e.g. React with CRA/Vite/Next.js). Research and provide a recommendation.
- Skill files must be validated for syntax correctness after generation.
- When there are too many reference files, prioritize depth for core dimensions over breadth.
- The packaging script path may vary by environment. Confirm `/app/.agents/skills/skill-creator-swarm/scripts/package_skill.py` exists.
