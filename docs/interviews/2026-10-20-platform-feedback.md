# Homelab Deployment First Impression: Client Feedback

**Date Recorded:** 2026-10-20  
**Context:** Homelab deployment to `armonia.roadtotech.me` as a live PoC and exploratory demo before Google OAuth implementation and distributed production deployment (Google Cloud Run / Cloudflare Pages / Turso DB).  
**Source:** Voice notes from Jazmín's assistant (`WhatsApp Ptt 2026-10-19` & `2026-10-20`) after hands-on testing of the live Homelab demo.  
**Purpose:** Document operational feedback, friction points, and user expectations, translating verbatim observations into actionable engineering requirements and user stories.

---

## Executive Summary & Core Insights

1. **Payment Delegation & Dual-Step Verification:** Jazmín's primary emotional and operational bottleneck is payment tracking and collections. The assistant can act as the intake frontline (registering payment vouchers), but the system must provide a **two-step verification** (Asistente acredita -> Jazmín concilia contra banco) without requiring repetitive WhatsApp back-and-forth.
2. **Commitment & Extension (Prórroga) Lifecycle:** Granting an extension without an automated alert or calendar reminder causes high operational anxiety regarding forgotten receivables.
3. **Clinical / Session History per Patient:** Concluding a session and taking notes must feed into a dedicated **Patient Profile** where historical session notes can be reviewed prior to future consultations.
4. **Vocabulary Accessibility:** Technical terminology like `"Buffer"` must be replaced with operational domain terms (e.g., *"Tiempo de descanso entre sesiones"*).
5. **WhatsApp Message Framing:** Automated/pre-filled WhatsApp messages received high praise for looking professional and reducing manual typing effort.

---

## Detailed Voice Note Transcripts & Technical Requirements

### 1. 2026-10-19 10:09:21 PM (`WhatsApp Ptt 2026-10-19 at 10.09.21 PM.ogg`)

#### A. Seguimiento y alertas de vencimiento en prórrogas de pago
> *"En el caso en que se concede una prórroga, yo no veo... cómo puedo estar pendiente de que se venza esa prórroga. O sea, como que no aparece tipo un calendario o algo así. Por ejemplo, en la que le di prórroga, escribí que se había comprometido para pagar el 30 de este mes. Pero entonces como que no entiendo yo cómo me voy a acordar de que él sí me haya pagado, un ejemplo el primero de noviembre, verificar que sí me haya pagado o no."*

**Key Takeaways & Requirements:**
- **Observable Debt Lifecycle:** Extensions need explicit structured dates (`due_date`) rather than free-form notes.
- **Engineering Implication:**
  - Add `due_date` to payment extension models.
  - Create a dashboard filter/widget for "Vencimientos Próximos / Vencidos" to alert the operator when a promised payment date arrives.

#### B. Claridad de vocabulario en interfaz ("Buffer")
> *"Y que no entendí qué significa 'buffer'. No sé, bueno, esa palabra no recuerdo bien qué es lo que representa."*

**Key Takeaways & Requirements:**
- **Cognitive Friction with Technical Jargon:** Developer-centric terminology creates confusion for operators.
- **Engineering Implication:** Replace `"Buffer"` with domain terms like *"Tiempo de descanso"* or *"Margen entre citas"* across all UI labels and tooltips.

#### C. Conciliación bancaria en dos pasos / Verificación de transferencias
> *"Supongamos que la persona me envía el comprobante de pago y yo listo, acredito el pago. Pero yo pienso que igual debe haber un momento en el mes, o varios momentos, en que Jazmín como que vaya y chequee efectivamente que esos pagos sí se hicieron y que tanto no es una imagen que la persona solo me envió y que sea falsa, porque tampoco me imagino yo estarle preguntando cada vez que alguien pague, porque si no qué sentido tendría como que estar ayudándola o asistiéndola... Veo que sale como para poner la nota, pero no sé si hay una manera como de ver todas las citas ya supuestamente pagadas (se hayan tomado o no) que fueron por transferencia y que ella pueda ir a checar de si se transfirió, si llegó, o como darle un ok, un segundo chequeo."*

