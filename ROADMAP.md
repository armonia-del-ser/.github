# 🗺️ Armonía del Ser — Ecosystem Engineering Roadmap

This document is the authoritative cross-repository engineering roadmap for **Armonía del Ser** ([`armonia-del-ser`](https://gitea.roadtotech.me/armonia-del-ser)). It reconciles the operational discovery baseline, current codebase state across [`armonia-api`](https://gitea.roadtotech.me/armonia-del-ser/armonia-api), [`armonia-dashboard`](https://gitea.roadtotech.me/armonia-del-ser/armonia-dashboard), and [`armonia-landing`](https://gitea.roadtotech.me/armonia-del-ser/armonia-landing), and the active deliverables tracked in [`todo.txt`](./todo.txt).

---

## 🎯 Governing Architectural & Operational Invariants

1. **Separated Booking & Payment Truth (`ADR-0002`):** Every booking tracks three orthogonal axes (`booking_status`, `payment_arrangement`, `payment_status`). Prepayment is the default (`PREPAID_DEFAULT`); delayed-payment exceptions (`DELAYED_ALLOWED`) are explicitly authorized and audited without falsifying `payment_status`.
2. **Deliberate Slot Release & 48-Hour Visibility (`ADR-0002`):** Bookings never expire automatically on a background timer. At 48 hours prior to `scheduled_at`, unacknowledged bookings surface in `GET /bookings/follow-up-queue` so Jasmin's assistant can follow up via WhatsApp and either acknowledge payment, authorize delayed payment, or deliberately release the slot (`RELEASED`).
3. **Mandatory 1-on-1 Buffer Times (`ADR-0003`):** Individual consultations enforce a post-session buffer interval (`duration_minutes + buffer_minutes`) to prevent appointment overruns.
4. **Server-Enforced Google OAuth + Turso Allowlist (`ADR-0005`):** OAuth secrets and session signing keys live strictly in `armonia-api`. Every administrative request verifies the operator's email against Turso's `admin_allowlist` table and logs mutations in `admin_audit_log`.
5. **Infrastructure-Agnostic Layered Backend (`armonia-api#6`):** FastAPI routers (`app/routers/`) act as pure HTTP transport, delegating domain rules to `app/services/` and persistence to `app/repositories/` backed by `SQLModel` + `sqlalchemy-libsql` (`sqlite:///local.db` locally, `sqlite+libsql://` on Turso Cloud).
6. **Zero-Cost Infrastructure (`$0/month`):** `armonia-api` runs scale-to-zero on Google Cloud Run with Turso free-tier edge SQLite; `armonia-dashboard` and `armonia-landing` deploy as static edge bundles on Cloudflare Pages.

---

## 📊 Milestone Overview

| Milestone | Title | Target Repo(s) | Gitea Issue(s) | Status | Summary |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **M0** | **Discovery, Architecture & Governance** | `.profile`, all repos | [`armonia-api#1`](https://gitea.roadtotech.me/armonia-del-ser/armonia-api/issues/1) | 🔄 **In Progress (Finalizing PRs)** | Operational discovery, SRS v1.1, ADRs `0001`–`0005`, hierarchical `AGENTS.md`, per-repo Nix `devShell`s (`flake.nix`), and `.profile` org repo. |
| **M1** | **Core API MVP & Working-Tree Convergence** | `armonia-api` | [`armonia-api#3`](https://gitea.roadtotech.me/armonia-del-ser/armonia-api/issues/3) | 🎯 **Up Next (Immediate)** | Commit & PR existing verified orthogonal state engine, 48h follow-up queue, Google OAuth + Turso allowlist; add `GET /availability` and Group Roster API (`/rosters`). |
| **M2** | **SQLModel + Repository Refactor & Mutation Audit** | `armonia-api` | [`armonia-api#6`](https://gitea.roadtotech.me/armonia-del-ser/armonia-api/issues/6) | ⏳ **Planned** | Decouple routers from raw SQL via `SQLModel` (`app/models/`), `app/repositories/`, and `app/services/`; add `pytest-mock` I/O isolation and `mutmut` boundary audits. |
| **M3** | **Google Calendar Pull + Push Coordination** | `armonia-api` | [`armonia-api#4`](https://gitea.roadtotech.me/armonia-del-ser/armonia-api/issues/4) | ⏳ **Planned** | Complete live Google Calendar API adapter (`app/services/calendar.py`), on-demand pull reconciliation during availability/agenda reads, and recoverable push sync (`POST /bookings/{id}/sync-calendar`). |
| **M4** | **Admin Dashboard Control Surface** | `armonia-dashboard` | [`armonia-dashboard#1`](https://gitea.roadtotech.me/armonia-del-ser/armonia-dashboard/issues/1) | ⏳ **Planned** | Scaffold `Vite+` + `Tailwind` + `HTMX` + `Biome` mobile-first operator UI (48h follow-up queue, 1-tap payment acknowledgment, delayed-payment authorization, slot release, package & roster views). |
| **M5** | **Public Landing & Self-Service Portal** | `armonia-landing` | [`armonia-landing#1`](https://gitea.roadtotech.me/armonia-del-ser/armonia-landing/issues/1) | ⏳ **Planned** | Scaffold public service catalog, conflict-free slot picker (`GET /availability`), SPEI `ADS-XXXX` instructions, and 1-tap `wa.me` receipt handoff. |

---

## 🔍 Issue Alignment Audit (Gitea Issues vs. Code Reality)

During our workspace audit on `2026-10-18`, we identified three areas where original issue descriptions or working trees drifted from current reality:

1. **Uncommitted Working-Tree Accumulation (`M0` / `M1`):**
   - `armonia-api` (`feat/operational-truth-mvp`) currently holds **1,249+ lines of verified, passing code** (orthogonal state engine, 48h follow-up queue, Google OAuth + Turso allowlist, `CalendarService` push/retry scaffold, and ADR-0005) alongside the new `AGENTS.md` and `flake.nix`.
   - `armonia-dashboard` (`chore/operational-truth-docs`) and `armonia-landing` (`chore/agents-and-devshell`) hold uncommitted `AGENTS.md`, `README.md`, and `flake.nix` updates.
   - **Resolution:** Before writing any new feature code, slice and commit the pending changes in all three repositories via `/git-commit` and open/merge their PRs.
2. **Stale Original Issue Bodies vs. Revised Comments (`armonia-api#3`, `armonia-dashboard#1`):**
   - The original issue bodies still mention the superseded `PENDING_PAYMENT` 2-hour auto-expiration and static `X-Admin-Key` auth, whereas the `2026-10-18 15:10` comments specify the approved orthogonal model (`RESERVED` / `RELEASED`, `PREPAID_DEFAULT` / `DELAYED_ALLOWED`, `AWAITING_ACKNOWLEDGMENT` / `ACKNOWLEDGED`) and Turso `admin_allowlist`.
   - **Remaining unimplemented items in `armonia-api#3`:**
     - `GET /availability` endpoint (returning open slots factoring in `duration_minutes + buffer_minutes`).
     - Group Workshop Roster API (`GET /rosters`, `POST /rosters/quick-checkin` + `attended` tracking for \$50 MXN workshops).
3. **Architecture Ordering (`M2` before `M3`):**
   - Extracting `app/models/` (`SQLModel`), `app/repositories/`, and `app/services/` (`armonia-api#6`) **before** expanding `armonia-api` with `GET /availability`, `/rosters`, and live Google Calendar pull-on-read (`armonia-api#4`) prevents writing more raw SQL in routers only to rewrite it later.

---

## 🛠️ Detailed Milestone Breakdown

### Milestone M0: Discovery, Architecture & Workspace Governance
- [x] Conduct operational discovery with Jasmin and assistant (`armonia-api#1`).
- [x] Author SRS v1.1, system architecture, and ADRs `0001`–`0005`.
- [x] Restructure `AGENTS.md` hierarchy across workspace root and sub-repositories.
- [x] Provision per-repository NixOS `devShell`s (`flake.nix` + `.envrc` + `flake.lock`).
- [x] Track workspace root as `armonia-del-ser/.profile` on Gitea.
- [x] Commit and open PRs for `AGENTS.md` + `flake.nix` in `armonia-dashboard` ([PR #3](https://gitea.roadtotech.me/armonia-del-ser/armonia-dashboard/pulls/3)) and `armonia-landing` ([PR #3](https://gitea.roadtotech.me/armonia-del-ser/armonia-landing/pulls/3)).

### Milestone M1: Core API MVP Convergence (`armonia-api#3`)
- [x] Implement orthogonal booking & payment state engine (`booking_status`, `payment_arrangement`, `payment_status`).
- [x] Implement 48-hour follow-up visibility queue (`GET /bookings/follow-up-queue`), delayed-payment authorization, and deliberate slot release (`RELEASED`).
- [x] Implement Google OAuth2 login/callback + Turso `admin_allowlist` & `admin_audit_log`.
- [x] Slice, commit (`git-commit`), and open PR for `feat/operational-truth-mvp` in `armonia-api` ([PR #7](https://gitea.roadtotech.me/armonia-del-ser/armonia-api/pulls/7)).

### Milestone M2: SQLModel + Repository Refactor & Mutation Testing (`armonia-api#6`)
- [ ] Add `sqlmodel`, `sqlalchemy-libsql`, `pytest-mock`, and `mutmut` to `armonia-api/pyproject.toml`.
- [ ] Define `SQLModel` table entities in `app/models/` and configure `app/db/database.py` for Turso/SQLite portability.
- [ ] Extract all SQL queries from `app/routers/` into `app/repositories/` and pure business logic into `app/services/`.
- [ ] Isolate external I/O in tests using `pytest-mock` and run `mutmut` audit against `app/services/`.
- [ ] Implement `GET /availability` and Group Roster API (`GET /rosters`, `POST /rosters/quick-checkin`) cleanly on top of the new service/repository layers (`armonia-api#3`).

### Milestone M3: Google Calendar Pull + Push Coordination (`armonia-api#4`)
- [ ] Implement real Google Calendar REST integration in `app/services/calendar.py` (with `pytest-mock` test coverage).
- [ ] Implement Pull-on-Read schedule reconciliation during `GET /availability` and agenda reads.
- [ ] Verify end-to-end Push-on-Write status transitions (`[PAGO PENDIENTE]`, `[PAGO DIFERIDO]`, `[CONFIRMADO]`, event deletion on `RELEASED`/`CANCELLED`) and manual retry (`POST /bookings/{id}/sync-calendar`).

### Milestone M4: Mobile-First Admin Dashboard (`armonia-dashboard#1`)
- [ ] Initialize `Vite+` (`vp`), `Tailwind CSS`, `HTMX`, and `Biome` in `armonia-dashboard`.
- [ ] Delegate to **Anne Droid** (`feat/anne/*`) to implement the 5-state resilient operator views (`>= 44x44px` touch targets):
  - Google OAuth login & session guard.
  - 48-Hour Payment Follow-Up Queue with 1-tap `wa.me` links, Acknowledge Payment, Authorize Delayed Payment, and Release Slot actions.
  - Agenda & Google Calendar sync health indicator (`SYNCED` vs `FAILED` retry).
  - Multi-session package counter (`8 de 10`) & Group Workshop Roster check-in.

### Milestone M5: Public Landing & Self-Service Portal (`armonia-landing#1`)
- [ ] Initialize `Vite+` (`vp`), `Tailwind CSS`, `HTMX`, and `Biome` in `armonia-landing`.
- [ ] Delegate to **Anne Droid** (`feat/anne/*`) to implement the holistic landing page & self-service booking flow:
  - Service catalog cards (`PRESENCIAL` & `ONLINE`).
  - Live conflict-free slot picker consuming `GET /availability`.
  - Reservation confirmation with `ADS-XXXX` folio, SPEI CLABE instructions, and 1-tap WhatsApp receipt handoff.
