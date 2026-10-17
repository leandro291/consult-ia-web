# 012 — Atender cita: datos del paciente y registro de consulta

Estado: implementado
Fase: 4
Depende de: 011 (`useConsultasStore`, `registrarConsulta`, `validarConsulta`, `medicoId` en la sesión, "Atender" en Mi agenda), 010 (`ChipEstadoCita.vue`)

## Contexto
Segundo spec de la fase 4. `/medico/atencion/:citaId` hoy muestra `EnConstruccionView.vue`. Este spec arma la pantalla "Atender cita" sin asistente IA: cabecera con el paciente y la cita, franja de alergias, última consulta del paciente y formulario de consulta. Al guardar se usa `useConsultasStore().registrar` (spec 011), que deja la cita `atendida`.
No existe `src/data/cie10.js`; se crea con una lista reducida para elegir diagnósticos. Se reutilizan `ChipEstadoCita.vue`, `ChipAlergia.vue`, `CampoError.vue`, `validarConsulta`, `calcularEdad`, `formatearFecha` y `nombreCompleto`.
Maqueta: `docs/diseno/pantallas/AtenderCita.dc.html` (cabecera, líneas 37-48; franja de alergias, 50-56; última consulta, 93-97; formulario de consulta, 104-157; barra inferior, 211-220). Franja de alergia: `DESIGN.md`, línea 267.

## Requisitos
- RF1. La cabecera muestra el enlace "Mi agenda" (a `/medico/agenda`), el nombre completo del paciente, su edad, sexo, DNI y grupo sanguíneo, y la hora de la cita, el consultorio con su piso y el estado de la cita con `ChipEstadoCita`.
- RF2. Si el paciente tiene alergias, justo debajo de la cabecera y antes de cualquier otro contenido se ve la franja roja de alergias de `DESIGN.md` ("Alérgico a …" con todas sus alergias) y sus antecedentes. Sin alergias, no hay franja y los antecedentes se muestran en la cabecera.
- RF3. El panel "Última consulta" muestra la consulta más reciente del paciente (de cualquier médico): fecha, médico, diagnósticos con código, temperatura y presión. Si no tiene consultas, dice "Sin consultas anteriores".
- RF4. El formulario "Consulta" muestra la fecha de la cita y tiene: motivo (precargado con el motivo de la cita), signos vitales (presión, frecuencia cardiaca, temperatura, peso y talla, con su unidad), examen físico, diagnósticos (CIE-10), plan y observaciones (opcional).
- RF5. Diagnósticos: "Agregar diagnóstico" permite buscar en `data/cie10.js` por código o descripción y agregar el elegido a la lista; cada diagnóstico de la lista muestra código y descripción y se puede quitar (nombre accesible "Quitar diagnóstico <código>"). No se agrega dos veces el mismo código.
- RF6. La barra inferior tiene "Guardar consulta". Al pulsarlo se valida con `validarConsulta` y los errores se muestran junto a cada campo. Si es válido, se registra la consulta; con éxito aparece el Toast "Consulta registrada." y se vuelve a `/medico/agenda`, donde la cita ya figura `atendida`. Si el servicio rechaza, se muestra su mensaje en un Toast de error y el formulario conserva lo escrito.
- RF7. Si la cita no existe o no es del médico logueado, la pantalla muestra "La cita no existe o no pertenece a su agenda." y el enlace a Mi agenda, sin datos del paciente ni formulario.
- RF8. Si la cita no está `programada` ni `confirmada`, se ven cabecera, franja y última consulta, pero en lugar del formulario aparece "Esta cita está <estado>; no se puede registrar una consulta." (regla de `CLAUDE.md`).
- RF9. `data/cie10.js` exporta una lista de `{ codigo, descripcion }` con unos 30 códigos frecuentes en consulta ambulatoria, que incluye todos los códigos usados en `data/seed.js`.

