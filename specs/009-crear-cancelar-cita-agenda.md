# 009 — Crear y cancelar citas desde la agenda de recepción

Estado: implementado
Fase: 3 (citas, interfaz)
Depende de: 006 (`citasService` y `useCitasStore`), 007 (`CalendarioCitas.vue` en `/recepcion/agenda`), 008 (`DetalleCita.vue`)

## Contexto
La agenda de recepción ya muestra, filtra y reprograma citas por arrastre (007), y abre un detalle de solo lectura al hacer clic (008). El servicio y el store ya exponen `crear` y `cancelar` con todas las reglas de negocio (`crearCita`, `cancelarCita`, `validarCita`): este spec no toca servicio, store ni utils. Solo agrega la interfaz: el diálogo "Nueva cita" y la acción "Cancelar cita…" del detalle.
Maqueta: `docs/diseno/pantallas/AgendaRecepcion.dc.html` (botón de cabecera, línea 62; hueco "Nueva cita", línea 140; botones del detalle, líneas 162-163). El diálogo sigue el patrón de `DialogoFormulario.vue` (el mismo de `MedicoForm.vue`).

## Requisitos
- RF1. La cabecera de la agenda tiene el botón "Nueva cita", que abre un diálogo de alta de cita. Si el filtro de médico tiene un médico elegido, llega preseleccionado.
- RF2. Hacer clic en un hueco libre del calendario abre el mismo diálogo con la fecha prellenada y, en las vistas de semana y día, también la hora del hueco.
- RF3. El diálogo pide paciente (buscable por DNI o nombre), médico, fecha, hora y motivo (opcional). Los errores de `validarCita` se muestran junto a cada campo antes de llamar al store.
- RF4. Al elegir un paciente con alergias, el diálogo muestra el aviso de alergias (`AlertaAlergias.vue`) antes de poder guardar.
- RF5. Al guardar, se llama a `useCitasStore().crear`. Si el servicio rechaza la cita (fuera de horario, solapamiento, fecha pasada), el mensaje se muestra dentro del diálogo, que queda abierto con los datos cargados. Si se crea, el diálogo se cierra, se muestra un Toast de éxito y la cita aparece en el calendario.
- RF6. El detalle de una cita `programada` o `confirmada` muestra el botón "Cancelar cita…"; en los demás estados no aparece.
- RF7. "Cancelar cita…" cierra el detalle y pide confirmación con el `ConfirmDialog` global (confirmación en rojo lleno, según `DESIGN.md`). Al confirmar se llama a `useCitasStore().cancelar`; al rechazar no cambia nada.
- RF8. Tras cancelar, se muestra un Toast de éxito y la cita queda en estado "Cancelada" en el calendario, sin poder arrastrarse. Si el servicio lanza error, se muestra en un Toast de error.

## Criterios de aceptación
- [ ] CA1. Dado el filtro en "Todos los médicos", cuando recepción pulsa "Nueva cita", entonces se abre el diálogo vacío; con un médico filtrado, ese médico aparece preseleccionado.
- [ ] CA2. Dada la vista de semana, cuando recepción hace clic en el hueco del jueves a las 10:30, entonces el diálogo abre con esa fecha y la hora 10:30.
- [ ] CA3. Dado el diálogo sin paciente ni hora, cuando se pulsa guardar, entonces aparecen los errores junto a esos campos y no se crea ninguna cita.
- [ ] CA4. Aviso de alergias antes de agendar (reglas de negocio de `CLAUDE.md`): dado el paciente semilla alérgico a la penicilina, cuando se lo elige, entonces se ve el aviso con esa alergia.
- [ ] CA5. Sin solapamiento, dentro del horario y sin fechas pasadas (reglas de negocio de `CLAUDE.md`): dada una hora que choca con otra cita del mismo médico, o fuera de su horario, o en el pasado, cuando se guarda, entonces el diálogo sigue abierto con el mensaje del servicio y no se crea la cita.
- [ ] CA6. Dados datos válidos, cuando se guarda, entonces el diálogo se cierra, aparece el Toast de éxito y la cita nueva se ve en el calendario en estado "Programada"; al recargar la página sigue ahí.
- [ ] CA7. Dada una cita programada, cuando recepción pulsa "Cancelar cita…" y confirma, entonces aparece el Toast de éxito y la cita se ve como "Cancelada" y ya no se puede arrastrar; si en cambio rechaza la confirmación, la cita no cambia.
- [ ] CA8. Dada una cita atendida, cancelada o "no asistió", cuando se abre su detalle, entonces no aparece el botón "Cancelar cita…".

## Tareas
- [x] T1. modificar `SETUP.md`: en el árbol, agregar `CitaForm.vue` en `components/citas/`. En "Estado actual", indicar que el spec 009 agrega crear y cancelar citas desde la agenda.
- [x] T2. crear `src/components/citas/CitaForm.vue`: diálogo de alta sobre `DialogoFormulario` (RF3 a RF5). Recibe visibilidad y valores iniciales (médico, fecha, hora), valida con `validarCita`, llama a `crear` del store y muestra su error. Emite el aviso de cita creada para que la vista muestre el Toast.
- [x] T3. modificar `src/components/citas/DetalleCita.vue`: botón "Cancelar cita…" (RF6) según la maqueta (línea 163), visible solo si el estado está en `ESTADOS_MOVIBLES_CITA`; emite `cancelar`.
- [x] T4. modificar `src/components/citas/CalendarioCitas.vue`: clic en un hueco libre emite `nueva-cita` con `{ fecha, hora }` (hora `null` en la vista de mes) (RF2). Al recibir `cancelar` de `DetalleCita`, cierra el detalle y reemite `cancelar` con la cita (RF7).
- [x] T5. modificar `src/views/recepcion/AgendaView.vue`: botón "Nueva cita" en la cabecera (RF1), abrir `CitaForm` desde el botón y desde `nueva-cita`, Toast de alta (RF5). Manejar `cancelar`: confirmación con `useConfirm` con el mismo patrón que `useEliminarPaciente.js`, llamada a `cancelar` y Toasts (RF7, RF8).

## Fuera de alcance
- Botón "Reprogramar" del detalle (maqueta, línea 162): la reprogramación sigue siendo por arrastre (007).
- Indicador visual del hueco al pasar el mouse ("Nueva cita 12:30" punteado, maqueta línea 140): el clic funciona sin él.
- Cambiar el estado a `confirmada` o `no_asistio`, y editar el paciente, el médico o el motivo de una cita existente.
- Mostrar solo las horas libres del médico en el diálogo: la validación la hace el servicio al guardar.
- Cambios en `citasService.js`, `stores/citas.js` o `utils/validaciones.js`.

## Decisiones abiertas
- RF2 (clic en un hueco para crear) viene de la maqueta, no del requerimiento. Si se prefiere solo el botón de cabecera, se quitan RF2, CA2 y la primera mitad de T4.
- El diálogo de cita no tiene maqueta propia (`docs/diseno/README.md`, "Pendiente de diseño"). Se propone seguir el patrón de "Nuevo paciente" a través de `DialogoFormulario`, sin leer `PacienteNuevo.dc.html`. Si hace falta alinearlo con esa maqueta, pido permiso para leer `docs/diseno/pantallas/PacienteNuevo.dc.html`.
