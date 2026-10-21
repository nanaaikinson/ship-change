# Security review checklist

Focus on the sections that match the touched areas and the project's threat model. Report concrete attacker → action → impact scenarios only.

**A. Input and injection** — Untrusted input (requests, files, CLI args, env, IPC, webhooks, imports) is validated at the boundary. No string-built SQL, shell commands, templates or paths; parameterised queries, exec-style argument lists, escaping for the target format (HTML, XML/plist, unit files, YAML). Parsers of untrusted input are bounded and fuzzed.

**B. Authentication and authorisation** — Every endpoint/operation checks the caller's right to this specific resource (no IDOR). Tokens: algorithm pinned, expiry/audience/issuer checked. Secrets compared in constant time. CSRF, Origin and Host checks for browser-reachable local or web APIs (DNS rebinding).

**C. Secrets and crypto** — No secrets in code, logs, errors, URLs or telemetry. Keys stored with the OS secret store or owner-only files. Randomness from a CSPRNG. Modern TLS; no disabled verification outside loopback test setups.

**D. File system and processes** — Writes don't follow attacker-placed symlinks; owner-only modes for private files; paths persisted for later execution (services, cron, login items) live in directories no other user can modify (check owners and group/other write bits up the tree, sticky dirs included). Child processes get exec-style args and minimal env.

**E. Network exposure** — Listeners bind to the narrowest interface; opt-in for wider exposure. Timeouts and size limits on every connection and body; no unbounded headers.

**F. Privacy and data handling** — User data stays where the product promises; sensitive fields masked or redacted in UI, logs and exports; retention and size caps enforced.

**G. Supply chain and CI** — New dependencies maintained, licensed compatibly and free of known vulnerabilities (run the audit tool). CI actions pinned by SHA; least-privilege tokens; untrusted event data never reaches shell steps; release builds don't reuse caches that untrusted steps can write.

**H. Resource exhaustion** — Bounded queues, workers, buffers, retries and polling; rate limits where promised; external calls have timeouts.

**I. Privilege and system changes** — Elevated components have a fixed operation list, validate every argument, verify their caller and record what they change so it can be undone.
