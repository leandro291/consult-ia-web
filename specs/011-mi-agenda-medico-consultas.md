# 011 — Mi agenda del médico y capa de datos de consultas

Estado: implementado
Fase: 4
Depende de: 006 (`citasService`, `useCitasStore`), 007 (`CalendarioCitas.vue`), 004 (pacientes), 003 (médicos)

## Contexto
Primer spec de la fase 4 (se parte en dos: este y el 012, pantalla "Atender cita"). `/medico/agenda` hoy muestra `EnConstruccionView.vue`; la sesión (`authService.login`) no guarda el `medicoId` y no existen `consultasService.js` ni `stores/consultas.js`.
Este spec agrega "Mi agenda" reutilizando `CalendarioCitas.vue` en modo de solo lectura, y la capa de datos de consultas siguiendo el patrón de `citasService.js` + `stores/citas.js` (`useCatalogo`).
Maqueta: `docs/diseno/pantallas/MiAgendaMedico.dc.html` (cabecera, líneas 59-71; barra del calendario, 79-87; citas del día, 89-155).

## Requisitos
- RF1. Al iniciar sesión un usuario con `medicoId`, la sesión guardada en `clinica_sesion` incluye ese `medicoId`. La sesión de recepción no cambia.
- RF2. `/medico/agenda` muestra el título "Mi agenda", el día de la semana y la fecha de hoy, y un calendario con **solo** las citas del médico logueado (todas sus citas, en cualquier estado).
- RF3. El calendario abre en la vista Día, en la fecha de hoy, y permite cambiar a Semana, ir a Hoy y avanzar o retroceder (misma barra que la agenda de recepción).
- RF4. En la agenda del médico no se puede arrastrar citas, crear citas haciendo clic en un hueco ni filtrar por médico.
- RF5. Cada cita muestra hora, paciente, el ícono de alergia si el paciente tiene alergias y, en la vista Día, el motivo y el estado en texto. Las citas `programada` y `confirmada` muestran la acción "Atender" (nombre accesible "Atender a <paciente>"), que lleva a `/medico/atencion/:citaId`. Las demás no la tienen.
- RF6. `validarConsulta(datos)` en `utils/validaciones.js` devuelve un error por campo: motivo obligatorio; al menos un diagnóstico con descripción; presión, si viene, con formato `sistólica/diastólica` (ej. `120/80`); frecuencia cardiaca, temperatura, peso y talla, si vienen, números mayores que 0. Los signos vitales vacíos son válidos.
- RF7. `citasService` expone `atenderCita(id)`: pasa la cita a `atendida` y rechaza con `Error` en español si no existe o no está `programada` o `confirmada` (misma regla que reprogramar y cancelar).
- RF8. `consultasService` expone `listarConsultas()` y `registrarConsulta(datos)`. `registrarConsulta` valida con `validarConsulta`, llama a `atenderCita(datos.citaId)` y guarda en `clinica_consultas` un registro con el modelo de `CLAUDE.md` (`id`, `creadoEn`, `citaId`, `pacienteId` y `medicoId` tomados de la cita, `fecha` de la cita, `motivo`, `signosVitales`, `examenFisico`, `diagnosticos`, `plan`, `observaciones`). Si algo falla, no guarda nada ni cambia la cita.
- RF9. `useConsultasStore` expone `lista`, `cargando`, `error`, `cargar()` y `registrar(datos)`, con `useCatalogo` como `useCitasStore`.

## Criterios de aceptación
- [ ] CA1. Dado `medico1@clinica.com` logueado, cuando entra a `/medico/agenda`, entonces ve la vista Día de hoy con solo sus citas; en Semana tampoco aparecen citas de otros médicos. `clinica_sesion` tiene su `medicoId`; la de recepción no tiene `medicoId`.
- [ ] CA2. Dada la agenda del médico, entonces no hay filtro de médico, arrastrar una cita no la mueve y hacer clic en un hueco no abre ningún diálogo.
- [ ] CA3. Dada una cita `programada` o `confirmada`, cuando el médico activa "Atender" (clic o teclado), entonces navega a `/medico/atencion/<id de la cita>`. Una cita `atendida`, `cancelada` o `no_asistio` no ofrece "Atender".
- [ ] CA4. Dado el paciente semilla alérgico a la penicilina con una cita del médico, entonces esa cita muestra el ícono de alergia.
- [ ] CA5. Dada la agenda de recepción, entonces sigue igual que antes: filtro de médico, arrastrar para reprogramar, clic en hueco para crear y detalle de cita al hacer clic.
- [ ] CA6. Regla de `CLAUDE.md`: dado `registrarConsulta` sobre una cita `programada` o `confirmada` con datos válidos, entonces se agrega la consulta a `clinica_consultas` y la cita queda `atendida`; sobre una cita en otro estado, lanza "Solo se puede atender una cita programada o confirmada." y no se guarda nada.
- [ ] CA7. Dado `registrarConsulta` sin motivo, sin diagnósticos, con presión `12080` o con temperatura `abc`, entonces lanza el `Error` del campo en español, no guarda la consulta y la cita no cambia de estado.
- [ ] CA8. Dado `useConsultasStore().registrar` con un error de servicio, entonces devuelve `false` y deja el mensaje en `error`; con éxito devuelve `true` y `lista` incluye la nueva consulta.

