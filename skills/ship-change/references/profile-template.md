# `.claude/ship-change.md` template

Copy into the project as `.claude/ship-change.md` and fill in what applies; delete what doesn't. Keep it short — it's read at the start of every run. Project instructions (CLAUDE.md, AGENTS.md) still win over it; link to them rather than repeating them.

```markdown
# ship-change profile: <project name>

## Commands
- Format: `<cmd>`            (fix in place)
- Format check: `<cmd>`      (fails if anything needs formatting)
- Lint: `<cmd>`
- Build: `<cmd>`
- Test: `<cmd>`
- Race / integration / e2e: `<cmd>` (when to run it)
- Vulnerability / audit: `<cmd>`
- Dependency tidy + check: `<cmd>` / `<cmd>`
- Codegen: `<cmd>` (never hand-edit: <paths>)
- Done gate: `<cmd>`         (what CI runs before merge)

## Requirements
- Source: <spec/plan/issue tracker paths or URLs>
- IDs: <scheme, e.g. FR-123, JIRA-456>; cite them in test names and commits
- "Next item": <how to pick it, e.g. lowest todo in docs/PLAN.md whose deps are done>
- Mark done: <how, e.g. set Status to `done (<sha>)`>

## Sensitive areas
- <paths or components that always get a full security review>

## Threat model notes
- <what an attacker could reach: public endpoints, local sockets, privileged helpers, user data>

## Branching and merging
- Base: <branch>; branch names: <pattern>
- Merge: <merge commit / squash / PR only>; delete branch after: <yes/no>
- Never without asking: <push, PRs, merges to main, releases, deploys>

## Commits
- Convention: <e.g. Conventional Commits, `<area>: <summary> (<ID>)`>

## User-facing changes
- Record in: <CHANGELOG.md / changesets / releases.json>, following <doc>
- Validate with: `<cmd>`

## Docs to keep in sync
- <spec sections, READMEs, API docs>

## Test conventions
- <frameworks, helpers, fixtures, leak checks, naming>

## Safety rules
- Tests and agents must never touch: <real services, login items, keychains, accounts, devices>
- Use instead: <seams, fakes, temp homes, test accounts>

## Dependencies
- Allowed without asking: <list or "none">
```