**Key Takeaways & Requirements:**
- **Dual-State Payment Workflow:** Assistants verify receipt vouchers; owner verifies actual bank settlement.
- **Engineering Implication:**
  - Transition payment status from boolean to state machine: `pending` -> `voucher_received` (operator accredited) -> `reconciled` (owner bank verified).
  - Dedicated "Conciliación Bancaria" view listing all transfer-based payments awaiting owner audit.

---

### 2. 2026-10-19 10:11:51 PM (`WhatsApp Ptt 2026-10-19 at 10.11.51 PM.ogg`)

#### Diferenciación de roles y escalamiento (Asistente vs. Jazmín)
> *"Bueno, y vi que existe como mi perfil de ASU [asistente], entonces yo me imaginé si de pronto eso último que acabo de decir se podía hacer como desde otro perfil o algo así... como cuando uno escala algo a un superior en un call center, algo así."*

**Key Takeaways & Requirements:**
- **Role-Based Views (RBAC):** Operator interface focuses on booking and receipt registration; Owner interface focuses on reconciliation, financial reporting, and clinical notes.
- **Engineering Implication:** Enforce role views (`operator` vs `therapist/admin`) in dashboard routing and permissions.

---

### 3. 2026-10-19 10:15:08 PM (`WhatsApp Ptt 2026-10-19 at 10.15.08 PM.ogg`)

#### Perfil de paciente e historial de notas de consulta
> *"Por ejemplo concluí una cita —me imagino que ese botón significa que ya la paciente tomó la cita— y pues dejé como una nota. Entonces luego quise ir al perfil de la paciente, pero no abre como que un perfil de paciente como para ver esa nota que había dejado, cosa que ella [Jazmín] pudiese como dejar notas de cada paciente y así antes de conectarse a la llamada pues pueda ver en qué quedaron o qué venían trabajando, como un historial del paciente o algo así sería demasiado chimba tener... Porque a veces como consultante uno espera que el psicólogo se acuerde de todo... entonces tenerlo en el mismo sistema sería muy chimba."*

**Key Takeaways & Requirements:**
- **Session Notes Continuity:** Clicking a patient must open a dossier with session history and past notes.
- **Engineering Implication:**
  - Build `PatientDetailView` / Client Dossier showing past appointments, notes timeline, and payment history.
  - Privacy boundary: Clinical consultation notes should only be readable by Jazmín (`therapist`), not regular operators.

---

### 4. 2026-10-19 10:17:47 PM (`WhatsApp Ptt 2026-10-19 at 10.17.47 PM.ogg`)

#### Plantilla y redacción de mensajes de WhatsApp
> *"También corrí con la suerte de que pude programar una cita justo con un perfil que tenía el número de Jazmín (número válido en WhatsApp). Entonces vi cómo plantea el mensaje y me encantó... quedó muy pro."*

**Key Takeaways & Requirements:**
- **High User Satisfaction:** The automated WhatsApp booking confirmation message meets professional standards and eliminates manual drafting effort.
- **Engineering Implication:** Maintain the current message template structure; consider configurable variables (e.g. video link, payment instructions) for future iterations.

---

### 5. 2026-10-19 10:29:32 PM (`WhatsApp Ptt 2026-10-19 at 10.29.32 PM.ogg`)

#### Dudas de arquitectura / Servidor vs. Front-end y costos
> *"Solo para corroborar que entendí bien: tanto este servidor (no sé cómo llamarlo, página) como una página web serían gratuitas, pero lo que sí es un servidor no lo entiendo bien cómo se tiene... si yo no tengo encendido el computador cómo funciona, si eso sí habría que pagarlo... Pero en verdad sí suena como que es una súper herramienta..."*

