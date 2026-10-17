# 008 — Detalle de cita en la agenda de recepción

Estado: implementado
Fase: 3 (citas, interfaz)
Depende de: 007 (agenda de `/recepcion/agenda` con `CalendarioCitas.vue`)

## Contexto
Hoy la agenda (spec 007) muestra las citas y permite arrastrarlas, pero al hacer clic en una no pasa nada. Este spec agrega un detalle de solo lectura que se abre al hacer clic en una cita: el popover de `docs/diseno/pantallas/AgendaRecepcion.dc.html` (líneas 148-166). Se reutilizan los stores de citas, médicos, pacientes y consultorios, `IconoAlergia.vue`, `nombreCompleto` de `utils/formato.js` y el `Popover` de PrimeVue. Las etiquetas de estado, que hoy viven dentro de `CalendarioCitas.vue`, pasan a `utils/formato.js` para compartirlas (DRY de `CLAUDE.md`).

## Requisitos
- RF1. Al hacer clic en una cita del calendario (vistas de semana y de día, con o sin filtro de médico), se abre un detalle anclado a esa cita. Se abre para citas en **cualquier** estado. Solo hay un detalle abierto a la vez: un clic en otra cita lo reemplaza.
- RF2. Contenido, en el orden de la maqueta:
  - Título: nombre completo del paciente.
  - Si el paciente tiene alergias: aviso con `IconoAlergia` y el texto "Alérgico a …" ("Alérgica a …" si `sexo` es `F`), con todas las alergias separadas por coma. Sin alergias, no hay aviso.
  - Horario: día abreviado, fecha `DD/MM` y rango de horas, por ejemplo "Sáb 26/09 · 10:30–11:00". El fin se calcula con `duracionMin`.
  - Médico (`nombreCompleto`, sin "Dr./Dra.", porque el modelo no guarda ese dato), consultorio de la cita ("nombre · Piso n"), motivo ("Sin motivo" si está vacío) y estado.
  - El estado se muestra **con texto** ("Programada", "Confirmada", "Atendida", "Cancelada", "No asistió"), no solo por color.
- RF3. La nota "También puede arrastrarla a otro horario libre." solo aparece en citas `programada` o `confirmada` (las que se pueden arrastrar, según el spec 007). El detalle no tiene botones de acción.
- RF4. Si el paciente, el médico o el consultorio de la cita no existen, se muestran "Paciente desconocido", "Médico desconocido" o "Consultorio desconocido", y el detalle se abre igual.
- RF5. Soltar una cita tras arrastrarla no abre el detalle, ni si la reprogramación sale bien ni si se revierte.
- RF6. Accesibilidad:
  - El detalle es un diálogo con nombre accesible igual al título (paciente).
  - Se cierra con el botón "Cerrar" (`aria-label`), con `Escape` o con un clic fuera.
  - Con teclado, cada cita es alcanzable con `Tab` y se abre con `Enter`. Al abrir, el foco pasa al detalle; al cerrar con el botón o con `Escape`, vuelve a la cita.
- RF7. Abrir el detalle es solo lectura: no modifica ningún dato.
- RF8. La agenda también carga los consultorios. Si esa carga falla, se muestra el mismo estado de error de carga del spec 007.

## Criterios de aceptación
- [ ] CA1. Dado recepción con los datos semilla, cuando hace clic en una cita de un paciente alérgico a la penicilina, entonces ve un detalle con su nombre como título, el aviso "Alérgico a penicilina" (o "Alérgica…" si es mujer), el horario en formato "Día DD/MM · HH:mm–HH:mm", el médico, el consultorio con su piso, el motivo y el estado en texto.
- [ ] CA2. Dada una cita de un paciente sin alergias, cuando se abre su detalle, entonces no aparece ningún aviso de alergias.
- [ ] CA3. Dada una cita `cancelada`, `atendida` o `no_asistio`, cuando se hace clic en ella, entonces el detalle se abre con su estado en texto, sin la nota de arrastre y sin botones de acción. En una `programada` o `confirmada`, la nota sí aparece.
- [ ] CA4. Cuando se arrastra una cita y se suelta en otra franja, entonces no se abre ningún detalle, tanto si se guarda como si vuelve a su lugar por una regla del servicio.
- [ ] CA5. Con un detalle abierto, cuando se pulsa "Cerrar", `Escape` o se hace clic fuera, entonces se cierra. Con el botón o con `Escape`, el foco vuelve a la cita.
- [ ] CA6. Solo con teclado: cuando se llega con `Tab` a una cita y se pulsa `Enter`, entonces se abre su detalle con el foco dentro, y un lector de pantalla lo anuncia como diálogo con el nombre del paciente.
- [ ] CA7. Con un detalle abierto, cuando se hace clic en otra cita, entonces queda abierto solo el de la nueva cita. Abrir y cerrar detalles no cambia `clinica_citas`.
- [ ] CA8. Con el filtro en un médico, o en la vista de día, cuando se hace clic en una cita visible, entonces su detalle se abre igual que en la vista de semana sin filtro.

## Tareas
- [x] T1. modificar `SETUP.md`: en el árbol, agregar `DetalleCita.vue` en `components/citas/` y `ETIQUETAS_ESTADO_CITA` y `formatearHorarioCita` al comentario de `utils/formato.js`. En "Estado actual", indicar que el spec 008 agrega el detalle de cita.
- [x] T2. modificar `src/utils/formato.js`: agregar `ETIQUETAS_ESTADO_CITA` (estado → etiqueta con mayúscula, RF2) `formatearHorarioCita(cita)` ("Sáb 26/09 · 10:30–11:00", RF2) y `ESTADOS_MOVIBLES_CITA` (RF3, compartida con el calendario).
- [x] T3. crear `src/components/citas/DetalleCita.vue`: `Popover` de PrimeVue con el contenido de RF2 a RF4, según la maqueta (líneas 148-165, sin los botones). Recibe la cita y su paciente, médico y consultorio ya resueltos. Se abre anclado al elemento de la cita y cumple RF6.
- [x] T4. modificar `src/components/citas/CalendarioCitas.vue`: nueva prop `consultorios`. Maneja el clic y el `Enter` en una cita para abrir `DetalleCita`, sin abrirlo al soltar un arrastre (RF1, RF5, RF6). Usa `ETIQUETAS_ESTADO_CITA` y `ESTADOS_MOVIBLES_CITA` en lugar de sus constantes locales.
- [x] T5. modificar `src/views/recepcion/AgendaView.vue`: cargar `useConsultoriosStore`, incluir su error en el error de carga y pasar la lista a `CalendarioCitas` (RF8).

## Fuera de alcance
- Botones "Reprogramar" y "Cancelar cita…" del popover de la maqueta, la cancelación en dos pasos y el formulario o botón de "Nueva cita": quedan para un spec posterior. Reprogramar sigue siendo por arrastre (spec 007).
- Cambiar el estado de la cita desde el detalle (confirmar, marcar no asistió).
- Enlaces del detalle a la ficha del paciente o del médico.
- `/medico/agenda` (fase 4).
- Tests (fase 8).
