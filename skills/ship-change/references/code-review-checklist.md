# Code review checklist

Apply the sections that match the touched code. Every finding needs a concrete failure scenario.

**Correctness** — Does the code do what the requirement says, including limits, defaults, error messages and exit/status codes? Edge cases: empty input, maximum sizes, timeouts, boundary values, missing dependencies or services. Errors carry context and are never silently swallowed; servers don't crash on bad input.

**Concurrency and resources** — No data races; shared state guarded or owned by one goroutine/thread/task. Every background task has a way to stop and doesn't outlive its owner. Files, sockets, response bodies, child processes and handles are closed or reaped on every path, including errors. Hot paths don't block on slow I/O.

**State and data** — Migrations are reversible or safe; writes are atomic where a crash could corrupt; caches are invalidated; IDs and ordering are stable.

**Architecture** — The change respects the project's layering and rules (CLAUDE.md/AGENTS.md). API and schema changes stay backward compatible as the project requires. New dependencies are justified and allowed.

**Cross-platform** — Paths built with the platform's path helpers; OS-specific code behind build tags/conditionals with a clear "unsupported" path elsewhere; per-platform behaviour matches the docs.

**Build and dependencies** — Lock/sum files are complete for every package that is built on its own (workspace builds can hide gaps); generated code is regenerated, not hand-edited.

**Clarity** — Names say what things are; functions do one thing; no dead code, speculative abstraction or commented-out blocks; comments explain why.

**Tests and docs** — Tests cite the requirement ID or criterion number, cover every criterion, cover error paths and can fail; a bug fix has a test that reproduces the bug. Spec, README, changelog and API docs updated where behaviour changed.
