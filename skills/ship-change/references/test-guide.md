# Test guide

- **Write from the requirement and the interface.** A test should fail if the requirement stopped being met, whatever the implementation does.
- **Prove each key test can fail.** Break the behaviour briefly with an in-place edit, watch the test fail, restore it with your editor (never `git checkout`/`stash`/`reset`).
- **Hermetic by default.** Temp directories, loopback listeners on port 0, injected clocks, fakes for system integrations. Anything needing privileges, real devices or real services goes behind the project's integration tag.
- **Never touch real user or system state.** Replace service managers, keychains, trust stores and similar with fakes (for example, fake binaries first on `PATH`), point homes at temp dirs, and check where libraries actually write — some read the home directory from the OS user database and ignore `$HOME`. Add a guard that makes the real path fail in tests when you can.
- **No sleeps for synchronisation.** Use channels, events, polling with deadlines or injected clocks.
- **Clean up everything you start** — servers, processes, goroutines/threads, temp files — so leak checks stay green.
- **Error paths and limits count as much as the happy path:** timeouts, malformed input, boundary values, concurrent callers, the dependency being down.
- **Name tests after behaviour and cite the requirement ID or criterion number** (`R1`, `US-12`, …) in the name or a comment. Prefer table-driven cases and readable diffs (`cmp.Diff`, `assert.deepEqual`, snapshot tools) over hand-rolled field checks.
- **For a bug fix, reproduce it first.** At least one test encodes the reported inputs and expected result and fails on the pre-fix code. Check that against a snapshot of the base commit, never by reverting the fix in the working tree. Add a case for the nearest behaviour that must not regress.
- **Match the project's frameworks and helpers;** don't introduce a new test framework without asking.
