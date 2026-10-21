---
name: ship-change
description: Delivers a code change end to end through a gated pipeline — intake, build, independent tests, code review + fix, security review + fix, finish — handing tests and reviews to fresh-context subagents (ship-test-writer, ship-code-reviewer, ship-security-reviewer). Works in any repository and language; it adapts to the project's own conventions (CLAUDE.md, AGENTS.md, CONTRIBUTING, build files) or a `.claude/ship-change.md` profile. Use it whenever you're asked to implement, build, add, fix, refactor or change code and the work should be tested and reviewed before it lands — "ship this", "build and review", "implement FR-12", "do the next ticket/package", "fix this properly" — even if the user doesn't mention tests or reviews. Not for docs-only edits, questions about code, or throwaway prototypes.
---

# Ship a change

You are the orchestrator and the builder. You keep the context of the whole change, write the code and apply every fix. Tests and reviews go to subagents because they need a fresh view: whoever wrote the code tends to test and review what they *meant*, not what they *wrote*. Splitting the roles is the point of this skill; keep it even when doing everything yourself would be faster.

Track the phases in your task list so the user can follow along:
**Intake → Build → Tests → Code review + fix → Security review + fix → Finish.**

Reference files (read when the phase needs them, not all up front):

| File | Read it for |
| --- | --- |
| `references/pipeline.md` | Change sizes, phase gates, findings format, severities, fix loop, final report template |
| `references/discovery.md` | Building the project profile when there isn't one |
| `references/profile-template.md` | The profile format, to save one for the project |
| `references/code-review-checklist.md` | Passed to the code reviewer |
| `references/security-checklist.md` | Passed to the security reviewer |
| `references/test-guide.md` | Passed to the test writer |

Paths are relative to this skill's base directory; give subagents absolute paths.

## Step 0 — Load the project profile

Every project does builds, tests, branches and release notes differently. Learn this project's way before touching code:

1. **Project instructions win.** Read CLAUDE.md / AGENTS.md / CONTRIBUTING and anything they point to. If the project has its own delivery workflow document, follow it and use this skill to fill gaps.
2. **Profile.** If `.claude/ship-change.md` exists, read it: it says the commands, requirement sources, sensitive areas, branching, release-notes process and safety rules for this repo.
3. **No profile?** Build one from the repo using `references/discovery.md`. Note what you couldn't determine. At the end of the first run, offer once to save it as `.claude/ship-change.md` (using `references/profile-template.md`) so later runs start faster. Don't write it without asking — it's a file in their repo.

The profile gives you, at minimum: the format, lint, build, test and "done" gate commands; where requirements live; the sensitive areas; the base branch and branch naming; and how user-facing changes are recorded. Missing pieces are fine — say so in the report rather than guessing at commands that might do something unexpected.

## Phase 0 — Intake

1. **Find the requirement.** A ticket, spec row, plan item, issue or the user's words. If the project has a build plan or backlog and the user says "next", take the next item whose dependencies are done. Read the requirement and the sections it references.
2. **Close open questions before building.** If the request conflicts with the spec, leaves product behaviour open, or depends on a TBD, ask — guessing at behaviour costs more than a question. Ask with concrete options and a recommended default (AskUserQuestion when available), batch related questions, and keep building anything that doesn't depend on the answer.
3. **Size it** (Docs / Small / Standard, see `references/pipeline.md`) and list the sensitive areas it touches. Docs-only: stop using this skill and just make the edit.
4. **Branch.** Follow the project's branching rules. Look for an existing branch for this work first and continue it; otherwise branch from an up-to-date base. Note the base for reviewers.
5. **Tell the user** in two or three lines: requirement, size, plan, and anything you decided on their behalf.

## Phase 1 — Build

Write the code the way the surrounding code is written: naming, error handling, comment density, structure. Keep it testable — inject clocks, file systems, network listeners, process runners and anything that touches real system state, so tests can replace them.

- **Dependencies:** prefer what the project already uses or lists; ask before adding new ones unless the profile allows it. After adding one, run the project's tidy/lock step and its check that the lock files are clean — dependency drift is easy to miss when the workspace build still passes.
- **Generated code:** regenerate rather than hand-edit.
- **Gate:** the format, lint and build commands pass. Commit using the project's commit conventions, citing the requirement ID.

## Phase 2 — Tests (ship-test-writer)

Spawn `ship-test-writer` with: the requirement text (paste it), the changed files, the public interfaces to test (signatures, endpoints, commands, RPCs), the base commit, the profile's test commands and safety rules, and the absolute path to `references/test-guide.md`. Give it the requirement and the interfaces, **not** your implementation reasoning — the point is tests that check the spec rather than mirror the code.