**Key Takeaways & Requirements:**
- **Mental Model of Cloud Hosting:** Need clear, non-technical onboarding explaining cloud deployment (running 24/7 in Google Cloud / Cloudflare without requiring their personal laptops to stay on).

---

### 6. 2026-10-19 10:29:49 PM (`WhatsApp Ptt 2026-10-19 at 10.29.49 PM.ogg`)

#### Modelo comercial / Comisión
> *"Yo no sé, un 10%... Bueno, vamos a ver qué hablamos mañana."*

**Key Takeaways & Requirements:**
- **Assistant Compensation Structure:** Assistant contemplates compensation based on ~10% operational commission. Financial reporting could later surface gross receipts for easy commission calculation.

---

### 7. 2026-10-20 03:47:25 PM (`WhatsApp Ptt 2026-10-20 at 3.47.25 PM.ogg`)

#### Alcance de la demostración para la cliente (MVP vs. Producto terminado)
> *"Por eso me estaba sintiendo así de hoy qué le muestro, qué quiero hablar con Andrés, porque no es como que tenemos que tener todo listo..."*

**Key Takeaways & Requirements:**
- **Expectation Management:** Clearly delineate demo capabilities so Jazmín evaluates core booking/payment flow without expecting complete historical data migration on day one.

---

### 8. 2026-10-20 03:51:03 PM (`WhatsApp Ptt 2026-10-20 at 3.51.03 PM.ogg`)

#### Dolor operativo clave del cliente (Tercerización de cobranzas y pagos)
> *"La segunda [razón] es pues que yo sé que ella se siente así como que: 'ay, ya quiero dejar yo de manejar lo de los pagos y como poderlo tercerizar'..."*

**Key Takeaways & Requirements:**
- **Core Value Proposition:** Offloading collections and payment verification from the therapist to the assistant is the single biggest value driver for adopting the software.

---

## Synthesis Matrix: Operational Need to Engineering Requirement

| # | Operational Friction | Proposed Solution | Target Component |
|---|----------------------|-------------------|------------------|
| 1 | Prórrogas forgotten after commitment date | Add `due_date` & filter for pending/expired extensions | `armonia-api` / `armonia-dashboard` |
| 2 | "Buffer" term confusion | Rename to *"Tiempo de descanso"* / *"Margen entre citas"* | `armonia-dashboard` UI copy |
| 3 | Payment fraud risk / lack of bank audit | 2-step verification: `voucher_received` -> `reconciled` | `armonia-api` (Payments) & Dashboard View |
| 4 | Clinical history inaccessible prior to session | Client Dossier with consultation notes timeline | `armonia-dashboard` (`PatientDetailView`) |
| 5 | Role confusion between operator & owner | Role-based routing (Operator Intake vs Owner Audit) | `armonia-dashboard` Auth & Navigation |


---

## User Stories & Acceptance Criteria

### Story 1: Tracking Payment Extension Deadlines (Prórrogas con Fecha Límite)
* **As an:** Operador / Asistente
* **I want to:** Asignar una fecha de compromiso de pago (`due_date`) al registrar una prórroga (`DELAYED_ALLOWED`) y ver una lista filtrable de prórrogas por vencer o vencidas en el dashboard
* **So that:** Pueda dar seguimiento puntual al consultante sin depender de mi memoria o de notas manuales en papel.
* **Acceptance Criteria:**
  - **Given** una cita en estado `RESERVED` con modalidad de pago diferido (`DELAYED_ALLOWED`),
  - **When** el operador registra la excepción,
  - **Then** el sistema exige ingresar una `due_date` (fecha prometida de pago) obligatoria.
  - **Given** que la fecha actual es igual o posterior a la `due_date` y el pago sigue pendiente,
  - **Then** la cita se resalta visualmente en una vista dedicada *"Prórrogas por cobrar"* con indicador de días restantes o días de retraso.

