# 007 — Agenda de recepción con FullCalendar

Estado: implementado
Fase: 3 (citas, interfaz)
Depende de: 005, 006 (debe estar implementado antes: aporta `useCitasStore` y las reglas de citas)

## Contexto
Segundo spec de la fase 3: la pantalla `/recepcion/agenda`, que hoy usa `EnConstruccionView`. Muestra todas las citas en un calendario, filtra por médico y reprograma arrastrando. Las reglas (solapamiento, horario y fecha no pasada) ya las aplica `citasService` a través de `useCitasStore.reprogramar` (spec 006); aquí no se duplican. Los paquetes de FullCalendar ya están documentados en `SETUP.md` (instalación "Fase 3 (agenda)" y tabla de dependencias), pero todavía no están instalados. Los nombres se sacan de `useMedicosStore`/`usePacientesStore` y de `nombreCompleto` de `utils/formato.js`. Pantalla de referencia: `docs/diseno/pantallas/AgendaRecepcion.dc.html`.

## Requisitos
- RF1. `/recepcion/agenda` muestra un calendario en español con las citas de `clinica_citas`. Por defecto abre la semana actual, se puede cambiar a vista de día y navegar entre semanas o días.
- RF2. Cada cita ocupa su franja (`fecha` + `hora`, duración `duracionMin`). Muestra la hora y el paciente; el estado y el médico van en el `aria-label` (y el médico también en `title`), y el estado además en la leyenda, no solo por color (reglas de `DESIGN.md`).
- RF3. Hay un filtro de médico con la opción "Todos los médicos" (la inicial) y un elemento por médico. Si se elige un médico, solo se ven sus citas. Al navegar entre fechas, el filtro se mantiene.
- RF4. Una cita `programada` o `confirmada` se puede arrastrar a otra franja para reprogramarla, y se guarda con `useCitasStore.reprogramar`. Las citas en otros estados no se pueden arrastrar. Ninguna cita cambia de duración desde el calendario.
- RF5. Si la reprogramación sale bien, la cita queda en su nueva franja y aparece un `Toast` de éxito. Si el store devuelve `false`, la cita vuelve a su franja original y un `Toast` de error muestra el mensaje del store.
- RF6. Mientras cargan las citas, los médicos y los pacientes, la pantalla muestra un estado de carga. Si falla la carga, muestra el error, como en `docs/diseno/pantallas/EstadosVaciosCarga.dc.html`.
- RF7. Visual fiel a `docs/diseno/pantallas/AgendaRecepcion.dc.html` (leer solo las líneas 56-177, la pantalla), con los tokens existentes de `src/assets`:
  - Cabecera de página: "Agenda" y debajo el día de hoy (ej. "Sábado, 26/09/2026"), con la fecha en la fuente de datos. Sin botón "Nueva cita".
  - Dentro de la hoja, barra de herramientas propia: "Hoy", anterior/siguiente, título del rango, filtro de médico y selector Semana/Día (sin Mes). La toolbar nativa de FullCalendar queda oculta.
  - Encabezado de días "Lun 21", con hoy resaltado y sin domingo. Grilla de 08:00 a 14:00, 48 px por franja de 30 min (como la maqueta).
  - Cada cita en una línea compacta: "HH:mm" + nombre del paciente, con estilo distinto por estado y un icono de triángulo si el paciente tiene alergias. Solo se arrastran `programada`/`confirmada` (RF4).
  - Debajo del calendario, leyenda de estados, alergias y horario de atención.

