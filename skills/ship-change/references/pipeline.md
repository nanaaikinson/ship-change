# Pipeline rules

Sizes, gates, findings, severities, the fix loop and the report. The project's own workflow document, if it has one, overrides anything here.

## Contents

1. Change size
2. Phase gates
3. Findings format
4. Severities
5. Fix loop
6. Final report template

## 1. Change size

Pick the size in Phase 0. It decides how much of the pipeline runs.

| Size | What it is | Pipeline |
| --- | --- | --- |
| **Docs** | Docs, comments or release-note wording only; no code or config | No pipeline. Check links; run the release-notes validator if those files changed. |
| **Small** | Under ~50 changed lines, one package/module, not in a sensitive area | Phases 0–2 and 5, plus **one combined review** using both checklists |
| **Standard** | Anything else | All phases |

**Sensitive areas** are never Small, and Phase 4 reviews them in depth. Use the profile's list; if there is none, treat these as sensitive: authentication and authorisation; session, token and secret handling; cryptography and certificates; anything running with elevated privileges or installing system components; network listeners and request parsing; file-system writes outside the project's own data; process execution and shell-outs; deserialisation and parsers of untrusted input; payments and billing; data export or upload; build, release, packaging and CI configuration; install scripts; the public API surface (schemas, RPC definitions).

## 2. Phase gates

| Phase | Gate | Commit |
| --- | --- | --- |
| 0 Intake | Requirement (ID, or the user's own description) written as numbered testable criteria (for a bug: the reproduction), size, sensitive areas, plan, branch | — |
| 1 Build | Format + lint + build pass | `feat(<area>): <summary> (<ID>)` |
| 2 Tests | Test command passes; race/integration variant for changed packages where the project has one; key tests proven able to fail | `test(<area>): <summary>` |
| 3 Code review | No open `blocker`/`major`; every finding has a disposition | `fix(<area>): address review` |
| 4 Security review | No open `critical`/`high`; any open `medium` approved by the user as an accepted risk; regression tests for every `critical`/`high` fix | `fix(security): <summary>` |
| 5 Finish | Done gate passes; docs, spec and user-facing notes updated; plan/ticket marked | merge per branching rules |

Use the project's commit convention if it differs from the examples above. The Commit column is the proposed message: nothing is committed until the user approves it, or has opted in to automatic commits for the run (see "Commit policy" in SKILL.md). A gate that says "commit" is still met when the user declines; report the changes as uncommitted.

## 3. Findings format

Reviewers return one block per finding, most severe first:

```text
[<severity>] <category> — <path>:<line>
Problem: <what is wrong and the concrete scenario where it breaks>
Fix: <the smallest change that fixes it>
```

And end with one line: `Verdict: APPROVE` or `Verdict: CHANGES REQUESTED`.

A finding needs a concrete failure scenario: inputs or state that lead to a wrong result, crash, leak or exploit. "Could be cleaner" without a consequence is a `nit` at most.

## 4. Severities

| Code review | Meaning | Must fix? |
| --- | --- | --- |
| `blocker` | Wrong behaviour, data loss, crash, breaks the build, CI or the spec | Yes |
| `major` | Likely bug, race, leak, missing error handling, architecture rule broken | Yes |
| `minor` | Real but low-impact; unclear code that will cause bugs later | If cheap, else defer |
| `nit` | Style or preference | No |

| Security | Meaning | Must fix? |
| --- | --- | --- |
| `critical` | Remote or cross-site exploit, privilege escalation, sensitive data leaving its boundary | Yes, before anything else |
| `high` | Exploitable with some preconditions; auth or limit bypass; secret exposure | Yes |
| `medium` | Defence-in-depth gap; exploit needs unlikely conditions | Fix, or accepted risk with the user's approval |
| `low` | Hardening suggestion | Optional — take it when cheap |

## 5. Fix loop

Record a disposition for every finding:

- **fixed** — with the commit.
- **won't fix** — the finding is wrong; say why with evidence (code, spec, test, measurement). The reviewer can be mistaken; don't make a change you can't justify.
- **deferred** — real but out of scope; becomes a follow-up. Not allowed for `blocker`, `major`, `critical` or `high`.

After fixing, re-run the Phase 1 and 2 gates, then send **only the open findings with their dispositions and the new diff range** back to the same reviewer for re-review. After two re-review rounds with blocking findings still open, stop and ask the user: list what's open and why it's hard.

New findings in a re-review are normal; treat them the same way.

## 6. Final report template

```markdown
## <Change title>

**Requirements:** <IDs, or the criteria R1…Rn derived from the user's request>; for a bug, the reproduction
**Size:** Small | Standard   **Branch:** <branch>   **Commits:** <short SHAs>

### What changed
- <one line per meaningful change>

### Tests
- <tests added, what each covers, requirement IDs>
- Gates: <done gate> ✔ · <race/integration> ✔ · <vuln/audit> ✔

### Code review (<n> rounds)
| Severity | Finding | Disposition |
| --- | --- | --- |

### Security review (<n> rounds)
| Severity | Finding | Disposition |
| --- | --- | --- |

### Follow-ups
- <deferred findings and open questions, or "none">

### Not verified
- <anything not run or checked: other platforms, real integrations deliberately not exercised>
```

In the chat, lead with the outcome and anything the user must decide or do; the tables can go in a file or PR description if they're long.
