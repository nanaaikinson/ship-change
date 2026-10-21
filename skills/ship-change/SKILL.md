---
name: ship-change
description: >-
  Delivers a code change end to end through a gated pipeline — intake, build, independent tests, code review + fix, security review + fix, finish — handing tests and reviews to fresh-context subagents (ship-test-writer, ship-code-reviewer, ship-security-reviewer). Works in any repository and language; it adapts to the project's own conventions (CLAUDE.md, AGENTS.md, CONTRIBUTING, build files) or a `.claude/ship-change.md` profile. It needs no spec or ticket: the user can simply describe what to build, fix or change. Use it whenever you're asked to implement, build, add, fix, refactor or change code and the work should be tested and reviewed before it lands — "ship this", "build and review", "add a retry to the uploader", "implement FR-12", "do the next ticket/package", "fix this properly" — even if the user doesn't mention tests or reviews. Not for docs-only edits, questions about code, or throwaway prototypes.
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

## Commit policy — ask first

Nothing is committed without the user's say-so. At every commit point below (build, tests, review fixes, security fixes), stop and ask once with AskUserQuestion, naming what would be committed (files, proposed message). Offer three answers:

1. **Commit this** — commit now; ask again at the next commit point.
2. **Commit this and auto-commit the rest of this run** — the opt-in. Every later commit point in this run commits without asking, using the same message conventions. It lasts for this run only: never save it to the profile or carry it into the next run, and never let it cover pushing, PRs, merging or deploying.
3. **Don't commit** — leave the changes in the working tree and carry on. Ask again at the next commit point.

Every time you ask, include the opt-in option, even if the user declined earlier. If AskUserQuestion isn't available, ask in plain text with the same three choices and wait for the answer.

Subagents never commit unless you tell them a commit is approved and give them the exact message; pass that only after the user said yes (or opted in). The test writer stages named paths only.

When changes are uncommitted, reviewers can't be given a commit range. Copy the working tree to a scratch directory and give them that snapshot plus the base commit to diff against, as in "Working with subagents".

## Phase 0 — Intake

1. **Find the requirement.** A spec is not required: the user describing what to build or fix in their own words is a full requirement. Otherwise use the ticket, spec row, plan item or issue. If the project has a build plan or backlog and the user says "next", take the next item whose dependencies are done. Read the requirement and the sections it references. For a bug, find or write the reproduction (inputs, actual result, expected result) and, where you can, the cause.
2. **Write the criteria.** Turn the requirement into 2–5 short, testable statements, numbered `R1`, `R2`, … unless the project has its own IDs (then use those). Each says what must be true when the change is done, including the error or limit cases the request implies; for a bug, `R1` is "the reproduction no longer fails" plus what must keep working. These criteria are what the test writer and both reviewers receive, so they must stand without your reasoning. Don't invent product behaviour: if the user's words leave a decision open, that is a question (next step), not a criterion.
3. **Close open questions before building.** If the request conflicts with the spec, leaves product behaviour open, or depends on a TBD, ask — guessing at behaviour costs more than a question. Ask with concrete options and a recommended default (AskUserQuestion when available), batch related questions, and keep building anything that doesn't depend on the answer. For a request that is clear, show the criteria in step 6 instead of asking about them.
4. **Size it** (Docs / Small / Standard, see `references/pipeline.md`) and list the sensitive areas it touches. Docs-only: stop using this skill and just make the edit.
5. **Branch.** Follow the project's branching rules. Look for an existing branch for this work first and continue it; otherwise branch from an up-to-date base. Note the base for reviewers.
6. **Tell the user** in a few lines: the criteria, size, plan, and anything you decided on their behalf.

## Phase 1 — Build

Write the code the way the surrounding code is written: naming, error handling, comment density, structure. Keep it testable — inject clocks, file systems, network listeners, process runners and anything that touches real system state, so tests can replace them.

- **Dependencies:** prefer what the project already uses or lists; ask before adding new ones unless the profile allows it. After adding one, run the project's tidy/lock step and its check that the lock files are clean — dependency drift is easy to miss when the workspace build still passes.
- **Generated code:** regenerate rather than hand-edit.
- **Gate:** the format, lint and build commands pass. Then commit per the commit policy (ask first), using the project's commit conventions and citing the requirement ID when there is one.

## Phase 2 — Tests (ship-test-writer)

Spawn `ship-test-writer` with: the requirement text and the numbered criteria from Phase 0 (paste both), the changed files, the public interfaces to test (signatures, endpoints, commands, RPCs), the base commit, the profile's test commands and safety rules, and the absolute path to `references/test-guide.md`. Give it the requirement and the interfaces, **not** your implementation reasoning — the point is tests that check the spec rather than mirror the code.

