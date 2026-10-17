# 020 — Acciones en el detalle de cita del médico

Estado: implementado
Fase: 4
Depende de: 011 (Mi agenda), 012 (Atender cita), 013 (Historia clínica), 014 (Emitir receta)

## Contexto
En "Mi agenda" (`/medico/agenda`), vista Semana, un clic en una cita abre `src/components/citas/DetalleCitaDialog.vue` (montado por `CalendarioCitas.vue` en modo `soloLectura`), que hoy solo muestra datos y "Cerrar". En la vista Día, `EventoCita.vue` ya ofrece "Atender", "Ver historia" y "Generar receta PDF", y `MiAgendaView.vue` ya resuelve el `consultaId` de cada cita (`citasDelMedico`) y navega con `atender`, `verHistoria` y `generarReceta`. Este spec suma al diálogo el acceso a la historia clínica y una acción según el estado de la cita, reutilizando esa cadena de eventos: no hace falta ninguna función nueva en `consultasService.js` ni en `useConsultasStore`.

**Decisión: la acción depende del `estado` de la cita, no de su fecha.** Lo determinan las reglas de negocio de `CLAUDE.md`: "solo se puede registrar una consulta para una cita en estado `programada` o `confirmada`" y "una receta siempre pertenece a una consulta existente". Una cita `cancelada` o `no_asistio` no tiene ni puede tener consulta. Se verificó que el flujo de atención no restringe por fecha: `atenderCita` (`citasService.js`) y `AtencionView.vue` solo miran el estado; la validación "no puede empezar en el pasado" de `utils/validaciones.js` aplica al agendar y reprogramar, no al atender. Una cita `programada` de un día pasado sigue ofreciendo "Atender cita".

## Requisitos
- RF1. El diálogo ofrece siempre, en cualquier estado de la cita, un acceso "Ver historia clínica" que lleva a `/medico/historia/:pacienteId` con el `pacienteId` de la cita mostrada.
- RF2. Si la cita está `programada` o `confirmada` (`ESTADOS_MOVIBLES_CITA` de `utils/formato.js`, el mismo criterio que `atendible` en la vista Día), el diálogo muestra el botón "Atender cita", que lleva a `/medico/atencion/:citaId`.
- RF3. Si la cita está `atendida` y tiene consulta registrada (`cita.consultaId`, ya resuelto por `MiAgendaView.vue`), el diálogo muestra "Generar receta PDF", que lleva a `/medico/receta/:consultaId` con el id de la consulta, nunca con el de la cita.
- RF4. Si la cita está `atendida` pero no tiene consulta registrada (dato inconsistente), "Generar receta PDF" se muestra deshabilitado junto al texto "No se encontró la consulta de esta cita." y no navega a ninguna parte.
- RF5. Si la cita está `cancelada` o `no_asistio`, el diálogo no muestra ni "Atender cita" ni "Generar receta PDF"; solo "Ver historia clínica" y "Cerrar".
- RF6. El diálogo no navega por su cuenta: emite la acción y `CalendarioCitas.vue` la reemite con los mismos eventos que ya usa `EventoCita.vue` (`atender`, `ver-historia`, `generar-receta`), de modo que `MiAgendaView.vue` navega sin cambios. Al elegir una acción, el diálogo se cierra.

## Criterios de aceptación
- [ ] CA1. Dado un médico en Mi agenda, vista Semana, cuando abre una cita en cualquiera de los cinco estados, entonces ve "Ver historia clínica", y al pulsarlo llega a la Historia clínica del paciente de esa cita.
- [ ] CA2. Dada una cita `programada` o `confirmada` (incluida una de fecha pasada), cuando el médico abre su detalle, entonces ve "Atender cita", y al pulsarlo llega a `/medico/atencion/<id de esa cita>` con el formulario de consulta habilitado.
- [ ] CA3. Dada una cita `atendida` con consulta registrada, cuando el médico pulsa "Generar receta PDF" en el diálogo, entonces llega a Emitir receta de esa consulta (la URL lleva el id de la consulta, no el de la cita) y la pantalla carga el paciente sin error.
- [ ] CA4. Dada una cita `atendida` cuya consulta no existe en `clinica_consultas` (se borra a mano desde las herramientas del navegador), cuando el médico abre su detalle, entonces "Generar receta PDF" está deshabilitado, se lee "No se encontró la consulta de esta cita." y pulsarlo no cambia de pantalla.
- [ ] CA5. Dada una cita `cancelada` o `no_asistio`, cuando el médico abre su detalle, entonces no aparecen "Atender cita" ni "Generar receta PDF".
- [ ] CA6. Regla de `CLAUDE.md`: solo se ofrece registrar consulta (Atender cita) para citas `programada` o `confirmada`.
- [ ] CA7. Regla de `CLAUDE.md`: la receta solo se ofrece ligada a una consulta existente.
- [ ] CA8. La vista Día, la agenda de recepción (popover `DetalleCita.vue`) y la navegación desde `EventoCita.vue` siguen funcionando igual; `npm run lint` y `npm run build` terminan sin errores y los tests existentes pasan.

## Tareas
- [x] T1. modificar `SETUP.md`: agregar `DetalleCitaDialog.vue` (diálogo de detalle de la agenda del médico, vista Semana, con "Ver historia clínica" y la acción según estado) a la lista de `components/citas/` del árbol y mencionar el spec 020 en "Estado actual".
- [x] T2. modificar `src/components/citas/DetalleCitaDialog.vue`: declarar los eventos `atender`, `ver-historia` y `generar-receta`; agregar "Ver historia clínica" y la acción condicional por estado (RF1 a RF5), con el aviso de consulta faltante (RF4); cerrar el diálogo al elegir una acción.
- [x] T3. modificar `src/components/citas/CalendarioCitas.vue`: escuchar los tres eventos de `DetalleCitaDialog` y reemitirlos hacia la vista (RF6), igual que hace con `EventoCita`.

## Fuera de alcance
- Cambios en `MiAgendaView.vue`, `consultasService.js`, `useConsultasStore` o `tests/services/consultasService.test.js`: el `consultaId` ya llega en la cita y la navegación ya existe.
- Cualquier restricción por fecha para atender una cita.
- Acciones en el popover de recepción (`DetalleCita.vue`) y cambios en `EventoCita.vue` (vista Día).
- Marcar una cita como `no_asistio`, confirmarla o cancelarla desde el diálogo del médico.
- Diseño nuevo del diálogo: `docs/diseno/pantallas/MiAgendaMedico.dc.html` solo muestra la vista Día; los botones siguen el estilo de "Atender" y "Ver historia" de esa maqueta y los componentes de PrimeVue ya usados en el diálogo.

## Decisiones abiertas
- Etiqueta del botón de receta: el requerimiento dice "Generar receta"; el spec usa "Generar receta PDF" para coincidir con `EventoCita.vue`, `AtencionView.vue` y `CLAUDE.md`. Confirmar cuál se prefiere.
- `DetalleCitaDialog.vue` y el cambio de `CalendarioCitas.vue` que lo monta están sin commitear en la rama actual y no figuran en ningún spec ni en `SETUP.md` (por eso T1). Confirmar que ese trabajo previo se da por aprobado como base de este spec.
