# 010 — Panel del día de recepción (dashboard)

Estado: implementado
Fase: 6 (dashboard con indicadores), adelantada; ver "Decisiones abiertas"
Depende de: 004 (pacientes), 003 (médicos y consultorios), 006 (`useCitasStore`), 009 (`CitaForm.vue`)

## Contexto
`/recepcion/dashboard` hoy muestra `EnConstruccionView.vue`. Este spec agrega el "Panel del día": los conteos de las citas de hoy por estado, que también sirven de filtro, y la tabla con esas citas.
**No se tocan `citasService.js` ni `stores/citas.js`.** `useCitasStore().lista` ya trae todas las citas con `fecha` y `estado`. Filtrar las de hoy y contarlas es un cálculo de presentación en la vista. Se reutilizan `PaginaListado.vue`, `CitaForm.vue` y `ChipAlergia.vue`.
Maqueta: `docs/diseno/pantallas/RecepcionLayout.dc.html` (cabecera, líneas 61-74; pestañas con conteos, 79-85; tabla, 88-157; pie, 158-161). Regla de `DESIGN.md` → Navigation: los conteos son pestañas-filtro de la tabla, no tarjetas de métricas.

## Requisitos
- RF1. La cabecera muestra el título "Panel del día", el día de la semana y la fecha de hoy, y el botón "Nueva cita".
- RF2. La tabla lista las citas cuya fecha es hoy, ordenadas por hora, con estas columnas: hora; paciente, con un chip por cada alergia; médico con su especialidad; consultorio y piso; y estado.
- RF3. Sobre la tabla hay cinco pestañas-filtro, cada una con su número y su etiqueta: Todas, Programadas, Atendidas, Canceladas y No asistió. "Programadas" suma las citas en estado `programada` y `confirmada`. Los números siempre cuentan todas las citas de hoy, sin importar qué filtro esté activo.
- RF4. Al pulsar una pestaña, la tabla muestra solo las citas de ese estado y la pestaña queda marcada como activa (`aria-pressed`). Al entrar a la pantalla, la pestaña activa es "Todas".
- RF5. Debajo de la tabla, un pie muestra "Mostrando N de M citas" (N son las filas visibles; M, el total de hoy) y el enlace "Ver agenda completa", que lleva a `/recepcion/agenda`.
- RF6. El estado de cada cita se muestra con el chip de estado que define `DESIGN.md` (Chips). Ese mismo chip pasa a usarse en el detalle de cita de la agenda (`DetalleCita.vue`), para no duplicar estilos.
- RF7. "Nueva cita" abre `CitaForm` con la fecha de hoy ya cargada. Si la cita se crea, aparece un Toast "Cita creada." y, si es para hoy, la cita entra en la tabla y en los conteos sin recargar la página.
- RF8. Si no hay citas para mostrar, se ve el estado vacío de `PaginaListado` y las pestañas siguen visibles (con los conteos en 0 cuando no hay ninguna cita hoy).

## Criterios de aceptación
- [ ] CA1. Dado que hay citas de hoy y de otros días, cuando recepción entra a `/recepcion/dashboard`, entonces la tabla muestra solo las de hoy, ordenadas por hora, con hora, paciente, médico con especialidad, consultorio con piso y estado.
- [ ] CA2. Dadas las citas de hoy, entonces "Todas" muestra el total; "Programadas", la suma de programadas y confirmadas; y "Atendidas", "Canceladas" y "No asistió", sus conteos. Las cuatro pestañas de estado suman lo mismo que "Todas".
- [ ] CA3. Dada la pestaña "Atendidas" pulsada, entonces la tabla muestra solo las atendidas, esa pestaña queda marcada, los números de las pestañas no cambian y el pie dice "Mostrando N de M citas". Al volver a "Todas", se ven de nuevo todas.
- [ ] CA4. Dado el paciente semilla alérgico a la penicilina con una cita hoy, entonces su fila muestra el chip de alergia "Penicilina".
- [ ] CA5. Ningún estado indicado solo con color (`DESIGN.md`): cada chip de estado muestra su texto. En la agenda, el detalle de una cita sigue mostrando su estado con el mismo chip.
- [ ] CA6. Dado que no hay citas hoy, entonces las cinco pestañas muestran 0 y se ve el estado vacío con el botón "Nueva cita".
- [ ] CA7. Dado "Nueva cita" pulsado, entonces el diálogo abre con la fecha de hoy. Se aplican las reglas de citas de `CLAUDE.md` (sin solapamiento, dentro del horario, sin fechas pasadas y con aviso de alergias), igual que en la agenda. Al crear una cita para hoy, aparece en la tabla, sube el número de "Todas" y de "Programadas", y se muestra el Toast.
- [ ] CA8. Dado el enlace "Ver agenda completa", cuando se pulsa, entonces se navega a `/recepcion/agenda`.