- **Bug fix:** tell it the reproduction. At least one test must reproduce the bug, and it has to fail on the base commit (check against a `git archive` snapshot or a worktree, never by reverting your fix in the working tree) and pass on the fix.
- **Run it alongside Phase 3** when its work only adds test files: start the code reviewer at the same time on the build commit. Security review still waits for Phase 3's fixes.
- **Bugs found** → fix the production code yourself, then ask the same agent (SendMessage or equivalent, so it keeps context) to re-run.
- **You changed production code under it** (review fixes) → tell it what changed so it adapts instead of fighting stale assumptions.
- **Gaps** → fill them (send it back) or record them for the report.
- **Gate:** the project's test command passes, plus the race/integration variant for changed packages where the project has one. Commit the tests per the commit policy.

## Phase 3 — Code review + fix (ship-code-reviewer)

Spawn `ship-code-reviewer` with the base, the commit(s) to review, the requirement text and criteria, the profile's conventions and the absolute path to `references/code-review-checklist.md`. Nothing about how you built it.

Then run the fix loop from `references/pipeline.md`: triage every finding (fixed / won't fix with evidence / deferred), push back on findings you believe are wrong, fix, re-run the Phase 1–2 gates, commit per the commit policy (`fix(<area>): address review`), and send the reviewer only the open findings with your dispositions and the new diff range. Stop after two re-review rounds; if blockers or majors remain, ask the user.

## Phase 4 — Security review + fix (ship-security-reviewer)

Spawn `ship-security-reviewer` after Phase 3's fixes land, so it reviews what will merge. Give it the base, the commit range, the requirement and criteria, the sensitive areas touched, the profile's threat-model notes and the absolute path to `references/security-checklist.md`.

Same fix loop, with these differences: every `critical` or `high` fix gets a regression test (write it or send it to the test writer); a `medium` stays open only with the user's explicit approval as an accepted risk — ask, don't assume; commit per the commit policy as `fix(security): <summary>`. Cheap `low` fixes are usually worth taking.

## Small changes: one combined review

For Small changes, spawn `ship-code-reviewer` once and tell it to apply **both** checklists. If it finds the change touches a sensitive area after all, switch to the Standard path and run Phase 4 properly.

## Phase 5 — Finish

1. **User-facing record:** changelog, release notes or whatever the profile names, written for users.
2. **Docs and spec:** update them where behaviour now differs — new flags, limits, exit codes, defaults.
3. **Done gate:** run the project's full check. If it runs a vulnerability or audit step separately, run that too.
4. **Plan or ticket:** mark the item done the way the project does. Skip this when the request came from the user's own words.
5. **Report** with the template in `references/pipeline.md`, including review tables with dispositions and a "Not verified" list (platforms you couldn't build, real integrations you deliberately didn't exercise).
6. **Merge and clean up** per the branching rules, but only after asking: merging into the base branch (even a local fast-forward) needs the user's OK, and the commit-policy opt-in doesn't cover it. Don't push, open PRs, merge to release branches or deploy unless the user asked — offer it. If changes are still uncommitted, say so in the report.

## Working with subagents

These habits keep parallel work from going wrong:

- **Stable snapshots for reviewers.** When anyone else may edit the working tree, tell reviewers to review a commit from a snapshot (`git archive <sha> | tar -x -C <scratch>`), not the live tree.
- **Never lose others' work.** Agents restore mutation checks with an editor, never `git checkout`/`git stash`/`git reset`, and you commit only your own files while others have work in progress (`git add <paths>`, not `git add -A`). If you must look at a file's committed version, use `git show <sha>:<path>`.
- **Agent reports are claims.** Check the commit, the diff and the test run before telling the user something is done.
- **Interrupted agents** (rate limits, errors): resume the same agent with SendMessage (or your tool's equivalent) instead of starting a new one; it keeps its context. If you can't resume, start a fresh one and give it the open findings and the current diff.
- **Missing agent types** (other tools, installs that skip agent registration): spawn a general-purpose agent per phase whose prompt starts "Read `<abs path>/agents/<name>.md` and act as that agent" — the agent definitions are bundled in this skill's own `agents/` directory, so they are present however the skill was installed. With no subagents at all, do each phase yourself, re-reading the diff and checklist from scratch for each review, and say in the report that reviews weren't independent.

## Safety rules

- **Nothing touches real user or system state** — login items and services, keychains, trust stores, shell profiles, global config, real accounts, production data — not the code under test, not the tests, not the agents. Tests use seams, temporary homes and fake binaries on `PATH`. Libraries sometimes ignore `$HOME` (for example, reading the home directory from the OS user database), so check where a library actually writes before trusting a temp `HOME`. If anything real was touched, put it back and tell the user plainly.
- **Commits need the user's OK** (see Commit policy), and so do outward-facing actions: pushing, publishing, opening PRs, merging into the base branch, sending messages, deploying.
- **Stay in scope.** Real problems found along the way that aren't part of this change go into the report as follow-ups (or a separate session), not into this diff.
