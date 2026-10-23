# Primeras Impresiones del Despliegue en Homelab: Retroalimentación de la Operación

**Fecha de Registro:** 20-10-2026  
**Contexto:** Despliegue en Homelab (`armonia.roadtotech.me`) como prueba de concepto viva y demostración exploratoria previa a la implementación de Google OAuth y el despliegue distribuido en producción (Google Cloud Run / Cloudflare Pages / Turso DB).  
**Fuente:** Intercambio directo y retroalimentación de la asistente de Jazmín tras explorar la demo en vivo de Homelab.  
**Propósito:** Documentar observaciones operativas, puntos de fricción y expectativas de usuario, traduciendo la experiencia de uso en requerimientos de ingeniería e historias de usuario ágiles.

---

## Resumen Ejecutivo y Hallazgos Clave

1. **Delegación de Pagos y Verificación en Dos Pasos:** El mayor cuello de botella operativo y de carga mental para Jazmín es el seguimiento y cobro de citas. La asistente puede actuar como primera línea de registro (acreditando comprobantes recibidos), pero el sistema debe ofrecer una **verificación en dos pasos** (*Asistente acredita comprobante* $\to$ *Jazmín concilia contra cuenta bancaria*) sin requerir mensajes constantes de confirmación por WhatsApp.
2. **Ciclo de Vida de Prórrogas y Compromisos:** Otorgar una prórroga de pago sin una alerta automatizada o recordatorio visual en calendario genera incertidumbre respecto a cuentas por cobrar olvidadas.
3. **Expediente e Historial de Sesiones por Paciente:** Concluir una cita y redactar notas debe alimentar un **Expediente del Paciente**, donde Jazmín pueda consultar el historial clínico y notas de sesiones anteriores antes de iniciar futuras consultas.
4. **Accesibilidad del Vocabulario:** La terminología técnica en inglés (como `"Buffer"`) debe reemplazarse por conceptos naturales del consultorio (ej. *"Tiempo de descanso entre sesiones"* o *"Margen entre citas"*).
5. **Redacción de Mensajes de WhatsApp:** Las plantillas de mensajes automáticos y pre-llenados para WhatsApp recibieron excelente recepción por proyectar una imagen profesional y ahorrar tiempo de redacción manual.

---

## Puntos de Fricción Operativa y Brechas Identificadas

Tras analizar la retroalimentación recibida durante la exploración práctica de la demo en vivo, se identificaron brechas claras en el soporte operativo de la plataforma. Las principales áreas de mejora se concentran en:

* **Falta de Seguimiento a Promesas de Pago:** Al otorgar una prórroga de pago a un consultante, el sistema no registra una fecha límite formal con recordatorios proactivos, dependiendo enteramente de la memoria del operador.
* **Separación entre Recepción y Conciliación Bancaria:** Aunque la asistente puede validar rápidamente un comprobante enviado por WhatsApp, la dueña del negocio (Jazmín) no cuenta con una bandeja de auditoría para verificar periódicamente que los fondos se hayan acreditado efectivamente en su banco antes o después de la sesión.
* **Discontinuidad en el Contexto Clínico:** Marcar una cita como completada permite guardar notas, pero no existe una vista de expediente donde Jazmín pueda consultar la evolución histórica y acuerdos de tareas antes de entrar a sesión.
* **Tecnicismos en la Interfaz:** Términos de desarrollo como *"Buffer"* generan confusión en operadores no técnicos.
* **Mezcla de Roles:** Las tareas de recepción y logística (agendar, registrar comprobantes, enlaces de WhatsApp) se mezclan con las funciones del terapeuta/administrador (conciliación bancaria, notas clínicas privadas, métricas), en lugar de presentar vistas diferenciadas por rol.

Para convertir estas observaciones en entregables concretos de ingeniería, cada necesidad se formaliza a continuación como una historia de usuario ágil con criterios de aceptación verificables (BDD/Gherkin) para los próximos ciclos de desarrollo vertical.

---

## Matriz de Síntesis: Necesidad Operativa vs. Requerimiento Técnico

| # | Fricción Operativa | Solución Propuesta | Componente Destino |
|---|--------------------|-------------------|--------------------|
| 1 | Prórrogas olvidadas tras la fecha pactada | Añadir `due_date` y filtro de prórrogas pendientes/vencidas | `armonia-api` / `armonia-dashboard` |
| 2 | Confusión con el término "Buffer" | Renombrar a *"Tiempo de descanso"* / *"Margen entre citas"* | Interfaz de `armonia-dashboard` |
| 3 | Riesgo de comprobantes no válidos / falta de auditoría bancaria | Verificación en 2 pasos: `voucher_received` $\to$ `reconciled` | `armonia-api` (Pagos) y Vista de Dashboard |
| 4 | Historial clínico inaccesible antes de consulta | Expediente del Consultante con cronología de notas | `armonia-dashboard` (`PatientDetailView`) |
| 5 | Mezcla de funciones entre asistente y dueña | Navegación por roles (Recepción Asistente vs. Auditoría Terapeuta) | Permisos y Rutas de `armonia-dashboard` |

