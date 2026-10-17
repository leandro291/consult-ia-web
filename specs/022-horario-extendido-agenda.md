# 022 — Horario extendido de la agenda (hasta las 22:00)

Estado: implementado
Fase: 3
Depende de: 007 (Agenda con FullCalendar), 003 (Catálogos de médicos)

## Contexto
El calendario de citas (agenda de recepción y "Mi agenda") corta en 14:00 fijo (`src/components/citas/CalendarioCitas.vue`), y la leyenda dice "Lun–Sáb 08:00–14:00" (`src/components/citas/LeyendaCitas.vue`), así que se ocultan citas de médicos con horario más largo. Además, los médicos semilla terminan a las 14, 15 y 16 h, así que crear una cita por la tarde falla con "La cita está fuera del horario de atención del médico.". Este spec extiende la vista y los horarios semilla hasta las 22:00, y sube la versión de la semilla para que los navegadores con datos v1 la vuelvan a cargar una vez (se acepta que se reinicien los datos de demo). No cambian las reglas de "no en el pasado", de horario ni de solapamiento (`CLAUDE.md` → Reglas de negocio).

## Requisitos
- RF1. El calendario de citas muestra franjas desde las 08:00 hasta las 22:00 en la agenda de recepción y en "Mi agenda".
- RF2. La leyenda de la agenda muestra el horario de atención "Lun–Sáb 08:00–22:00".
- RF3. Los 3 médicos semilla atienden hasta las 22:00. Su hora de inicio y sus días no cambian (Montoya sigue lunes, miércoles y viernes).
- RF4. Al crear un médico nuevo, el formulario propone 22:00 como hora de fin por defecto.
- RF5. La versión de la semilla pasa de 1 a 2. Al iniciar, la app carga la semilla si no hay versión guardada **o** si la versión guardada es distinta de la actual. Si la versión guardada coincide, no toca los datos.
- RF6. Las citas semilla se siguen generando dentro del horario de cada médico y sin solapamientos. La fórmula actual (inicio + `paso % 4` h) llega como máximo a las 13:30, así que no hace falta cambiarla.

## Criterios de aceptación
- [ ] CA1. Dado un usuario en la agenda de recepción o en "Mi agenda" (vista semana o día), cuando mira el calendario, entonces la última franja visible es 21:30–22:00 y se ven las citas entre las 14:00 y las 22:00.
- [ ] CA2. Dado un día hábil y una hora futura libre a las 17:00, cuando recepción crea una cita a esa hora con cualquier médico semilla, en un día en que ese médico atiende, entonces la cita se guarda sin error de horario.
- [ ] CA3. Dada una cita a las 22:00 o a una hora anterior al inicio del médico, cuando se intenta guardar, entonces sigue apareciendo "La cita está fuera del horario de atención del médico." (la regla de horario de `CLAUDE.md` sigue vigente).
- [ ] CA4. Dada una cita que se solapa con otra del mismo médico, o una fecha y hora pasadas, cuando se intenta crear, entonces el rechazo es igual que antes de este spec.
- [ ] CA5. Dada la agenda, cuando se mira la leyenda, entonces dice "Lun–Sáb 08:00–22:00".
- [ ] CA6. Dado un `localStorage` con `clinica_version = 1` y datos modificados, cuando se abre la app, entonces se recargan los datos semilla y `clinica_version` pasa a 2. Al volver a abrir la app, los datos no se vuelven a pisar.
- [ ] CA7. Dado el formulario "Nuevo médico", cuando se abre, entonces la hora de fin propuesta es 22:00.
- [ ] CA8. `npm run lint`, `npm run build` y `npm test` terminan sin errores.

## Tareas
- [x] T1. modificar `src/components/citas/CalendarioCitas.vue`: `slotMaxTime` a `'22:00:00'`.
- [x] T2. modificar `src/components/citas/LeyendaCitas.vue`: texto del horario "Lun–Sáb 08:00–22:00".
- [x] T3. modificar `src/data/seed.js`: `VERSION_SEMILLA = 2` y `horario.fin: '22:00'` en los 3 médicos (sin tocar `inicio` ni `dias`).
- [x] T4. modificar `src/components/medicos/MedicoForm.vue`: `fin` por defecto `'22:00'`.
- [x] T5. modificar `src/services/semillaService.js`: `inicializarDatos` carga la semilla cuando la versión guardada `!== VERSION_SEMILLA` (esto incluye el caso sin versión) y se actualiza el comentario. Al recargar por cambio de versión (no en el primer arranque) limpia también la sesión (`guardar(CLAVES.sesion, null)`, como `authService`), para que el guard mande a `/login`.
- [x] T6. modificar `tests/data/seed.test.js`: la expectativa de `VERSION_SEMILLA` pasa de 1 a 2.
- [x] T7. modificar `tests/services/semillaService.test.js`: la versión esperada tras cargar pasa a ser `VERSION_SEMILLA`, se mantiene el caso "no pisa datos si la versión coincide" y se agrega un caso para "recarga si la versión guardada es distinta" (CA6).
- [x] T8. modificar `src/services/citasService.js` (`validarAgenda`): el error de horario conserva el prefijo "La cita está fuera del horario de atención del médico" y agrega el rango y la última hora de inicio posible (`fin` − `duracionMin`, con Day.js), p. ej. "La cita está fuera del horario de atención del médico (08:00–22:00; la última cita puede empezar a las 21:30)." En `tests/services/citasService.test.js`, agregar un caso con el mensaje completo para `13:31` (horario 08:00–14:00 → "13:30"). La regla no cambia: la cita debe terminar dentro del horario.

## Fuera de alcance
- Cambiar las reglas de "no en el pasado", de horario de atención o de solapamiento.
- Que el rango del calendario o la leyenda se calculen a partir del horario real de cada médico.
- Migrar los datos del usuario conservando sus cambios: la recarga reemplaza las colecciones de demo.
- Cambiar la distribución de horas de las citas semilla.

## Decisiones tomadas
- Sesión obsoleta tras la recarga: la semilla regenera ids, así que al recargar por cambio de versión se limpia `clinica_sesion` (incluido en T5). En el primer arranque no hay sesión que limpiar.
