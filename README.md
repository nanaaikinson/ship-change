# ship-change

A Claude Code plugin that delivers a code change end to end through a gated pipeline:

1. **Intake**: read the requirement and the project's conventions.
2. **Build**: write the change.
3. **Tests**: `ship-test-writer` writes tests from the requirement, independent of how the code was built.
4. **Code review + fix**: `ship-code-reviewer` returns severity-ranked findings; you triage and fix.
5. **Security review + fix**: `ship-security-reviewer` checks the diff against a security checklist.
6. **Finish**: final report with review tables and a "Not verified" list.

Tests and reviews go to fresh-context subagents so they check what the code *does*, not what its author *meant*. Works in any repository and language.

## Pipeline

```mermaid
flowchart TD
    A([Request]) --> P0["Phase 0 · Intake<br/>requirement → numbered criteria R1…Rn<br/>size, sensitive areas, branch"]
    P0 --> SZ{Size?}
    SZ -- Docs --> D["Edit directly<br/>(no pipeline)"]
    SZ -- Small --> P1
    SZ -- Standard --> P1
    P1["Phase 1 · Build<br/>gate: format + lint + build"] --> P2
    P2["Phase 2 · Tests<br/>ship-test-writer<br/>gate: tests pass, key tests can fail"] --> KS{Size?}
    KS -- Small --> CR1["One combined review<br/>ship-code-reviewer<br/>code + security checklists"]
    KS -- Standard --> P3
    P3["Phase 3 · Code review<br/>ship-code-reviewer"] --> F3{Open blocker<br/>or major?}
    F3 -- yes --> X3["Fix, re-run gates,<br/>re-review (max 2 rounds)"] --> P3
    F3 -- no --> P4
    P4["Phase 4 · Security review<br/>ship-security-reviewer"] --> F4{Open critical<br/>or high?}
    F4 -- yes --> X4["Fix + regression test,<br/>re-review"] --> P4
    F4 -- no --> P5
    CR1 --> P5
    P5["Phase 5 · Finish<br/>release notes, docs, full check, report"] --> Z([Done])
    D --> Z
```

Commits are always the user's call: the skill asks at each commit point and never pushes, opens PRs, merges or deploys unasked.

## Contents

```
.claude-plugin/        plugin.json, marketplace.json
skills/ship-change/    SKILL.md + references/ (pipeline, checklists, test guide, profile template)
agents/                ship-test-writer, ship-code-reviewer, ship-security-reviewer
```

## Install

From a local checkout:

```
/plugin marketplace add /path/to/ship-change
/plugin install ship-change@ship-change
```

From GitHub:

```
/plugin marketplace add nanaaikinson/ship-change
/plugin install ship-change@ship-change
```

Manual install without the plugin system: copy `skills/ship-change` to `~/.claude/skills/` and `agents/*.md` to `~/.claude/agents/`.

## Usage

Ask Claude to implement, fix or refactor something and have it tested and reviewed: "ship this", "implement FR-12", "fix this properly". The skill triggers on its own.

## Per-project profile

The skill adapts to your repo's `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING` and build files. For tighter control, add `.claude/ship-change.md` (commands, requirement sources, sensitive areas, branching, safety rules). Template: `skills/ship-change/references/profile-template.md`. After the first run the skill offers to generate one.

## License

MIT