---

### Story 2: Two-Step Payment Verification & Audit (Acreditación y Conciliación Bancaria)
* **As an:** Dueña del negocio (Jazmín)
* **I want to:** Disponer de un flujo de verificación en dos pasos (`voucher_received` -> `reconciled` o `voucher_invalid`)
* **So that:** Mi asistente pueda habilitar citas al recibir comprobantes de WhatsApp, mientras yo mantengo una vista periódica para auditar que el dinero realmente ingresó a mi cuenta de banco.
* **Acceptance Criteria:**
  - **Given** un consultante que envía su comprobante por WhatsApp,
  - **When** el operador lo valida,
  - **Then** marca el estado del pago como `VOUCHER_RECEIVED` (registrando operador y hora), habilitando la sesión sin requerir la intervención inmediata de Jazmín.
  - **Given** una lista de pagos en estado `VOUCHER_RECEIVED`,
  - **When** Jazmín revisa su estado de cuenta bancario desde su vista de auditoría,
  - **Then** puede marcar individualmente cada pago como `RECONCILED` (confirmado) o `VOUCHER_INVALID` (rechazado con nota explicativa).

---

### Story 3: Patient Dossier & Consultation History (Historial de Notas por Paciente)
* **As a:** Terapeuta principal (Jazmín)
* **I want to:** Acceder al perfil o expediente del consultante para revisar el historial cronológico de citas pasadas y sus notas de sesión
* **So that:** Antes de iniciar cada consulta sepa exactamente qué temas se trabajaron, qué tareas quedaron pendientes y cuál ha sido la evolución del paciente.
* **Acceptance Criteria:**
  - **Given** que una cita se marca como `COMPLETED` con una nota de sesión registrada,
  - **When** Jazmín hace clic en el nombre del paciente en el calendario o lista de citas,
  - **Then** el sistema navega a la vista de detalle del consultante mostrando la cronología de todas sus sesiones y notas.
  - **Security / Privacy Rule:** Las notas clínicas de sesión solo son legibles y editables por el rol `therapist` (Jazmín). El rol `operator` (asistente) solo puede visualizar datos demográficos, citas agendadas y estado de pagos.

---

### Story 4: Terminology Localization ("Buffer" -> "Tiempo de Descanso")
* **As an:** Operador o administrador del consultorio
* **I want to:** Leer términos en lenguaje natural del dominio terapéutico en lugar de tecnicismos en inglés
* **So that:** La curva de aprendizaje sea nula y no exista confusión conceptual durante la configuración de servicios y agendas.
* **Acceptance Criteria:**
  - **Given** cualquier vista de configuración o detalle de servicios en el dashboard,
  - **When** se presenta el intervalo de separación entre consultas,
  - **Then** el texto mostrado debe ser *"Tiempo de descanso entre sesiones"* o *"Margen entre citas"*, eliminando por completo la palabra técnica *"Buffer"*.

---

### Story 5: Role-Based Dashboard Navigation (Separación de Vistas Asistente vs. Terapeuta)
* **As an:** Asistente operativo (`operator`)
* **I want to:** Tener un menú de navegación simplificado enfocado en agendamiento, registro de comprobantes y enlaces de WhatsApp
* **So that:** Mi interfaz sea ágil, libre de ruido analítico o clínico, y escale la conciliación bancaria a Jazmín.
* **Acceptance Criteria:**
  - **Given** una sesión iniciada con rol `operator`,
  - **When** navega por el dashboard,
  - **Then** solo ve módulos de: *Agenda / Calendario*, *Registro de Citas*, *Bandeja de Prórrogas*, y *Acreditación de Comprobantes*.
  - **Given** una sesión iniciada con rol `therapist`,
  - **Then** se habilitan adicionalmente: *Expedientes Clínicos*, *Conciliación Bancaria*, y *Métricas Financieras*.
