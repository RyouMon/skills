---
name: expo-best-practices-updater
description: >
  Update and refresh the expo-react-native-best-practices skill by researching
  the latest changes in the React Native / Expo ecosystem. Use when (1) a user
  explicitly asks to update the expo-react-native-best-practices skill,
  (2) a major Expo SDK release drops (e.g., SDK 53, 54), (3) a new React Native
  architecture change is announced, (4) a key dependency (NativeWind, Expo Router,
  Zustand, Reanimated) has a significant version change, or (5) current principles
  appear outdated or conflict with the latest best practices. Performs systematic
  research, evaluates new information against existing principles, and produces
  updated skill files.
license: MIT
---

# Expo Best Practices Updater

Systematic workflow to research and update the `expo-react-native-best-practices` skill.

## Update Workflow

Follow these steps in order. Do not skip steps unless there is a clear reason.

### Step 1: Read Current Skill Content

Read the existing skill at `/mnt/agents/output/expo-react-native-best-practices/` (or the path provided by the user):

1. Read `SKILL.md` — note the current principles and gotchas.
2. Read all files in `references/` — note the current detailed guidance.
3. Read `references/sources.md` — note the current citation dates and sources.
4. Summarize: what does the skill currently cover? What are the gaps or outdated items?

### Step 2: Research Latest Changes

Search for updates across these dimensions. See `references/research-checklist.md` for the full search query template.

Key research areas:

1. **Expo SDK releases**: Latest stable version, new features, breaking changes, deprecations.
2. **React Native releases**: New Architecture status, core API changes, performance improvements.
3. **Key dependencies**: NativeWind, Expo Router, Zustand, TanStack Query, Reanimated, React Native Testing Library.
4. **Community best practices**: Notable blog posts, conference talks, ecosystem shifts (e.g., new state management library gaining traction).
5. **TypeScript**: New language features relevant to React Native, strict mode evolution.

Use web search to find authoritative sources. Prioritize:
- Official documentation (expo.dev, reactnative.dev, nativewind.dev)
- GitHub release notes (expo/expo, facebook/react-native)
- Well-known community authors and organizations (Saritasa, Callstack, Infinite Red, Expo team blog)

### Step 3: Evaluate & Synthesize

For each piece of new information:

1. **Is it authoritative?** Favor official docs over blog posts. Favor actively maintained projects.
2. **Is it actionable?** Does it change a concrete coding decision (e.g., "use X instead of Y")?
3. **Is it stable?** Avoid recommending alpha/beta features unless explicitly requested.
4. **Does it replace an existing principle?** Mark conflicts for replacement.
5. **Does it fill a gap?** Note new areas the skill doesn't currently cover.

Produce a summary document with three sections:
- **Replace**: Principles that should be updated or replaced.
- **Add**: New principles or sections to add.
- **Remove**: Principles that are no longer relevant.

### Step 4: Update Skill Files

Update files in this order:

1. **`references/sources.md`**: Add new sources, update version numbers, mark deprecated sources.
2. **Individual `references/*.md` files**: Update the detailed guidance. Keep the same structure — do not reorganize unless the architecture itself changes.
3. **`SKILL.md`**: Update core principles, gotchas, and reference file loading guidance. Keep it under 500 lines.

Rules when editing:
- Preserve the existing file structure and section organization.
- Use imperative/infinitive form in all instructions.
- Keep code examples concise and realistic.
- Update version references (e.g., "Expo SDK 52+" → "Expo SDK 53+").
- Add new gotchas when a common mistake is discovered.

### Step 5: Validate

Before packaging, verify:

- [ ] All `references/*.md` files are readable and well-structured.
- [ ] `SKILL.md` is under 500 lines.
- [ ] All code examples are syntactically valid.
- [ ] No broken internal references.
- [ ] `sources.md` cites every external source used.
- [ ] No `any` types in TypeScript examples.

### Step 6: Package

Run the packaging script to produce the updated `.skill` file:

```bash
python3 /app/.agents/skills/skill-creator/scripts/package_skill.py /mnt/agents/output/expo-react-native-best-practices /mnt/agents/output/
```

## Source Evaluation Criteria

When deciding whether to include a source:

| Tier | Source Type | Weight |
|------|-------------|--------|
| 1 | Official docs (expo.dev, reactnative.dev) | Highest |
| 2 | GitHub release notes / RFCs | High |
| 3 | Recognized community authors (Expo team, Callstack, Infinite Red) | Medium-High |
| 4 | Established dev blogs (Saritasa, S-PRO, Moquaya) | Medium |
| 5 | Individual developer blogs | Low — cross-verify with official docs |

## Gotchas

- Expo SDK releases are ~quarterly. Check https://expo.dev/changelog for the latest.
- NativeWind v4 introduced significant API changes from v2. Verify the version referenced.
- The "New Architecture" (Fabric + TurboModules) is gradually becoming default — monitor its status.
- A library becoming popular does not mean it should replace an existing recommendation. Evaluate stability and ecosystem maturity.
- When updating version numbers, check ALL references — they may appear in multiple files.
- Do not remove existing principles unless they are demonstrably outdated. Add new principles alongside existing ones when both remain valid.
