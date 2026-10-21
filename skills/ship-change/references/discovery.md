# Discovering a project's conventions

Use this when the repo has no `.claude/ship-change.md`. The goal is a profile (see `profile-template.md`) good enough to run the pipeline without guessing. Read before you run anything; prefer the project's wrappers (Makefile, scripts, task runners) over raw tool invocations, because they often set flags the project depends on.

## Where to look, in order

1. **Agent and contributor docs:** `CLAUDE.md`, `AGENTS.md`, `.claude/`, `CONTRIBUTING.md`, `README.md`, `docs/` (look for workflow, architecture, spec, ADR, roadmap or plan documents).
2. **Task runners:** `Makefile` (`make help`), `justfile`, `Taskfile.yml`, `package.json` scripts, `pyproject.toml` (`[tool.*]`, scripts), `Cargo.toml` aliases, `build.gradle(.kts)`, `pom.xml`, `mix.exs`, `composer.json`, `Rakefile`, `scripts/`.
3. **CI:** `.github/workflows/`, `.gitlab-ci.yml`, `.circleci/`, `azure-pipelines.yml`, `Jenkinsfile`, `buildkite/`. CI shows the commands that actually gate merges — the most reliable "done" gate.
4. **Tool configs:** linters and formatters (`.golangci.yml`, `.eslintrc*`, `eslint.config.*`, `biome.json`, `.prettierrc*`, `ruff.toml`, `.flake8`, `rustfmt.toml`, `clippy.toml`, `.rubocop.yml`, `.editorconfig`), type checkers (`tsconfig.json`, `mypy.ini`, `pyrightconfig.json`), test configs (`jest.config.*`, `vitest.config.*`, `pytest.ini`, `conftest.py`).
5. **Git history:** `git log --oneline -30` for commit style; `git branch -a` for branch naming and the integration branch; tags for release style.
6. **Release records:** `CHANGELOG.md`, `.changeset/`, `release-please` config, `releases.json`, `docs/releases/`, GitHub release templates.

## What to extract

| Profile field | Typical sources | If you can't find it |
| --- | --- | --- |
| Format / lint / build / test / done-gate commands | Task runner, CI | Use the ecosystem default *only* if it's read-only and safe (e.g. `go test ./...`, `npm test`, `pytest`, `cargo test`); say so in the report |
| Race, integration, e2e variants | CI matrix, test tags, `*.integration.*` | Skip, list under "Not verified" |
| Vulnerability / audit step | CI (`govulncheck`, `npm audit`, `pip-audit`, `cargo audit`, Dependabot) | Run the ecosystem's read-only audit if installed |
| Dependency lock hygiene | `go mod tidy`, lockfile checks in CI | Check lock files are unchanged after the build |
| Requirements source and ID scheme | Spec/PRD/plan docs, issue tracker refs in commits | Use the user's words and the issue text |
| Sensitive areas | Security docs, CODEOWNERS, architecture docs | Use the default list in `pipeline.md` |
| Base branch and naming | `git branch -a`, CONTRIBUTING, CI triggers | Default branch; `feat/`/`fix/`/`chore/` prefixes |
| Merge style | CONTRIBUTING, history (merge commits vs squash) | Ask before merging; otherwise leave the branch for a PR |
| Commit convention | History (Conventional Commits, ticket prefixes) | Conventional Commits |
| User-facing record | Changelog tooling, release docs | Mention user-visible changes in the report |
| Docs to keep in sync | Spec, README per package, API docs | Update what the change makes wrong |
| Safety rules | Docs about real accounts, devices, services; test helpers that fake system state | The defaults in SKILL.md "Safety rules" |

## Monorepos and workspaces

Find the module/package boundaries (`go.work`, `pnpm-workspace.yaml`, `turbo.json`, `nx.json`, Cargo workspaces, Gradle includes). Run gates per affected package where the project does, and check whether CI also builds packages in isolation — workspace builds can hide missing per-package dependency entries.

## Before you start

Write the profile down (in your notes or the task list) and show the user the commands you intend to use in your Phase 0 summary. If a command could modify things outside the repo (deploy, publish, migrate, seed a shared database), don't run it as a gate.
