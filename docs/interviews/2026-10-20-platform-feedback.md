# Homelab Deployment First Impression: Client Feedback

**Date Recorded:** 2026-10-20  
**Context:** Homelab deployment to `armonia.roadtotech.me` as a live PoC and exploratory demo before Google OAuth implementation and distributed production deployment (Google Cloud Run / Cloudflare Pages / Turso DB).  
**Source:** Direct exchange and feedback from Jazmín's assistant following hands-on exploration of the live Homelab demo.  
**Purpose:** Document operational feedback, friction points, and user expectations, translating operator observations into actionable engineering requirements and user stories.

---

## Executive Summary & Core Insights

1. **Payment Delegation & Dual-Step Verification:** Jazmín's primary emotional and operational bottleneck is payment tracking and collections. The assistant can act as the intake frontline (registering payment vouchers), but the system must provide a **two-step verification** (*Assistant acknowledges voucher* $\to$ *Jazmín reconciles against bank account*) without requiring repetitive WhatsApp back-and-forth.
2. **Commitment & Extension (Prórroga) Lifecycle:** Granting an extension without an automated alert or calendar reminder causes high operational anxiety regarding forgotten receivables.
3. **Clinical / Session History per Patient:** Concluding a session and taking notes must feed into a dedicated **Patient Profile** where historical session notes can be reviewed prior to future consultations.
4. **Vocabulary Accessibility:** Technical terminology like `"Buffer"` must be replaced with operational domain terms (e.g., *"Tiempo de descanso entre sesiones"* / Inter-session break time).
5. **WhatsApp Message Framing:** Automated/pre-filled WhatsApp messages received high praise for looking professional and reducing manual typing effort.

---

## Identified Gaps & Operational Friction Points

Upon receiving and analyzing client feedback from hands-on testing of the live Homelab demo, we identified key gaps in how the platform supports day-to-day operations. The primary friction points center around payment assurance, commitment tracking, clinical context continuity, and interface terminology:

* **Unmonitored Payment Promises:** When the assistant grants a payment extension (*prórroga*), the system does not actively track the agreed deadline. Without proactive reminders, uncollected fees depend entirely on operator memory.
* **Payment Intake vs. Bank Audit:** While the assistant can quickly acknowledge payment vouchers received over WhatsApp, the business owner (Jazmín) lacks an audited reconciliation routine to confirm that funds actually cleared the bank account before or after sessions.
* **Discontinuous Session Context:** Marking an appointment as complete allows writing a session note, but there is no patient dossier view where historical consultation notes can be reviewed prior to future sessions.
* **Technical Jargon in Domain Interfaces:** Developer-centric terminology like *"Buffer"* generates confusion for non-technical operators.
* **Role Conflation:** Administrative intake tasks (registering vouchers, WhatsApp links) are mixed with owner responsibilities (bank reconciliation, clinical notes, financial analytics) instead of presenting tailored, role-specific views.

To bridge these operational observations directly into engineering deliverables, each pain point is formalized below as an Agile user story with verifiable acceptance criteria (BDD/Gherkin) for upcoming vertical slices.

---

## Synthesis Matrix: Operational Need to Engineering Requirement

| # | Operational Friction | Proposed Solution | Target Component |
|---|----------------------|-------------------|------------------|
| 1 | Payment extensions forgotten after commitment date | Add `due_date` & filter for pending/expired extensions | `armonia-api` / `armonia-dashboard` |
| 2 | "Buffer" term confusion | Rename to *"Tiempo de descanso"* / *"Margen entre citas"* | `armonia-dashboard` UI copy |
| 3 | Payment fraud risk / lack of bank audit | 2-step verification: `voucher_received` -> `reconciled` | `armonia-api` (Payments) & Dashboard View |
| 4 | Clinical history inaccessible prior to session | Client Dossier with consultation notes timeline | `armonia-dashboard` (`PatientDetailView`) |
| 5 | Role confusion between operator & owner | Role-based routing (Operator Intake vs Owner Audit) | `armonia-dashboard` Auth & Navigation |

---

## User Stories & Acceptance Criteria

