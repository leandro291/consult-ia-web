# 006 — Citas: servicio y store

Estado: implementado
Fase: 3 (citas, capa de datos)
Depende de: 003, 004

## Contexto
Primer spec de la fase 3: capa de datos de citas, sin interfaz. Hoy `clinica_citas` solo se llena desde `seed.js` (con la constante `DURACION_CITA_MIN` sin exportar) y no hay `citasService` ni store de citas. Se siguen el patrón de `medicosService.js` (validación con `utils/validaciones.js` + `primerError`, lectura de otras claves solo para verificar existencia) y el de `stores/pacientes.js` (reutiliza `useCatalogo`). La agenda con FullCalendar va en el spec 007.

## Requisitos
- RF1. Listar todas las citas guardadas en `clinica_citas`.
- RF2. Crear una cita a partir de `{ pacienteId, medicoId, fecha, hora, motivo }`:
  - El servicio completa `id`, `creadoEn`, `duracionMin: 30`, `estado: 'programada'` y `consultorioId`, que se toma del consultorio actual del médico.
  - Paciente y médico deben existir. `fecha` y `hora` son obligatorias; `motivo` es opcional.
- RF3. Reprogramar una cita cambiando solo `fecha` y `hora`. Conserva id, paciente, médico, consultorio, motivo y estado.
- RF4. Cancelar una cita: pasa a `estado: 'cancelada'`.
- RF5. Solo se reprograman o cancelan citas en estado `programada` o `confirmada`.
- RF6. Reglas al crear y al reprogramar (reglas de citas de `CLAUDE.md`):
  - Sin solapamiento con otra cita del mismo médico en la misma fecha. Dos citas se solapan si sus intervalos `[hora, hora + duracionMin)` se cruzan; citas contiguas (una termina 09:30, otra empieza 09:30) no se solapan.
  - Las citas `cancelada` no cuentan para el solapamiento. Al reprogramar, la cita no se compara consigo misma.
  - La fecha cae en un día de `horario.dias` del médico, y la cita empieza en o después de `horario.inicio` y termina en o antes de `horario.fin`.
  - El inicio de la cita (fecha + hora) no es anterior al momento actual.
- RF7. Toda regla rota lanza `Error` con un mensaje en español que dice cuál regla falló, y no se guarda nada.
- RF8. Las comprobaciones puras (campos obligatorios, fecha no pasada, solapamiento entre dos citas, dentro del horario) viven en `utils/validaciones.js` para que el spec 007 las reutilice. El servicio solo lee datos y las aplica.
- RF9. Store `useCitasStore` con `lista`, `cargando`, `error` y las acciones `cargar`, `crear`, `reprogramar` y `cancelar`. Cada acción devuelve `true`/`false` y deja el mensaje del servicio en `error`, como los stores de catálogo.

## Criterios de aceptación
- [ ] CA1. Dado un paciente y un médico existentes, cuando creo una cita válida para un día y hora futuros dentro de su horario, entonces aparece en `clinica_citas` con `id`, `creadoEn`, `duracionMin: 30`, `estado: 'programada'` y el `consultorioId` del médico. Con un `pacienteId` o `medicoId` inexistente, lanza error y no guarda.
- [ ] CA2. Dado que el médico A tiene una cita `programada` a las 09:00 de una fecha, cuando creo otra para A a las 09:15 esa fecha, entonces lanza error de solapamiento; a las 09:30 se crea bien; y para el médico B a las 09:00 también se crea bien.
- [ ] CA3. Dado que la cita de A a las 09:00 está `cancelada`, cuando creo otra para A a las 09:00, entonces se crea bien.
- [ ] CA4. Dado un médico con horario lunes a viernes de 08:00 a 14:00, cuando creo una cita un sábado, a las 07:30 o a las 13:45 (terminaría 14:15), entonces lanza error de horario; a las 13:30 se crea bien.
- [ ] CA5. Cuando creo o reprogramo una cita para ayer, o para hoy a una hora ya pasada, entonces lanza error de fecha pasada.
- [ ] CA6. Dada una cita de A a las 09:00, cuando la reprogramo a las 09:15 del mismo día sin otras citas cerca, entonces se guarda (no choca consigo misma) y conserva id, paciente, motivo y estado. Si la reprogramo encima de otra cita no cancelada de A, lanza error.
- [ ] CA7. Cuando cancelo una cita `programada` o `confirmada`, entonces queda `cancelada`. Cuando intento cancelar o reprogramar una cita `atendida`, `cancelada` o `no_asistio`, entonces lanza error y no cambia.
- [ ] CA8. Dada una acción del store que viola una regla, cuando se ejecuta, entonces devuelve `false`, `error` tiene el mensaje del servicio, `cargando` vuelve a `false` y `lista` no cambia; si sale bien, devuelve `true` y `lista` refleja el cambio.

## Tareas
- [x] T1. modificar `src/data/seed.js`: exportar `DURACION_CITA_MIN` para que el servicio use la misma constante.
- [x] T2. modificar `src/utils/validaciones.js`: agregar la validación de campos de cita (obligatorios y fecha/hora no pasada, con Day.js como `validarPaciente`) y las comprobaciones puras de solapamiento entre dos citas y de cita dentro del horario de un médico (RF6, RF8).
- [x] T3. crear `src/services/citasService.js`: `listarCitas`, `crearCita`, `reprogramarCita(id, { fecha, hora })`, `cancelarCita(id)` con las reglas de RF2 a RF7. Lee `clinica_pacientes` y `clinica_medicos` solo para verificar existencia y obtener horario y consultorio.
- [x] T4. crear `src/stores/citas.js`: `useCitasStore` sobre `useCatalogo(listarCitas)` (RF9).
- [x] T5. modificar `src/composables/useCatalogo.js`: el comentario de cabecera menciona también el store de citas.
- [x] T6. modificar `SETUP.md`: en el árbol, la descripción de `useCatalogo.js` incluye citas; en "Estado actual", indicar que la fase 3 empezó con el spec 006 (capa de datos de citas).

## Fuera de alcance
- Agenda con FullCalendar, filtro por médico, arrastrar para reprogramar, aviso de alergias y cualquier vista o componente (spec 007).
- Eliminar citas; cambiar paciente, médico o motivo de una cita existente.
- Transiciones a `confirmada`, `atendida` o `no_asistio` (fase 4 y siguientes).
- Tests (fase 8).

## Decisiones abiertas
- `consultorioId` se toma del médico al crear y no cambia al reprogramar. Alternativa: pedirlo como dato de la cita. Se eligió derivarlo porque el médico ya tiene un consultorio asignado.
- `motivo` quedó opcional. Si debe ser obligatorio, se agrega a la validación de T2.
- "Fecha no pasada" se interpreta como "inicio no anterior al momento actual" (bloquea hoy a una hora ya pasada). Si solo debe compararse la fecha, se ajusta CA5.
