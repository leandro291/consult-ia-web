# 013 — Historia clínica del paciente (línea de tiempo)

Estado: implementado
Fase: 4
Depende de: 011 (`useConsultasStore`, Mi agenda, `EventoCita.vue`), 012 (`CabeceraAtencion.vue`, `UltimaConsulta.vue`)

## Contexto
Se construye `/medico/historia/:pacienteId`: la ficha del paciente y la línea de tiempo de todas sus consultas. La maqueta es `docs/diseno/pantallas/HistoriaClinica.dc.html` (cabecera: líneas 59-72; ficha: 76-93; línea de tiempo: 95-166). Hoy la ruta existe en el router pero apunta a `EnConstruccionView`. `listarConsultas` y `useConsultasStore.lista` ya alcanzan (se filtran por `pacienteId` en la vista, como hace `AtencionView`), así que no se tocan el servicio ni el store. Se agregan además los accesos a la historia desde Mi agenda y Atender cita, que los specs 011 y 012 dejaron pendientes.

## Requisitos
- RF1. Cabecera: enlace "Mi agenda" que vuelve a `/medico/agenda`, título "Historia clínica" y subtítulo con nombre del paciente, cantidad de consultas y mes/año (`MM/YYYY`) de la más antigua. Sin consultas, el subtítulo dice "Sin consultas registradas".
- RF2. Ficha del paciente: nombre, aviso de alergias (solo si tiene), DNI, fecha de nacimiento con edad, sexo, grupo sanguíneo, teléfono y antecedentes.
- RF3. Línea de tiempo con **todas** las consultas del paciente, de cualquier médico. Orden por fecha, con el selector "Más recientes" (por defecto) / "Más antiguas".
- RF4. Consulta expandida: título = descripción del primer diagnóstico (si no hay, el motivo); fecha, nombre y especialidad del médico; línea de signos vitales (presión, frecuencia cardiaca, temperatura, peso, talla; "—" en los vacíos); motivo, examen físico, diagnósticos (código y descripción) y plan. Se omiten las filas vacías.
- RF5. Las dos primeras consultas del orden actual se muestran expandidas; las demás, resumidas (título, fecha, médico y código del primer diagnóstico) con "Ver detalle", que expande y vuelve a colapsar.
- RF6. Estados: cargando, error de carga con "Reintentar", paciente inexistente ("Paciente no encontrado" con enlace a Mi agenda) y paciente sin consultas (estado vacío en el lugar de la línea de tiempo).
- RF7. En Mi agenda, cada cita `atendida` muestra la acción "Ver historia", que abre la historia de su paciente (maqueta `MiAgendaMedico.dc.html`, línea 105).
- RF8. En Atender cita, el botón "Historia clínica" de la cabecera y el enlace "Ver historia clínica completa" del panel "Última consulta" abren la historia del paciente (maqueta `AtenderCita.dc.html`, líneas 46 y 96). Este último solo aparece si hay una consulta previa.

## Criterios de aceptación
- [ ] CA1. Dado un médico logueado, cuando abre la historia de un paciente semilla con consultas, entonces ve la ficha de RF2 y el subtítulo con la cantidad y el mes/año de la primera consulta.
- [ ] CA2. Dado un paciente alérgico a la penicilina, cuando se abre su historia, entonces la ficha muestra el aviso rojo con "Penicilina"; un paciente sin alergias no muestra el aviso.
- [ ] CA3. Dado un paciente con consultas de dos médicos distintos, cuando se abre su historia, entonces aparecen todas, la más reciente primero; al elegir "Más antiguas", el orden se invierte y el botón activo queda con `aria-pressed="true"`.
- [ ] CA4. Dada una consulta sin examen físico ni talla, cuando se muestra expandida, entonces no aparece la fila "Examen" y la talla figura como "—".
- [ ] CA5. Dado un paciente con 3 o más consultas, cuando se abre su historia, entonces a partir de la tercera se ven resumidas; "Ver detalle" la expande y cambia `aria-expanded`; volver a pulsarlo la colapsa.
- [ ] CA6. Dada una URL con un `pacienteId` inexistente, entonces se ve "Paciente no encontrado" con enlace a Mi agenda; dado un paciente sin consultas, se ve el estado vacío.
- [ ] CA7. Dada una cita atendida en Mi agenda, cuando se pulsa "Ver historia" (con `aria-label` que nombra al paciente), entonces se navega a su historia; las citas en otros estados no muestran esa acción.
- [ ] CA8. Dada la pantalla Atender cita, cuando se pulsa "Historia clínica" o "Ver historia clínica completa", entonces se navega a la historia del paciente de la cita.