### Story 1: Tracking Payment Extension Deadlines (Prórrogas)
* **As an:** Intake Operator / Assistant
* **I want to:** Assign a mandatory commitment deadline (`due_date`) when granting an authorized payment extension (`DELAYED_ALLOWED`), and view a filterable queue of pending/overdue extensions in the dashboard
* **So that:** I can follow up proactively with clients on promised payments without relying on memory or disconnected manual notes.
* **Acceptance Criteria:**
  - **Given** an appointment in `RESERVED` status with delayed payment modality (`DELAYED_ALLOWED`),
  - **When** the operator authorizes the extension,
  - **Then** the system requires entering a mandatory promised payment date (`due_date`).
  - **Given** that the current date is on or after `due_date` and payment remains pending,
  - **Then** the appointment is highlighted visually in a dedicated "Pending Collections / Prórrogas" queue, displaying days remaining or days overdue.

---

### Story 2: Two-Step Payment Verification & Audit (Voucher Acknowledgment & Bank Reconciliation)
* **As a:** Practice Owner (Jazmín)
* **I want to:** Implement a two-step payment workflow (`voucher_received` -> `reconciled` or `voucher_invalid`)
* **So that:** My assistant can unblock and confirm sessions upon receiving WhatsApp transfer vouchers, while I retain a dedicated routine to audit and confirm that funds actually cleared the bank account.
* **Acceptance Criteria:**
  - **Given** a client who sends a transfer receipt voucher via WhatsApp,
  - **When** the operator validates it at intake,
  - **Then** the payment status transitions to `VOUCHER_RECEIVED` (logging operator ID and timestamp), clearing the appointment to proceed without requiring immediate owner sign-off.
  - **Given** a list of payments in `VOUCHER_RECEIVED` status,
  - **When** Jazmín reviews bank statement deposits from her audit dashboard,
  - **Then** she can individually mark each payment as `RECONCILED` (settled) or `VOUCHER_INVALID` (rejected with a mandatory note).

---

### Story 3: Patient Dossier & Consultation History
* **As a:** Principal Therapist (Jazmín)
* **I want to:** Open an individual client dossier showing chronological consultation history and session notes
* **So that:** Prior to each call, I can review past discussion topics, pending homework assignments, and treatment progress.
* **Acceptance Criteria:**
  - **Given** an appointment marked as `COMPLETED` with session notes recorded,
  - **When** Jazmín clicks on the client's name from the calendar or appointment table,
  - **Then** the system opens the Client Dossier displaying the full historical timeline of appointments and session notes.
  - **Privacy Boundary Rule:** Clinical consultation notes are strictly confidential and restricted to the `therapist` role. The `operator` (assistant) role can only view contact details, scheduled appointments, and payment statuses.

---

### Story 4: Terminology Localization ("Buffer" -> "Break Time / Interval")
* **As a:** Practice Operator / Administrator
* **I want to:** View natural clinical domain terminology rather than English developer jargon
* **So that:** Onboarding is frictionless and scheduling intervals between consecutive sessions are immediately understood.
* **Acceptance Criteria:**
  - **Given** any service configuration or scheduling view in the dashboard,
  - **When** the separation interval between consecutive appointments is displayed,
  - **Then** the interface label must read *"Tiempo de descanso entre sesiones"* or *"Margen entre citas"* (Break time / Inter-session interval), completely removing the technical term *"Buffer"*.

---

### Story 5: Role-Based Dashboard Navigation (Operator vs. Therapist Views)
* **As an:** Operations Assistant (`operator`)
* **I want to:** Access a streamlined navigation menu focused exclusively on scheduling, voucher intake, and WhatsApp communication
* **So that:** My daily workspace is fast, uncluttered by financial metrics or clinical records that fall outside my duties.
* **Acceptance Criteria:**
  - **Given** an active session authenticated as `operator`,
  - **When** viewing the dashboard menu,
  - **Then** navigation is limited to: *Calendar / Schedule*, *Book Appointment*, *Extensions Queue*, and *Voucher Intake*.
  - **Given** an active session authenticated as `therapist`,
  - **Then** the navigation additionally exposes: *Client Dossiers*, *Bank Reconciliation*, and *Financial Analytics*.
