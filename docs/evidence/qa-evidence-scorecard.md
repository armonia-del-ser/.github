# Independent QA Evidence Scorecard: Group Workshop Roster Quick Check-in

> **Status:** Ratified & Frozen  
> **Evaluation Date:** 2026-10-27  
> **Evaluator:** Independent QA Specialist (Phase 4 Verification)  
> **Governing Verification Contract:** [`VERIFICATION-PACKET-ROSTER.md`](file:///home/kiskaadee/Projects/active/armonia/docs/pilot/VERIFICATION-PACKET-ROSTER.md)  
> **Release Authority:** None (Independent verification report submitted to Orchestrator)

---

## 1. Information Barrier Compliance Declaration

In strict compliance with QA Operational Invariant #1:
- Verification was conducted entirely via black-box interaction with the live frontend UI (`http://localhost:5173`) and running backend API (`http://localhost:8000`).
- Zero inspection of implementation source code (`src/` in dashboard, backend route implementations), pull request discussions, or developer self-audit was performed.
- All evaluation criteria were derived solely from the ratified Gate 2 Verification Packet.

---

## 2. Test Environment & Runtime Verification

| Component | Target URL / Port | Runtime Process | Database Target |
| :--- | :--- | :--- | :--- |
| **Frontend UI** | `http://localhost:5173` | Vite v8.3.4 (PID 1520429) | N/A (Consumes Backend API) |
| **Backend API** | `http://localhost:8000` | FastAPI / Uvicorn (PID 1535519) | SQLite (`test_armonia.db`) |
| **Headless Engine**| Chrome 155.0.8059.84 | Playwright / Chromium Driver | Linux x86_64 |

Preflight assertions formally confirmed:
1. `circulo-cacao-presencial` initialized with `max_capacity = 15`.
2. Today's session anchored at 18:00 hrs local business time (`2026-10-26`).
3. Seed attendees: Elena Gómez (`ADS-1001`, `RESERVED`, `ACKNOWLEDGED`, `attended: false`) and Carlos Méndez (`ADS-1002`, `RESERVED`, `AWAITING_ACKNOWLEDGMENT`, `attended: false`).
4. Preflight API check returned `occupied_count: 2`, `attended_count: 0`.

---

## 3. Executive Scorecard: Acceptance Criteria & Scenario Matrix

| Scenario & Criterion | Acceptance Criterion Description | Verified Persistence | Final Verdict |
| :--- | :--- | :--- | :---: |
| **Scenario 1 (`AC-1`)** | Navigation via top bar updates URL to `#rosters`, sets `aria-selected="true"`, renders workshop card at 18:00 hrs with `Ocupación: 2 / 15 (0 presentes)`. | N/A (Read) | **`PASS`** |
| **Scenario 2 (`AC-1`, `AC-2`)** | Roster lists Elena Gómez (`ADS-1001`, `REGISTRADO`, `PAGADO`) and Carlos Méndez (`ADS-1002`, `REGISTRADO`, `PENDIENTE`); direct WhatsApp links (`wa.me/...`) rendered. | N/A (Read) | **`PASS`** |
| **Scenario 3 (`AC-3`)** | Pre-paid Check-in for Elena Gómez: POST HTTP 200 (`paid_cash: false`), badge updates to `ASISTIÓ`, counter updates to `Ocupación: 2 / 15 (1 presentes)`, accessible toast displayed. | **F5 Reload & Backend API** | **`PASS`** |
| **Scenario 4 (`AC-4`)** | Cash Collection Check-in for Carlos Méndez: Checkbox checked, POST HTTP 200 (`paid_cash: true`), badge updates to `ASISTIÓ`, payment updates from `PENDIENTE` to `PAGADO`, counter to `2 presentes`. | **F5 Reload & Backend API** | **`PASS`** |
| **Scenario 5 (`AC-5`)** | Walk-In Registration: Modal opens, cash toggle defaults to checked (`true`), submit button prevents double-submission under rapid double-click, exactly 1 record created (`Lucía Vega`), counter to `3 / 15 (3 presentes)`. | **F5 Reload & Backend API** | **`PASS`** |
| **Scenario 6A (`AC-6`)** | 409 Conflict Disambiguation (Already Checked-In): Out-of-band check-in triggers HTTP 409; UI catches error, displays toast (*"El asistente ya contaba con asistencia registrada"*), reconciles row to `ASISTIÓ` without application crash. | UI State Reconciled | **`PASS`** |
| **Scenario 6B (`AC-6`)** | 409 Conflict Disambiguation (Capacity Reached): At capacity limit, walk-in returns HTTP 409 (`"El taller ha alcanzado su cupo máximo..."`); UI displays explicit alert, preserves form inputs, blocks insertion. | Form Data Preserved | **`PASS`** |
| **Scenario 7 (`AC-6`)** | Network Outage & Actionable Retry: Backend shutdown triggers error banner (*"No se pudieron cargar los talleres. Verifica tu conexión."*) with "Reintentar" button; backend restoration + clicking "Reintentar" recovers session card seamlessly without page reload. | Actionable Recovery | **`PASS`** |
| **Scenario 8 (`AC-7`)** | Mobile Ergonomics & Accessibility Tree: Viewport 375x812 displays zero horizontal scroll overflow; all check-in buttons, toggles, and modal inputs have touch bounding boxes $\ge 44 \times 44$ px; keyboard tab navigation exhibits visible focus outlines; toast container possesses `role="status"` and `aria-live="polite"`. | Viewport & A11y Tree | **`PASS`** |

---

## 4. Consequential Mutation Verification Audit

Every state-mutating scenario was validated through a three-stage assertion cycle:
1. **In-Session Reactive DOM Verification**: Immediate visual badge transitions and counter updates.
2. **Browser Reload Persistence (`F5`)**: Hard refresh to verify local cache re-hydration and state permanence.
3. **Out-of-Band Backend API Query**: Direct `GET /rosters?date_from=2026-10-26&date_to=2026-10-26` with Bearer auth token.

### Mutation Audit Table

| Scenario | Target Attendee | State Mutation Verified | F5 Reload Result | Backend API Read Payload Verification | Verdict |
| :--- | :--- | :--- | :--- | :--- | :---: |
| **Scenario 3** | Elena Gómez (`ADS-1001`) | `attended: false` $\rightarrow$ `true` | Badge: `ASISTIÓ`<br/>Counter: `1 presentes` | `{"booking_code": "ADS-1001", "attended": true}`<br/>`"attended_count": 1` | **`PASS`** |
| **Scenario 4** | Carlos Méndez (`ADS-1002`) | `attended: false` $\rightarrow$ `true`<br/>`payment_status: "AWAITING_ACKNOWLEDGMENT"` $\rightarrow$ `"ACKNOWLEDGED"` | Badge: `ASISTIÓ`<br/>Badge: `PAGADO`<br/>Counter: `2 presentes` | `{"booking_code": "ADS-1002", "attended": true, "payment_status": "ACKNOWLEDGED"}`<br/>`"attended_count": 2` | **`PASS`** |
| **Scenario 5** | Lucía Vega (Walk-In) | New booking created<br/>`attended: true`<br/>`payment_status: "ACKNOWLEDGED"` | Attendee listed<br/>Counter: `3 / 15 (3 presentes)` | `{"client_name": "Lucía Vega", "attended": true, "payment_status": "ACKNOWLEDGED"}`<br/>`"occupied_count": 3, "attended_count": 3` | **`PASS`** |
| **Scenario 6A** | Elena Gómez (`ADS-1001`) | Duplicate check-in rejected with 409 Conflict | Badge reconciles to `ASISTIÓ` | `{"booking_code": "ADS-1001", "attended": true}` | **`PASS`** |
| **Scenario 6B** | Sofía Castro (Walk-In) | Rejected with 409 Conflict (Capacity reached) | No attendee added<br/>Counter remains `2 / 2` | Attendee `"Sofía Castro"` is absent from backend API roster | **`PASS`** |

---

## 5. Observable Evidence Dossier & Artifact Citations

All visual evidence has been captured using headless Google Chrome and stored under `/home/kiskaadee/Projects/active/armonia/docs/evidence/qa/`:

### Visual Evidence Index

1. **Scenario 1 — Navigation, Hash & Capacity Overview:**
   - Evidence file: [`scenario_1_navigation_capacity.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_1_navigation_capacity.png)
   - Highlights: URL hash `#rosters`, tab `aria-selected="true"`, card title "Círculo de Cacao (Presencial)", time 18:00 hrs, counter `Ocupación: 2 / 15 (0 presentes)`.

2. **Scenario 2 — Roster Rendering & WhatsApp Direct Links:**
   - Evidence file: [`scenario_2_roster_badges.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_2_roster_badges.png)
   - Highlights: Elena Gómez (`ADS-1001`, `REGISTRADO`, `PAGADO`), Carlos Méndez (`ADS-1002`, `REGISTRADO`, `PENDIENTE`), direct WhatsApp anchor links `wa.me/525511112222` and `wa.me/525533334444`.

3. **Scenario 3 — Pre-paid Check-in & Persistence:**
   - Action Screenshot: [`scenario_3_prepaid_checkin.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_3_prepaid_checkin.png)
   - Reload Screenshot: [`scenario_3_persistence_reload.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_3_persistence_reload.png)
   - HTTP Request: `POST http://localhost:8000/rosters/quick-checkin`
     ```json
     {"booking_id": "b1111111-1111-1111-1111-111111111111", "paid_cash": false}
     ```
   - HTTP Response: `HTTP 200 OK`
     ```json
     {
       "walk_in": false,
       "attendee": {
         "booking_id": "b1111111-1111-1111-1111-111111111111",
         "booking_code": "ADS-1001",
         "booking_status": "COMPLETED",
         "payment_status": "ACKNOWLEDGED",
         "attended": true
       },
       "payment_id": null
     }
     ```

4. **Scenario 4 — Cash Collection Check-in & Payment Transition:**
   - Action Screenshot: [`scenario_4_cash_checkin.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_4_cash_checkin.png)
   - Reload Screenshot: [`scenario_4_persistence_reload.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_4_persistence_reload.png)
   - HTTP Request: `POST http://localhost:8000/rosters/quick-checkin`
     ```json
     {"booking_id": "b2222222-2222-2222-2222-222222222222", "paid_cash": true, "notes": "Cobro en puerta"}
     ```
   - HTTP Response: `HTTP 200 OK`
     ```json
     {
       "walk_in": false,
       "attendee": {
         "booking_id": "b2222222-2222-2222-2222-222222222222",
         "booking_code": "ADS-1002",
         "payment_status": "ACKNOWLEDGED",
         "attended": true
       },
       "payment_id": "f69f2d53-1cc5-423a-87b7-0b693a6718ef"
     }
     ```

5. **Scenario 5 — Walk-In Registration & Double-Submission Defense:**
   - Modal Open Screenshot: [`scenario_5_walkin_modal.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_5_walkin_modal.png)
   - Registered Roster Screenshot: [`scenario_5_walkin_registered.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_5_walkin_registered.png)
   - Reload Screenshot: [`scenario_5_persistence_reload.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_5_persistence_reload.png)
   - Double-Submission Test: Button disabled immediately upon initial click; rapid second click was ignored/blocked. Exactly 1 POST response received.
   - Result: Lucía Vega added with `ASISTIÓ` and `PAGADO`. Capacity updated to `3 / 15 (3 presentes)`.

6. **Scenario 6 — 409 Conflict Disambiguation:**
   - Case 6A Screenshot: [`scenario_6a_conflict_already_checked_in.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_6a_conflict_already_checked_in.png)
     - API 409 Payload: `{"detail": "This attendee has already checked in."}`
     - UI Feedback: Toast notification `"El asistente ya contaba con asistencia registrada"`.
   - Case 6B Screenshot: [`scenario_6b_conflict_capacity_reached.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_6b_conflict_capacity_reached.png)
     - API 409 Payload: `{"detail": "El taller ha alcanzado su cupo máximo (2) para esta sesión."}`
     - UI Feedback: Explicit error banner inside modal; modal inputs (`Sofía Castro`, `525512345678`) preserved; attendee list unaffected.

7. **Scenario 7 — Network Outage & Actionable Retry:**
   - Error Banner Screenshot: [`scenario_7_backend_down_error.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_7_backend_down_error.png)
   - Seamless Recovery Screenshot: [`scenario_7_retry_success.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_7_retry_success.png)
   - UI Behavior: Displays `"No se pudieron cargar los talleres. Verifica tu conexión."` and "Reintentar" button. Upon clicking "Reintentar" after server restart, the session card re-hydrated without browser reload.

8. **Scenario 8 — Mobile Ergonomics & Accessibility:**
   - Mobile 375px Roster Screenshot: [`scenario_8_mobile_375px.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_8_mobile_375px.png)
   - Mobile Walk-in Modal Screenshot: [`scenario_8_mobile_modal_375px.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_8_mobile_modal_375px.png)
   - Keyboard Focus Screenshot: [`scenario_8_keyboard_focus.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_8_keyboard_focus.png)
   - Touch Target Dimensions:
     - `Registrar Asistencia` button: `313.0px x 44.0px`
     - `＋ Registrar Asistente Sin Cita` button: `313.0px x 44.0px`
     - Cash toggle wrapper (`.cash-toggle-label`): `313.0px x 44.0px`
     - Modal Name input (`#walkin-name`): `295.0px x 44.0px`
     - Modal Phone input (`#walkin-phone`): `295.0px x 44.0px`
     - Modal Email input (`#walkin-email`): `295.0px x 44.0px`
     - Modal Cash checkbox label: `295.0px x 44.0px`
     - Modal Submit button: `191.8px x 54.0px`
     - Modal Cancel button: `93.2px x 54.0px`
     - Modal Close button: `44.0px x 44.0px`
   - A11y Tree: Toast container has `role="status"` and `aria-live="polite"`.

---

## 6. Console Log Audit & Defect Summary

- **Total Uncaught JavaScript Exceptions:** **0**
- **Console Errors Observed:** Zero uncaught runtime errors.
- **Contract Frictions or Anomalies:** None. The API contracts for quick check-in (`POST /rosters/quick-checkin`) and roster retrieval (`GET /rosters`) match the ratified specification in all respects, including idempotency behavior, cash attribution, capacity enforcement, and date boundary formatting.

---

## 7. QA Verdict & Conclusion

The implementation of the **Group Workshop Roster Quick Check-in** feature on branch `feat/anne/workshop-roster` satisfies 100% of the acceptance criteria defined in `VERIFICATION-PACKET-ROSTER.md` (`AC-1` through `AC-7`).

**Formal QA Verdict: `ALL SCENARIOS PASS` (8 / 8 PASS, 0 FAIL, 0 BLOCKED).**

---

## 8. Phase 4.1 Targeted Regression Addendum

**Execution Date:** 2026-10-27  
**Scope:** Targeted live browser DOM regressions for adversarial HTML/XSS input sanitization and out-of-band 409 counter consistency.

### 8.1 Test 1: Adversarial HTML / XSS Input Regression (Live Browser DOM)

- **Test Target:** Walk-In Registration modal form input handling (`#walkin-name`) and DOM projection.
- **Input Payload:** `Lucía <img src=x onerror="window.__xss_triggered=true"> "Vega" & Co.`
- **Phone:** `525598765432`
- **Initial Baseline:** `Ocupación: 2 / 15 (0 presentes)`.

#### Live DOM & Post-F5 Reload Observations

| Checkpoint | Expected Behavior | Live DOM Measurement | Post-F5 Reload Measurement | Verdict |
| :--- | :--- | :--- | :--- | :---: |
| **XSS Breakout Prevention** | `window.__xss_triggered === undefined` (zero script execution) | `null` / `undefined` | `null` / `undefined` | **`PASS`** |
| **HTML Element Injection** | Zero `<img>` elements rendered inside `.roster-attendee-row` | `0` `<img>` tags found | `0` `<img>` tags found | **`PASS`** |
| **Name Text Content** | Exact string preserved via text node assignment | `Lucía <img src=x onerror="window.__xss_triggered=true"> "Vega" & Co.` | `Lucía <img src=x onerror="window.__xss_triggered=true"> "Vega" & Co.` | **`PASS`** |
| **Attribute Escaping** | `.btn-wa-link[aria-label]` intact, not truncated by double quotes | `Contactar a Lucía <img src=x onerror="window.__xss_triggered=true"> "Vega" & Co. por WhatsApp` | `Contactar a Lucía <img src=x onerror="window.__xss_triggered=true"> "Vega" & Co. por WhatsApp` | **`PASS`** |
| **Backend API Invariant** | Direct `GET /rosters` returns exact payload without mutation | `client_name`: exact string stored and returned in JSON | Persistent across server restarts | **`PASS`** |

- **Screenshot Evidence:** [`scenario_9_xss_sanitization.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_9_xss_sanitization.png)
- **Verdict: `PASS`**.

---

### 8.2 Test 2: Live Out-of-Band 409 Counter Consistency (Single vs. Multi-Mutation)

#### Case 2A: Single Out-of-Band Check-in
1. Session initialized to `Ocupación: 2 / 15 (0 presentes)`.
2. Out-of-band API check-in executed for Elena Gómez (`ADS-1001`, `b1111111-1111-1111-1111-111111111111`).
3. Authoritative server state via `GET /rosters`: `occupied_count: 2, attended_count: 1`.
4. In the open, unrefreshed browser window, user clicked "Registrar Asistencia" on Elena's row.
5. Server returned `HTTP 409 Conflict` (`{"detail": "This attendee has already checked in."}`).
6. **Observed DOM Results:**
   - `#capacity-counter-0`: `Ocupación: 2 / 15 (1 presentes)`.
   - **Consistency with Server:** **100% Consistent** (`1 presentes` in DOM matches server `attended_count: 1`).
   - Elena's row reconciled to `ASISTIÓ` status badge.
   - Screenshot Evidence: [`scenario_6a_regression_single_oob.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_6a_regression_single_oob.png).
- **Verdict: `PASS`**.

#### Case 2B: Concurrent Multi-Attendee Out-of-Band Staleness Check & Re-verification

##### Initial Evaluation (Commit `9191899` - Phase 4.1): `FAIL`
1. Session initialized to `Ocupación: 2 / 15 (0 presentes)`.
2. Out-of-band API check-ins executed for **both** Elena Gómez (`ADS-1001`) AND Carlos Méndez (`ADS-1002`, `paid_cash: true`).
3. Authoritative server state via `GET /rosters`: `occupied_count: 2, attended_count: 2`.
4. In the open, unrefreshed browser window, user clicked "Registrar Asistencia" on Elena's row.
5. Server returned `HTTP 409 Conflict`.
6. **Initial Observed DOM Results:**
   - `#capacity-counter-0`: `Ocupación: 2 / 15 (1 presentes)` (diverged from server `attended_count: 2`).
   - Carlos Méndez's row: Remained visually stale as `REGISTRADO` with active check-in button and `PENDIENTE` badge.
   - Historical Screenshot: [`scenario_6a_regression_multi_oob.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_6a_regression_multi_oob.png).
7. **Verdict:** **`FAIL`** (Defect: Divergence between post-409 reconciled UI state and authoritative server state; client only performed local isolated increment rather than full authoritative reconciliation).

##### Phase 4.2 Targeted Live Re-verification (Updated Application): `PASS`
Following implementation of full authoritative roster reconciliation on 409 Conflict, Case 2B was re-evaluated under strict, falsifiable pass/fail criteria without pressing `F5` or clicking `Actualizar`:

| Assertion | Verification Checkpoint | Live DOM Measurement | Server Authoritative State | Assertion Verdict |
| :--- | :--- | :--- | :--- | :---: |
| **Assertion 1** | Capacity counter text matches authoritative server count | `Ocupación: 2 / 15 (2 presentes)` | `occupied_count: 2, attended_count: 2` | **`PASS`** |
| **Assertion 2** | Elena's row reflects checked-in state | Badge: `ASISTIÓ`<br/>Button: `✓ Asistió` (disabled) | `attended: true` | **`PASS`** |
| **Assertion 3** | Carlos's row reflects concurrent out-of-band check-in | Badge: `ASISTIÓ`<br/>Badge: `PAGADO`<br/>Button: `✓ Asistió` (disabled) | `attended: true`<br/>`payment_status: "ACKNOWLEDGED"` | **`PASS`** |

- **Re-verification Screenshot:** [`scenario_6a_regression_multi_oob_v2.png`](file:///home/kiskaadee/Projects/active/armonia/docs/evidence/qa/scenario_6a_regression_multi_oob_v2.png)
- **Phase 4.2 Re-verification Verdict:** **`PASS`** (All 3 strict criteria satisfied; zero DOM staleness or counter divergence).

---

### 8.3 Post-Regression Preflight State Restoration
- SQLite test database (`test_armonia.db`) was cleaned and reseeded to the baseline preflight state via `reset_db.py`.
- Final state confirmed: `occupied_count: 2`, `attended_count: 0`, attendees: `ADS-1001` and `ADS-1002`.

