# Armonía del Ser — Workspace Guidelines

This file defines the strict runtime, development, and toolchain constraints for AI agents. You must adhere to these behavioral boundaries without deviation.

For operational context & ecosystem topology: [See README](./README.md)

---
## 1. System Discovery & Single Source of Truth (SSOT)

Do not assume architectural requirements or business logic rules. You must explicitly read the dedicated documentation inside each repository before modifying or writing code:
- **Repository Technical Blueprint:** Read the local `AGENTS.md` inside the target sub-repository (`armonia-api/`, `armonia-dashboard/`, or `armonia-landing/`) for repository-specific architecture, Nix `devShell` setup, and verification commands.
- **Product Requirements & Domain Model:** Review `docs/SRS.md` in the relevant repository.
- **System Topography:** Review `docs/architecture.md`.
- **Architectural Context & History:** Review Decisions in `docs/decisions/000*.md`.

### Documentation Drift Prevention Rule
Documentation must evolve symmetrically alongside the source code. If a feature implementation or bug fix alters system behavior, changes an invariant, or introduces a new pattern:
1. **Self-Documenting Code:** Write highly expressive code using explicit domain-driven function naming and clear test assertions.
2. **Inline & Static Sync:** You must simultaneously update the corresponding inline comments, type hints, Markdown documentation files, and Mermaid diagrams in the same commit.

---

## 2. Git & Branching Constraints

### Pre-Development Workflow
Before writing, modifying, or refactoring any code, you must execute the following terminal operations:
- **Clean Tree:** Verify the working directory is clean by running `git status`.
- **Sync Main:** Run: `git checkout main && git pull origin main`
- **Isolated Branch:** Check out a dedicated branch matching the patterns below.

### Branch Naming Conventions
- **Features:** `feat/<issue-id>-<short-description>` (e.g., `feat/123-add-login-form`)
- **Fixes:** `fix/<issue-id>-<short-description>` (e.g., `fix/456-resolve-null-pointer`)
- **Chores/Refactors:** `chore/<short-description>` (e.g., `chore/bump-dependencies`)

### Commit & Push Policy
- **Main Protection:** **NEVER** commit or push changes directly to the `main` branch.
- **Format:** Follow the [Conventional Commits](https://conventionalcommits.org) specification.
- **NixOS SSH Push Rule:** Prefix Git network push/pull operations with bypass flags to account for Nix store symlinks:
  ```bash
  GIT_SSH_COMMAND="ssh -F /dev/null" git push
  ```

---

## 3. Quality Assurance & TDD Mandate

### Test-Driven Development (TDD) Lifecycle
You must handle feature additions or bug fixes using an isolated **Red-Green-Refactor** workflow:
1. **Red:** Write dedicated, failing unit/integration tests before writing production logic.
2. **Green:** Write the minimal code required to pass the new tests.
3. **Refactor:** Clean code structures while maintaining green test flags.

### Anti-Regression & Tautology Safeguards
- **Strict Isolation:** Never modify or disable historical test suites written for other features.
- **No Tautological Tests:** Avoid writing meaningless assertions (e.g., matching basic types without checking real mutation mechanics or mocking database queries to return dummy values without exercising real SQLite/SQLModel constraints).

### Pre-Merge Toolchain Policy
Each sub-repository provides its own Nix `devShell` (`flake.nix` + `.envrc`) and defines its exact pre-commit verification gates in its local `AGENTS.md`. You must run and pass the target repository's verification suite locally with zero warnings or errors before proposing a commit or opening a pull request.

---

## 4. Gitea Organization & CLI (`tea`) Protocol

All repositories are hosted inside the private Gitea instance. Do not make unauthenticated external requests; always leverage the pre-configured `tea` CLI.

### Core Commands
- **List Repository Issues:** `tea issues list --repo armonia-del-ser/<repo-name>`
- **Read Issue Comments:** `tea issues <index> --repo armonia-del-ser/<repo-name> --comments`
- **Log Activity Comment:** `tea comment <index> --repo armonia-del-ser/<repo-name> "<comment>"`
- **Propose Pull Request:** `tea pr create --repo armonia-del-ser/<repo-name> --head <branch> --base main --title "<title>" --description "<body>"`

### Workflow Discipline
1. **Traceability:** Reference the working Gitea issue number in branch headers and commit message footers (e.g., `Refs: #1`).
2. **Secret Safety:** Never print or parse credentials, and never expose paths matching `~/.config/tea/config.yml` on terminal logs.

