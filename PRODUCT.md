# Product

## Platform

web

## Stack

Vue 3 + Vite (JavaScript, sin TypeScript), Vue Router, Pinia y PrimeVue. FullCalendar, Day.js y pdfmake. ESLint y Vitest. Detalle en `SETUP.md`.

## Users

Una sola app con login y dos roles, cada uno con su layout y menú:

- **Recepción / Administración**: gestiona pacientes, médicos, consultorios y la agenda de la clínica. Trabaja en mostrador, con muchas altas, búsquedas por DNI o nombre y reprogramaciones. Usa laptop.
- **Médico**: atiende con el paciente presente, frente a su laptop. Dicta la consulta mientras examina, revisa lo que sugiere la IA, guarda la consulta y emite la receta. Su atención no debe interrumpirse: si la IA falla o está desactivada, todo funciona en modo manual. La pantalla puede ser vista por el paciente.

Uso en laptop y tablet (responsive). La audiencia de la demo son docentes evaluadores.

## Product Purpose

Aplicación web académica para gestionar una clínica con varios consultorios: citas (agenda por médico y consultorio), historias clínicas (consultas por paciente) y recetas médicas en PDF descargable e imprimible. Éxito = flujo completo y claro de agendar, atender y recetar, fácil de explicar en una presentación. Se priorizan código claro y bien organizado sobre optimizaciones.

## Positioning

Función destacada: **Asistente de consulta por voz**, un "escriba clínico con IA". El médico dicta en lenguaje natural y la app prellena el formulario de consulta y un borrador de receta. La IA solo sugiere; el médico siempre revisa y confirma. La verificación de alergias es lógica local, no de la IA, y bloquea con confirmación explícita.

## Operating Context

- Sin backend ni base de datos: toda la persistencia es `localStorage` del navegador, accedida solo desde `src/services/`.
- Datos semilla de demo (consultorios, 3 médicos, ~10 pacientes, ~15 citas, consultas y recetas) y botón "Restablecer datos de demo".
- El asistente de voz usa Web Speech API (`es-PE`, mejor en Chrome o Edge) y la API de Claude vía `fetch`, con la API key guardada en el navegador.
- Respaldo: exportar e importar JSON, sin `iaApiKey`.
- Idioma: todo en español (interfaz, mensajes, comentarios).

## Capabilities and Constraints

- Reglas de negocio: sin citas solapadas por médico, solo dentro del horario de atención y en fechas no pasadas; DNI único de 8 dígitos; consulta solo para citas `programada` o `confirmada`; receta siempre ligada a una consulta; no se eliminan pacientes con consultas; aviso visible de alergias antes de agendar o recetar.
- Estados de cita: `programada`, `confirmada`, `atendida`, `cancelada`, `no_asistio`.
- A la IA nunca se envían nombre, DNI, teléfono, email ni dirección del paciente.
- Componentes PrimeVue obligatorios para listados (`DataTable`), modales (`Dialog`), avisos (`Toast`) y acciones destructivas (`ConfirmDialog`).
- Aviso fijo del asistente: "Sugerencias generadas por IA. Verifique toda la información antes de guardar."
- Limitaciones conocidas (a documentar en el README): datos solo en el navegador, tope de ~5 MB, autenticación simulada con contraseñas en texto plano (no apta para datos médicos reales), sin sincronización, API key visible en el navegador, las sugerencias de la IA pueden contener errores, la detección de alergias usa una tabla simplificada.

## Brand Commitments

Sin marca real definida. Nombre, dirección y teléfono del membrete de los PDFs usan un **placeholder de demo** ("Clínica Demo", dirección y teléfono ficticios), decisión del usuario; cambiarlos cuando exista un nombre real. No hay logo ni assets de marca.

## Evidence on Hand

- Texto de demo de transcripción (en `src/data/seed.js` cuando exista) para probar sin micrófono.
- Un paciente semilla alérgico a la penicilina para demostrar la alerta de alergias.
- No hay testimonios, clientes reales ni métricas; no inventarlos.

## Product Principles

1. **La IA sugiere, el médico decide.** Toda salida de IA es editable, está marcada como "Sugerido por IA" y requiere confirmación antes de guardarse.
2. **Seguridad clínica primero.** Las alergias se avisan antes de agendar o recetar, y el cruce con medicamentos bloquea con confirmación explícita.
3. **La atención nunca depende de la IA.** Sin key, sin conexión o con la IA desactivada, el flujo manual funciona completo.
4. **Privacidad por defecto.** Los datos identificables no salen del navegador hacia la IA.
5. **Claro y explicable.** Un proyecto académico: cada pantalla y regla debe poder explicarse en una demo.

## Accessibility & Inclusion

Objetivo WCAG AA (contraste y navegación por teclado). Estados de error junto a cada campo. La app se usa en laptop y tablet. La pantalla puede estar a la vista del paciente durante la consulta.
