# Output Format Template

The generated coding standards skill must follow this structure. Adjust per technology stack characteristics.

**Language Rule: ALL content must be written in English — SKILL.md, references, code examples, comments, and documentation.**

## Directory Structure

```
{skill-name}/
├── SKILL.md                    # Main file: core principles + cheat sheets + Gotchas
├── references/
│   ├── guidelines.md           # Complete coding principles (all dimensions)
│   ├── architecture.md         # Directory structure, layered patterns, module organization
│   ├── patterns.md             # Design patterns & code templates
│   ├── anti-patterns.md        # Anti-patterns list (common mistakes)
│   ├── testing.md              # Testing strategy & tools
│   ├── performance.md          # Performance optimization guide
│   ├── security-and-errors.md  # Security & error handling
│   ├── tooling.md              # Toolchain configuration
│   └── sources.md              # Citation index
```

**Minimum reference files**: `guidelines.md`, `architecture.md`, `sources.md`
**Recommended**: 8+ reference files, one per dimension

## SKILL.md Template

```markdown
---
name: {skill-name}
description: >
  Coding standards and best practices for {tech-stack}. Covers {dimension-list} and {N} dimensions.
  Provides {N}+ actionable principles, {N} design patterns, {N} anti-patterns, and complete code templates.
  Activate when working with {tech-stack} projects: {typical use cases}.
  Not for {excluded scenarios}.
---

# {Tech Stack} Best Practices

## Quick Decision: When to Load Which Reference

| Scenario | Load | Content |
|----------|------|---------|
| Starting a new project / designing module structure | **This file** Architecture Cheat Sheet | Directory structure, layering rules |
| Need complete coding principles | `references/guidelines.md` | Complete guide for all dimensions |
| Implementing specific design pattern code | `references/patterns.md` | Runnable templates |
| Code review / avoiding pitfalls | `references/anti-patterns.md` | Anti-patterns list |
| Verifying principle sources | `references/sources.md` | Citation index |

**Core principle:** Read SKILL.md first to establish a global view; load references on demand. Do not read all references at once.

---

## Architecture Cheat Sheet

### Directory Structure (Mandatory)

```
{directory structure tree}
```

**Key rule:** {most important architectural constraint}

### Layered Pattern

- **{Layer A}** (`{directory}/`): {responsibility description}
- **{Layer B}** (`{directory}/`): {responsibility description}

### Naming Conventions

| Item | {Language A} Side | {Language B} Side | Note |
|------|-------------------|-------------------|------|
| {item type} | `{naming style}` | `{naming style}` | {note} |

---

## Essential Principles (20% Core Rules)

> Full {N}+ principles in `references/guidelines.md`

### {Dimension A}

1. **{Principle 1}** — {brief explanation}
2. **{Principle 2}** — {brief explanation}

### {Dimension B}

3. **{Principle 3}** — {brief explanation}
4. **{Principle 4}** — {brief explanation}

---

## Gotchas (Most Common Pitfalls)

1. **{Common mistake 1}** → {consequence}. {Correct approach}.
2. **{Common mistake 2}** → {consequence}. {Correct approach}.

---

## Code Templates

### Template 1: {Description}

```{language}
// Complete runnable code example
```

## Reference Files

- `references/guidelines.md` — Complete coding principles
- `references/architecture.md` — Architecture deep dive
- `references/patterns.md` — Design patterns & code templates
- `references/anti-patterns.md` — Anti-patterns list
- `references/testing.md` — Testing strategy
- `references/performance.md` — Performance optimization
- `references/security-and-errors.md` — Security & error handling
- `references/tooling.md` — Toolchain configuration
- `references/sources.md` — Citations

## Sources

All principles are derived from authoritative sources. See `references/sources.md` for details.
```

## Reference File Templates

### guidelines.md

Organized by dimension. Each dimension contains:

```markdown
## {Dimension Name}

### {N}.{M} {Principle Title}

**Principle**: {one-sentence description}

**Rationale**: {why this principle matters}

**Implementation**:
- {specific practice 1}
- {specific practice 2}

**Code Example**:
```{language}
// Correct example
// Incorrect example (annotated)
```

**Anti-pattern**: {reference to anti-patterns.md item number}
```

### architecture.md

Contains:
- Directory structure detailed explanation
- Responsibilities of each directory
- File naming conventions
- Module dependency rules
- Cross-layer communication rules

### patterns.md

Each pattern contains:
- Pattern name and applicable scenarios
- UML/structural description
- Complete code implementation
- Usage example

### anti-patterns.md

Each anti-pattern contains:
- Anti-pattern name
- Symptoms (how to identify)
- Consequences
- Fix approach
- Reference to correct principle number

### testing.md

- Testing pyramid explanation
- Testing frameworks and tools per layer
- Test file organization
- Mock/Stub strategies
- Coverage targets

### performance.md

- Performance metrics and tools
- Common optimization patterns
- Memory management
- Async/concurrency optimization
- Monitoring and alerting

### security-and-errors.md

- Security checklist
- Error handling patterns
- Input validation rules
- Sensitive data handling
- Logging standards

### tooling.md

- Development environment configuration
- Lint/Format configuration
- Build configuration
- CI/CD templates
- Debugging tools

### sources.md

```markdown
## Sources

### Official Documentation

- [{Doc Name}]({URL}) — {version/date}

### GitHub

- [{Repo Name}]({URL}) — {version/date}

### Community Articles

- [{Article Title}]({URL}) — {author} — {date}

### Talks/Videos

- [{Title}]({URL}) — {speaker} — {date}
```

## Quality Standards

The generated skill must satisfy:

### Hard Metrics
- [ ] At least 10 dimensions of coding principles
- [ ] At least 40 actionable principles (each with rationale + implementation + code example + anti-pattern reference)
- [ ] At least 10 Gotchas (with symptoms/consequences/correct approach)
- [ ] At least 5 complete code templates (runnable, with comments)
- [ ] At least 10 anti-patterns (with symptoms/consequences/fixes/correct principle number)
- [ ] 25+ research search records
- [ ] SKILL.md under 500 lines

### Structural Requirements
- [ ] SKILL.md contains "Quick Decision Table"
- [ ] SKILL.md contains "Architecture Cheat Sheet" (directory structure + naming conventions)
- [ ] SKILL.md contains "Essential Principles" (brief version of 20% core rules)
- [ ] SKILL.md contains 2-3 key code templates
- [ ] Anti-patterns cross-referenced with principles (Principle X.Y ↔ Anti-pattern Z)
- [ ] sources.md contains all external sources (prioritize Tier 1-3)
- [ ] All tools/frameworks have version numbers or time ranges

### Quality Requirements
- [ ] All principles have source citations
- [ ] Principles are actionable ("Use X instead of Y", not "Consider using X")
- [ ] Code examples are syntactically correct
- [ ] No broken internal references
- [ ] Reference files are well-structured
- [ ] **ALL content is in English**