---

## Historias de Usuario y Criterios de Aceptación

### Historia 1: Seguimiento a Vencimientos de Prórroga
* **Como:** Operador / Asistente
* **Quiero:** Asignar una fecha límite obligatoria (`due_date`) al registrar una prórroga (`DELAYED_ALLOWED`) y contar con una lista filtrable de prórrogas por vencer o vencidas en el panel
* **Para:** Dar seguimiento oportuno al consultante sin depender de mi memoria ni de apuntes sueltos en papel.
* **Criterios de Aceptación:**
  - **Dado** que se registra una cita con modalidad de pago diferido (`DELAYED_ALLOWED`),
  - **Cuando** el operador autoriza la prórroga,
  - **Entonces** el sistema solicita de forma obligatoria la fecha prometida de pago (`due_date`).
  - **Dado** que la fecha actual es igual o posterior a la `due_date` y el pago sigue pendiente,
  - **Entonces** la cita se destaca visualmente en una bandeja dedicada de *"Prórrogas por Cobrar"* indicando los días restantes o de retraso.

---

### Historia 2: Verificación de Pagos en Dos Pasos (Acreditación y Conciliación)
* **Como:** Dueña del consultorio (Jazmín)
* **Quiero:** Un flujo de validación en dos etapas (`voucher_received` $\to$ `reconciled` o `voucher_invalid`)
* **Para:** Que mi asistente habilite citas al recibir comprobantes por WhatsApp, mientras yo conservo una vista periódica para auditar que el dinero ingresó formalmente a mi cuenta bancaria.
* **Criterios de Aceptación:**
  - **Dado** que un consultante envía su comprobante por WhatsApp,
  - **Cuando** la asistente lo valida en ventanilla,
  - **Entonces** cambia el estado del pago a `VOUCHER_RECEIVED` (registrando fecha y operador), habilitando la sesión sin requerir la intervención inmediata de Jazmín.
  - **Dada** la lista de pagos en estado `VOUCHER_RECEIVED`,
  - **Cuando** Jazmín audita su estado de cuenta bancario desde su vista de conciliación,
  - **Entonces** puede marcar individualmente cada pago como `RECONCILED` (confirmado) o `VOUCHER_INVALID` (rechazado con nota explicativa).

---

### Historia 3: Expediente del Paciente e Historial de Consultas
* **Como:** Terapeuta principal (Jazmín)
* **Quiero:** Acceder al expediente individual de cada consultante para consultar la cronología de sesiones previas y sus notas clínicas
* **Para:** Recordar antes de cada llamada qué temas se abordaron, qué tareas quedaron asignadas y la evolución del proceso terapéutico.
* **Criterios de Aceptación:**
  - **Dada** una cita marcada como completada (`COMPLETED`) con notas de sesión registradas,
  - **Cuando** Jazmín hace clic sobre el nombre del paciente en la agenda o lista de citas,
  - **Entonces** el sistema abre el expediente del consultante desplegando la cronología completa de citas y notas de consulta.
  - **Regla de Privacidad y Confidencialidad:** Las notas clínicas son de acceso exclusivo para el rol `therapist` (Jazmín). El rol `operator` (asistente) solo puede visualizar datos de contacto, citas agendadas y estados de cobranza.

---

### Historia 4: Adaptación del Vocabulario ("Buffer" $\to$ "Tiempo de Descanso")
* **Como:** Operador o administrador del consultorio
* **Quiero:** Leer términos expresados en lenguaje natural de la práctica terapéutica en lugar de tecnicismos en inglés
* **Para:** Evitar dudas conceptuales y agilizar la configuración de servicios y horarios de agenda.
* **Criterios de Aceptación:**
  - **Dada** cualquier pantalla de configuración o catálogo de servicios en el panel,
  - **Cuando** se visualiza el tiempo de separación entre consultas consecutivas,
  - **Entonces** la etiqueta visible debe ser *"Tiempo de descanso entre sesiones"* o *"Margen entre citas"*, omitiendo por completo el término técnico *"Buffer"*.

---

### Historia 5: Navegación Diferenciada por Rol (Asistente vs. Terapeuta)
* **Como:** Asistente operativa (`operator`)
* **Quiero:** Un menú de navegación enfocado exclusivamente en agendamiento, registro de comprobantes y enlaces de WhatsApp
* **Para:** Trabajar en una interfaz limpia, sin distracciones de análisis financiero ni notas clínicas que no corresponden a mi labor.
* **Criterios de Aceptación:**
  - **Dada** una sesión activa con rol `operator`,
  - **Cuando** visualiza el menú del panel,
  - **Entonces** solo tiene acceso a: *Agenda / Calendario*, *Registro de Citas*, *Bandeja de Prórrogas* y *Acreditación de Comprobantes*.
  - **Dada** una sesión activa con rol `therapist`,
  - **Entonces** se habilitan adicionalmente las secciones de: *Expedientes Clínicos*, *Conciliación Bancaria* y *Reportes Financieros*.