## Criterios de aceptación
- [ ] CA1. Dado el usuario de recepción con los datos semilla, cuando entra a `/recepcion/agenda`, entonces ve la semana actual en español con las citas semilla de esa semana. Cada una muestra hora y paciente; su estado y su médico están en el `aria-label`, y el estado también en la leyenda.
- [ ] CA2. Cuando se elige un médico en el filtro, entonces solo quedan sus citas. Al pasar a la semana siguiente, sigue filtrado. Cuando se vuelve a "Todos los médicos", entonces reaparecen todas.
- [ ] CA3. Dada una cita `programada`, cuando se arrastra a una franja futura libre dentro del horario de su médico, entonces queda en la nueva franja y aparece el `Toast` de éxito. Tras recargar la página sigue ahí, y en `clinica_citas` solo cambiaron `fecha` y `hora`.
- [ ] CA4. Dada una cita del médico A, cuando se arrastra encima de otra cita no cancelada de A, entonces vuelve a su franja original, aparece un `Toast` con el mensaje de solapamiento del servicio y `clinica_citas` no cambia (regla de solapamiento de `CLAUDE.md`).
- [ ] CA5. Cuando se arrastra una cita a un día que no está en `horario.dias` de su médico, a una hora fuera de su horario de atención o a una franja ya pasada, entonces vuelve a su franja original y el `Toast` muestra el mensaje correspondiente del servicio (reglas de horario y de fecha no pasada de `CLAUDE.md`).
- [ ] CA6. Dada una cita `atendida`, `cancelada` o `no_asistio`, cuando se intenta arrastrarla, entonces no se mueve. Ninguna cita se puede estirar ni acortar desde el calendario.
- [ ] CA7. Dada una cita del médico A a las 09:00, cuando se arrastra a las 09:00 de un día en que el médico B ya tiene una cita (y A está libre y dentro de su horario), entonces se guarda sin error.

## Tareas
- [x] T1. modificar `SETUP.md`: en el árbol, dentro de `components/citas/`, listar `CalendarioCitas.vue` y `FiltroMedico.vue`; en "Estado actual", indicar que el spec 007 agrega la agenda de recepción.
- [x] T2. modificar `package.json`: instalar los paquetes de FullCalendar de la sección "Fase 3 (agenda)" de `SETUP.md`, sin agregar otros.
- [x] T3. crear `src/components/citas/FiltroMedico.vue`: `Select` de PrimeVue con "Todos los médicos" y los médicos por `nombreCompleto`. Recibe los médicos por prop y expone el `medicoId` elegido (o `null`) con `v-model` (RF3).
- [x] T4. crear `src/components/citas/CalendarioCitas.vue`: FullCalendar con semana y día, en español y sin fila de "todo el día". Recibe por props las citas ya filtradas, los pacientes y los médicos, y los convierte en eventos (RF2). Solo se arrastran las citas `programada`/`confirmada` y no se permite cambiar la duración. Al soltar una cita, emite `reprogramar` con el id, la nueva `fecha` (`YYYY-MM-DD`), la nueva `hora` (`HH:mm`) y una forma de revertir el movimiento (RF4).
- [x] T5. crear `src/views/recepcion/AgendaView.vue`: vista delgada. Carga los stores de citas, médicos y pacientes; guarda el filtro, pasa al calendario las citas filtradas y, ante `reprogramar`, llama a `useCitasStore.reprogramar`. Muestra el `Toast` de éxito o error, y si falla revierte el movimiento (RF1, RF5, RF6).
- [x] T6. modificar `src/router/index.js`: la hija `agenda` de `/recepcion` usa `AgendaView` en lugar de `EnConstruccionView`.
- [x] T7. Rehacer la presentación según la maqueta (RF visual): `CalendarioCitas.vue`, `FiltroMedico.vue`, `AgendaView.vue` (RF7).

## Fuera de alcance
- Crear citas (hacer clic en una franja, elegir paciente, aviso de alergias) y cancelarlas desde la agenda: van en un spec siguiente (ver Decisiones abiertas).
- Detalle de una cita al hacer clic y cambiar su estado.
- Marcar en la grilla el horario de atención de cada médico (la leyenda de RF7 sí va).
- Botón "Nueva cita", popover de detalle con Reprogramar/Cancelar y vista Mes: van al spec 008.
- `/medico/agenda` (fase 4).
- Tests (fase 8).

## Decisiones abiertas
- Crear y cancelar citas desde la agenda (lo pide `CLAUDE.md` para la Agenda) no está en el requerimiento. Propuesta: un spec 008 que incluya el formulario de nueva cita con el aviso de alergias y la cancelación en dos pasos. Confirmar si va aparte o si se suma a este spec (así superaría el límite de tamaño).
- Las citas `cancelada` se muestran en el calendario, con su estado en texto. Alternativa: ocultarlas. Se eligió mostrarlas porque la agenda dice "todas las citas".
- Reprogramar arrastrando no pide confirmación: soltar la cita es la acción, y si se rompe una regla la cita vuelve a su lugar. Reprogramar no es destructivo, así que no aplica la regla de los dos pasos. Si se quiere un `ConfirmDialog` antes de guardar, agregarlo a T5.
- El spec 006 sigue en `borrador`. Este spec solo puede implementarse después de que el 006 esté aprobado e implementado.