- **Run it alongside Phase 3** when its work only adds test files: start the code reviewer at the same time on the build commit. Security review still waits for Phase 3's fixes.
- **Bugs found** → fix the production code yourself, then ask the same agent (SendMessage, so it keeps context) to re-run.
- **You changed production code under it** (review fixes) → tell it what changed so it adapts instead of fighting stale assumptions.
- **Gaps** → fill them (send it back) or record them for the report.
- **Gate:** the project's test command passes, plus the race/integration variant for changed packages where the project has one. Commit the tests.

## Phase 3 — Code review + fix (ship-code-reviewer)

Spawn `ship-code-reviewer` with the base, the commit(s) to review, the requirement text, the profile's conventions and the absolute path to `references/code-review-checklist.md`. Nothing about how you built it.

Then run the fix loop from `references/pipeline.md`: triage every finding (fixed / won't fix with evidence / deferred), push back on findings you believe are wrong, fix, re-run the Phase 1–2 gates, commit (`fix(<area>): address review`), and send the reviewer only the open findings with your dispositions and the new diff range. Stop after two re-review rounds; if blockers or majors remain, ask the user.

## Phase 4 — Security review + fix (ship-security-reviewer)

Spawn `ship-security-reviewer` after Phase 3's fixes land, so it reviews what will merge. Give it the base, the commit range, the requirement, the sensitive areas touched, the profile's threat-model notes and the absolute path to `references/security-checklist.md`.

Same fix loop, with these differences: every `critical` or `high` fix gets a regression test (write it or send it to the test writer); a `medium` stays open only with the user's explicit approval as an accepted risk — ask, don't assume; commit as `fix(security): <summary>`. Cheap `low` fixes are usually worth taking.

## Small changes: one combined review

For Small changes, spawn `ship-code-reviewer` once and tell it to apply **both** checklists. If it finds the change touches a sensitive area after all, switch to the Standard path and run Phase 4 properly.

## Phase 5 — Finish

1. **User-facing record:** changelog, release notes or whatever the profile names, written for users.
2. **Docs and spec:** update them where behaviour now differs — new flags, limits, exit codes, defaults.
3. **Done gate:** run the project's full check. If it runs a vulnerability or audit step separately, run that too.
4. **Plan or ticket:** mark the item done the way the project does.
5. **Report** with the template in `references/pipeline.md`, including review tables with dispositions and a "Not verified" list (platforms you couldn't build, real integrations you deliberately didn't exercise).
6. **Merge and clean up** per the branching rules. Don't push, open PRs, merge to release branches or deploy unless the user asked — offer it.

## Working with subagents

These habits keep parallel work from going wrong:

- **Stable snapshots for reviewers.** When anyone else may edit the working tree, tell reviewers to review a commit from a snapshot (`git archive <sha> | tar -x -C <scratch>`), not the live tree.
- **Never lose others' work.** Agents restore mutation checks with an editor, never `git checkout`/`git stash`/`git reset`, and you commit only your own files while others have work in progress (`git add <paths>`, not `git add -A`). If you must look at a file's committed version, use `git show <sha>:<path>`.
- **Agent reports are claims.** Check the commit, the diff and the test run before telling the user something is done.
- **Interrupted agents** (rate limits, errors): resume the same agent with SendMessage instead of starting a new one; it keeps its context.
- **Missing agent types** (other tools, older setups): spawn a general-purpose agent per phase whose prompt starts "Read `<abs path>/agents/<name>.md` and act as that agent" — the agent files live in the plugin's `agents/` directory (or `~/.claude/agents/` for a manual install). With no subagents at all, do each phase yourself, re-reading the diff and checklist from scratch for each review, and say in the report that reviews weren't independent.

## Safety rules

- **Nothing touches real user or system state** — login items and services, keychains, trust stores, shell profiles, global config, real accounts, production data — not the code under test, not the tests, not the agents. Tests use seams, temporary homes and fake binaries on `PATH`. Libraries sometimes ignore `$HOME` (for example, reading the home directory from the OS user database), so check where a library actually writes before trusting a temp `HOME`. If anything real was touched, put it back and tell the user plainly.
- **Outward-facing actions need the user's OK:** pushing, publishing, opening PRs, sending messages, deploying.
- **Stay in scope.** Real problems found along the way that aren't part of this change go into the report as follow-ups (or a separate session), not into this diff.