## Tareas
- [x] T1. modificar `SETUP.md`: agregar `FichaPaciente.vue` en `components/pacientes/` y `TarjetaConsulta.vue` en `components/consultas/`; en `EventoCita.vue`, mencionar "Ver historia"; en "Estado actual", indicar que el spec 013 agrega la historia clínica (sin PDF).
- [x] T2. crear `src/components/pacientes/FichaPaciente.vue`: ficha de RF2; reutiliza `AlertaAlergias.vue`, `nombreCompleto`, `formatearFecha` y `calcularEdad`. Maqueta, líneas 76-93.
- [x] T3. crear `src/components/consultas/TarjetaConsulta.vue`: recibe la consulta, el médico y si arranca expandida; muestra RF4 o el resumen de RF5 con el botón "Ver detalle". Maqueta, líneas 99-115 y 142-152.
- [x] T4. crear `src/views/medico/HistoriaView.vue`: carga los stores de pacientes, médicos y consultas; resuelve el paciente del parámetro; filtra y ordena sus consultas según el selector; arma cabecera (RF1), `FichaPaciente`, la línea de tiempo con `TarjetaConsulta` y los estados de RF6.
- [x] T5. modificar `src/router/index.js`: `historia/:pacienteId` usa `HistoriaView`.
- [x] T6. modificar `src/components/citas/EventoCita.vue`: botón "Ver historia" cuando la cita está atendida; emite `ver-historia` con el id del paciente.
- [x] T7. modificar `src/components/citas/CalendarioCitas.vue`: pasa a `EventoCita` el id del paciente y si la cita está atendida, y reemite `ver-historia` (solo en modo `soloLectura`).
- [x] T8. modificar `src/views/medico/MiAgendaView.vue`: con `ver-historia`, navega a `/medico/historia/:pacienteId`.
- [x] T9. modificar `src/components/consultas/CabeceraAtencion.vue`: enlace "Historia clínica" a la historia del paciente.
- [x] T10. modificar `src/components/consultas/UltimaConsulta.vue`: enlace "Ver historia clínica completa" cuando hay consulta.

## Fuera de alcance
- Botón "Exportar PDF" (maqueta, línea 70) y `pdf/historiaPdf.js`: fase 5. En este spec **no se muestra** el botón.
- Pie de receta de cada consulta ("Receta", medicamentos, "Ver receta" y "Sin receta emitida"; maqueta, líneas 116-121 y 138): fase 5, junto con recetas.
- Marca "Dictada con asistente IA" (maqueta, línea 107): fase 7.
- Acceso a la historia desde recepción y restricción por médico: cualquier médico ve la historia de cualquier paciente, como en la maqueta, que muestra consultas de varios médicos.
- Refactorizar `AtencionView` para compartir el filtrado de consultas por paciente.

## Decisiones abiertas
- **Sin botón "Exportar PDF" en vez de mostrarlo deshabilitado.** Un botón muerto confunde al usuario y la fase 5 lo agrega junto con `historiaPdf.js`. Si preferís mostrarlo deshabilitado desde ya, se suma a T4.
- **Aviso de alergias con `AlertaAlergias.vue` (chips)** en lugar de copiar el recuadro de la maqueta (ícono + texto). `FranjaAlergias.vue` no aplica: es la franja de todo el ancho de Atender cita y Emitir receta. Si se exige el recuadro exacto de la maqueta, habría que adaptar `AlertaAlergias.vue` (lo usan otras pantallas).
- **Dos consultas expandidas por defecto**, como la maqueta. La alternativa es expandir solo la más reciente.