## Tareas
- [x] T1. modificar `SETUP.md`:
  - En el árbol, agregar `ChipEstadoCita.vue` y `FiltroEstadosCita.vue` en `components/citas/`.
  - Mencionar el slot de pie de `PaginaListado`.
  - En "Estado actual", indicar que el spec 010 agrega el panel del día de recepción.
- [x] T2. crear `src/components/citas/ChipEstadoCita.vue`: chip de estado de cita según `DESIGN.md` (Chips). Recibe el estado y muestra la etiqueta de `ETIQUETAS_ESTADO_CITA` (RF6).
- [x] T3. modificar `src/components/citas/DetalleCita.vue`: reemplazar su chip de estado y sus estilos `.detalle-estado*` por `ChipEstadoCita` (RF6).
- [x] T4. crear `src/components/citas/FiltroEstadosCita.vue`: grupo de cinco pestañas-filtro con número y etiqueta. Recibe los conteos y el filtro activo (`v-model`), y marca la pestaña activa con `aria-pressed` (RF3, RF4). Maqueta, líneas 79-85.
- [x] T5. modificar `src/components/comunes/PaginaListado.vue`: agregar un slot de pie debajo de la tabla, dentro de la hoja (RF5). Las vistas actuales no cambian.
- [x] T6. crear `src/views/recepcion/DashboardView.vue`:
  - Cargar los stores de citas, pacientes, médicos y consultorios.
  - Calcular las citas de hoy, los conteos y la lista filtrada (RF2 a RF4).
  - Armar `PaginaListado`, con `FiltroEstadosCita` en herramientas, las columnas y el pie (RF1, RF5, RF8).
  - Abrir `CitaForm` con la fecha de hoy y mostrar el Toast al crear (RF7).
- [x] T7. modificar `src/router/index.js`: la ruta `dashboard` usa `DashboardView` en lugar de `EnConstruccionView`. `respaldo` sigue como está.

## Fuera de alcance
- El buscador "Buscar por DNI o nombre" de la cabecera (maqueta, líneas 67-71): la búsqueda de pacientes ya está en `/recepcion/pacientes`.
- Los botones "Reprogramar" y "Cancelar" de cada fila (maqueta, líneas 130, 138 y 146): esas acciones quedan en la agenda.
- Gráficos (`chart.js` opcional) y otros indicadores que no sean los conteos por estado del día.
- Cambios en `citasService.js`, `stores/citas.js` o `utils/formato.js`.
- El diseño del menú lateral (spec 005).

## Decisiones abiertas
- **Sí hay maqueta:** según la tabla de `docs/diseno/README.md`, "Panel del día (recepción)" = `RecepcionLayout.dc.html`, que tiene el contenido del dashboard. El spec la sigue a ella y no a un diseño propuesto con Card.
- **Fase:** CLAUDE.md pone el dashboard en la fase 6, y las fases 4 y 5 no están hechas. Hay que confirmar si se adelanta.
- **"Programadas" suma `programada` y `confirmada`**, porque la maqueta tiene cinco pestañas y CLAUDE.md nombra cuatro estados. Alternativa: una sexta pestaña "Confirmadas".
- **"Nueva cita" (RF7, CA7) viene de la maqueta**, no del requerimiento. Si no se quiere, se quitan RF7, CA7 y el `CitaForm` de T6. `PaginaListado` exige el botón, así que habría que hacerlo opcional.
- **Acciones de fila** (reprogramar y cancelar): si se quieren en el panel, conviene un spec aparte que saque a un composable la confirmación de cancelar que hoy vive en `AgendaView.vue`.