## Tareas
- [x] T1. modificar `SETUP.md`:
  - En `authService.js`, anotar que la sesión incluye `medicoId` para los médicos.
  - En `components/citas/`, anotar que `CalendarioCitas.vue` tiene un modo de solo lectura para el médico.
  - En `useCatalogo.js`, anotar que también lo usa el store de consultas.
  - En "Estado actual", indicar que el spec 011 inicia la fase 4 con "Mi agenda" y la capa de datos de consultas.
- [x] T2. modificar `src/services/authService.js`: `login` agrega `medicoId` a la sesión cuando el usuario lo tiene (RF1).
- [x] T3. modificar `src/utils/validaciones.js`: agregar `validarConsulta(datos)` (RF6).
- [x] T4. modificar `src/services/citasService.js`: agregar `atenderCita(id)`, reutilizando `buscarActiva` (RF7).
- [x] T5. crear `src/services/consultasService.js`: `listarConsultas` y `registrarConsulta` (RF8).
- [x] T6. crear `src/stores/consultas.js`: `useConsultasStore` (RF9).
- [x] T7. modificar `src/components/citas/CalendarioCitas.vue`: prop de solo lectura que abre en Día, oculta el filtro de médico, desactiva arrastre y clic en hueco, muestra motivo y estado en la vista Día y la acción "Atender", que emite el id de la cita en lugar de abrir `DetalleCita`. Sin la prop, el comportamiento actual no cambia (RF3 a RF5).
- [x] T8. crear `src/views/medico/MiAgendaView.vue`: cabecera (RF2), carga los stores de citas, pacientes, médicos y consultorios, filtra las citas por el `medicoId` de la sesión y navega a la atención al recibir "Atender".
- [x] T9. modificar `src/router/index.js`: `agenda` del médico usa `MiAgendaView`. `atencion/:citaId` sigue con `EnConstruccionView` hasta el spec 012.

## Fuera de alcance
- La pantalla "Atender cita" (`/medico/atencion/:citaId`): spec 012.
- La historia clínica (`/medico/historia/:pacienteId`) y el enlace "Ver historia" de las citas atendidas (maqueta, líneas 105, 112 y 125): spec posterior.
- El indicador "Asistente IA activo" de la cabecera (maqueta, línea 65): fase 7.
- El resumen "N citas · N atendidas · siguiente a las HH:mm" (maqueta, línea 86).
- El texto "Sin citas de HH:mm a HH:mm" en los huecos (maqueta, línea 149).
- Marcar una cita como `no_asistio` o confirmarla desde la agenda del médico.
- Editar o eliminar consultas ya registradas.

## Decisiones abiertas
- **Modelo de `clinica_sesion`:** `CLAUDE.md` lo define como `{ usuarioId, rol, nombre }`. El requerimiento pide que el `medicoId` salga de la sesión, así que RF1 agrega `medicoId?`. El hilo principal debería actualizar esa línea de `CLAUDE.md` (este agente no puede). Un médico con sesión iniciada antes del cambio tiene que volver a entrar para ver su agenda. Alternativa sin tocar el modelo: buscar el `medicoId` en `clinica_usuarios` a partir de `usuarioId`.
- **Reutilizar `CalendarioCitas.vue` con una prop** en vez de crear un calendario nuevo, para no duplicar la configuración de FullCalendar ni sus estilos por estado. Si el componente pasa de ~200 líneas, el developer puede sacar el contenido del evento a un componente aparte dentro de `components/citas/`.
- **`fecha` de la consulta = fecha de la cita**, no la del día en que se registra.