## Criterios de aceptación
- [ ] CA1. Dada una cita `programada` del médico logueado, cuando pulsa "Atender" en Mi agenda, entonces ve la cabecera con nombre, edad, sexo, DNI, grupo sanguíneo, hora, consultorio, piso y estado, y el formulario con el motivo de la cita ya cargado.
- [ ] CA2. Regla de `CLAUDE.md` (aviso de alergias): dado el paciente semilla alérgico a la penicilina, entonces la franja roja con "Penicilina" aparece debajo de la cabecera, antes del formulario, con `role="note"` y `aria-label="Alergias del paciente"`. Un paciente sin alergias no muestra franja.
- [ ] CA3. Dado un paciente con consultas semilla, entonces "Última consulta" muestra la más reciente con fecha `DD/MM/YYYY`, médico, diagnóstico con código, temperatura y presión; uno sin consultas muestra "Sin consultas anteriores".
- [ ] CA4. Dado "Agregar diagnóstico", cuando el médico escribe `J02` o `faringitis`, entonces puede elegir el código y aparece en la lista; al quitarlo desaparece; elegir un código ya agregado no lo duplica.
- [ ] CA5. Dado "Guardar consulta" con el motivo vacío, sin diagnósticos o con presión `12080`, entonces se ve el error junto a cada campo inválido y no se guarda nada.
- [ ] CA6. Dados datos válidos, cuando pulsa "Guardar consulta", entonces aparece "Consulta registrada.", vuelve a Mi agenda, la cita figura `atendida` sin "Atender", y al volver a abrir `/medico/atencion/<id>` se ve el mensaje de RF8 en lugar del formulario.
- [ ] CA7. Regla de `CLAUDE.md` (solo citas `programada` o `confirmada`): dada una cita `cancelada`, `atendida` o `no_asistio` abierta por URL, entonces no hay formulario ni "Guardar consulta" y se ve el mensaje de RF8.
- [ ] CA8. Dado el id de una cita de otro médico o un id inexistente en la URL, entonces se ve "La cita no existe o no pertenece a su agenda." y ningún dato del paciente.

## Tareas
- [x] T1. modificar `SETUP.md`:
  - En `components/comunes/`, agregar `FranjaAlergias` (franja roja de Atender cita y Emitir receta).
  - En `components/consultas/`, agregar `CabeceraAtencion.vue`, `UltimaConsulta.vue`, `ConsultaForm.vue` y `DiagnosticosField.vue`.
  - En "Estado actual", indicar que el spec 012 agrega la pantalla "Atender cita" sin asistente IA.
- [x] T2. crear `src/data/cie10.js`: lista reducida de códigos (RF9).
- [x] T3. crear `src/components/comunes/FranjaAlergias.vue`: franja de alergias con antecedentes; no renderiza nada sin alergias (RF2). Maqueta, líneas 50-56.
- [x] T4. crear `src/components/consultas/CabeceraAtencion.vue`: cabecera de RF1 (y antecedentes cuando no hay alergias). Maqueta, líneas 37-48.
- [x] T5. crear `src/components/consultas/UltimaConsulta.vue`: recibe la consulta más reciente (o ninguna) y el médico, y muestra RF3. Maqueta, líneas 93-97.
- [x] T6. crear `src/components/consultas/DiagnosticosField.vue`: lista editable de diagnósticos con buscador sobre `cie10.js` (`v-model` y mensaje de error) (RF5). Maqueta, líneas 134-146.
- [x] T7. crear `src/components/consultas/ConsultaForm.vue`: campos de RF4 con errores por campo, usa `DiagnosticosField`, valida con `validarConsulta` y emite `guardar` con los datos; incluye la barra inferior con "Guardar consulta" (RF6). Maqueta, líneas 104-157 y 211-220.
- [x] T8. crear `src/views/medico/AtencionView.vue`: carga los stores de citas, pacientes, médicos, consultorios y consultas; resuelve la cita del parámetro y comprueba que sea del `medicoId` de la sesión (RF7); arma cabecera, franja, última consulta y formulario o el mensaje de RF8; al guardar llama a `registrar`, recarga el store de citas, muestra el Toast y navega (RF6).
- [x] T9. modificar `src/router/index.js`: `atencion/:citaId` usa `AtencionView`.

## Fuera de alcance
- El panel "Asistente de consulta", las marcas "Sugerido por IA" y el contador de sugerencias (maqueta, líneas 61-91, 108 y 214): fase 7.
- La línea "No recetar amoxicilina ni ampicilina" de la franja (maqueta, línea 53): depende de `utils/alergias.js`, fase 7.
- "Borrador de receta" y "Generar receta PDF" (maqueta, líneas 160-207 y 217): fase 5.
- Los enlaces "Historia clínica" y "Ver historia clínica completa" (maqueta, líneas 46 y 96): spec posterior de historia clínica.
- La marca "código a verificar" de diagnósticos: solo aplica a códigos que vienen de la IA (fase 7).
- Editar una consulta ya registrada.

## Decisiones abiertas
- **"Historial" = solo "Última consulta"**, como la maqueta. Si se quiere la lista de todas las consultas anteriores en esta pantalla, `UltimaConsulta.vue` pasa a recibir la lista.
- **Tras guardar se vuelve a Mi agenda.** La maqueta habilita "Generar receta PDF" en la misma pantalla; eso se resuelve en la fase 5, que decidirá si la pantalla queda abierta después de guardar.
- **`FranjaAlergias.vue` nuevo en vez de reutilizar `AlertaAlergias.vue`:** `DESIGN.md` define la franja de todo el ancho para Atender cita y Emitir receta, distinta del recuadro que usa `AlertaAlergias`.
