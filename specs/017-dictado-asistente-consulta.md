# 017 — Dictado y asistente de consulta en Atender cita

Estado: implementado
Fase: 7
Depende de: 012 (Atender cita, `ConsultaForm`, `DiagnosticosField`), 016 (`iaService.js` con `VITE_IA_SERVIDOR_URL`)

## Contexto
Agrega a `/medico/atencion/:citaId` el panel "Asistente de consulta": dictado por voz (Web Speech API, sin dependencias), "Generar con IA" y prellenado a lápiz del formulario de consulta (sección "Asistente de consulta por voz (IA)" de `CLAUDE.md`, pasos 1 a 7 y 9 sin receta). Hoy existen `generarSugerencias` en `iaService.js` (spec 016), `TRANSCRIPCION_DEMO` en `seed.js`, `CIE10` en `data/cie10.js` y el formulario de consulta de `ConsultaForm.vue`; no existen `useDictado.js`, `useAsistenteConsulta.js`, `anonimizar.js` ni `components/asistente/`. Maqueta: `docs/diseno/pantallas/AtenderCita.dc.html` (panel de la columna izquierda, sobre "Última consulta", y la hoja "Consulta").

## Requisitos
- RF1. El panel aparece solo cuando la cita admite consulta (programada o confirmada), con el aviso fijo de IA de `CLAUDE.md`, un indicador "IA activa" / "IA desactivada", el área de texto editable "Transcripción" y los botones "Generar con IA" y "Usar ejemplo".
- RF2. El asistente está activo solo si `VITE_IA_SERVIDOR_URL` tiene valor; `iaService.js` expone esa consulta. Desactivado, "Generar con IA" queda deshabilitado con el texto "El asistente de IA no está configurado en esta instalación." y la consulta se registra igual de forma manual.
- RF3. Dictado (`useDictado.js`, parámetros de `CLAUDE.md`): iniciar, pausar/reanudar, detener/seguir dictando y limpiar. Lo dictado se agrega al final del texto actual sin borrar lo escrito o editado por el médico. Mientras escucha se ve el punto rojo pulsante y un estado en texto ("Dictando", "Dictado en pausa", "Dictado detenido"). Si el navegador no soporta la API, no se muestran los controles de dictado; si se niega el micrófono o la API falla, se muestra un mensaje en español y el área de texto sigue usable.
- RF4. "Usar ejemplo" reemplaza el texto de la transcripción por `TRANSCRIPCION_DEMO`.
- RF5. "Generar con IA" llama a `generarSugerencias(transcripcion, contexto)`, con `contexto` = salida de `anonimizar.js`: `{ edad, sexo, alergias, antecedentes }` del paciente y nada más. Está deshabilitado con la transcripción vacía o mientras espera; durante la espera muestra spinner y "Analizando consulta…". Tras la primera generación correcta el botón pasa a "Volver a generar".
- RF6. Con respuesta válida se cargan en el formulario `motivo`, `signosVitales`, `examenFisico`, `diagnosticos` y `plan`. Los valores `null` o arrays vacíos no pisan lo que ya tiene el formulario. Cada campo cargado se ve a lápiz con la marca "Sugerido por IA" (regla lápiz → tinta de `DESIGN.md`) y la cabecera de la hoja muestra "N campos sugeridos por IA". Al editar un campo sugerido pierde la marca y pasa a tinta.
- RF7. `advertencias` de la respuesta se listan en el panel bajo "Revise antes de guardar · N"; sin advertencias, no se muestra la lista.
- RF8. Un diagnóstico cuyo código no está en `CIE10` o viene sin código lleva la marca "código a verificar"; se puede quitar como cualquier otro.
- RF9. Si `generarSugerencias` lanza error, su mensaje se muestra en el panel y la transcripción y el formulario quedan como estaban, para reintentar.
- RF10. Si hubo al menos una generación correcta, la consulta se guarda además con `generadaConIA: true` y `transcripcion` = el texto enviado en la última generación correcta (modelo de `clinica_consultas` en `CLAUDE.md`). Sin generación, la consulta no lleva esos campos.

## Criterios de aceptación
- [ ] CA1. Dado `VITE_IA_SERVIDOR_URL` vacía, cuando el médico abre una cita programada, entonces ve el panel con "IA desactivada", "Generar con IA" deshabilitado con el texto de RF2, y puede llenar y guardar la consulta a mano.
- [ ] CA2. Dado un navegador sin Web Speech API, entonces no aparecen los controles de dictado y el médico puede escribir o pegar en "Transcripción".
- [ ] CA3. Dado Chrome con micrófono permitido, cuando inicia el dictado y habla, entonces aparece el punto rojo y "Dictando", y el texto se agrega tras lo ya escrito; pausar, reanudar, detener y limpiar se comportan según RF3; con el permiso negado aparece el mensaje de error y el texto no se pierde.
- [ ] CA4. Dado el asistente activo, cuando el médico pulsa "Usar ejemplo" y luego "Generar con IA", entonces ve "Analizando consulta…" con el botón deshabilitado, y el cuerpo del POST lleva solo `transcripcion` y `contexto` con edad, sexo, alergias y antecedentes (sin nombre, DNI, teléfono, email ni dirección).
- [ ] CA5. Dada una respuesta con motivo, presión, frecuencia, temperatura, examen, un diagnóstico y plan, y `peso`/`talla` en `null`, entonces esos campos quedan cargados a lápiz con "Sugerido por IA", peso y talla siguen vacíos, la cabecera dice "7 campos sugeridos por IA" y las `advertencias` aparecen en el panel con su conteo.
- [ ] CA6. Dado un campo sugerido, cuando el médico lo edita, entonces pierde la marca y el contador baja; dado un diagnóstico sugerido con código `J02.0` (no está en `CIE10`), entonces se ve "código a verificar".
- [ ] CA7. Dado el servidor apagado, un 500 o una respuesta mal formada, cuando pulsa "Generar con IA", entonces ve el mensaje de `iaService.js` en el panel y la transcripción y el formulario no cambian.
- [ ] CA8. Dada una consulta guardada tras generar con IA, entonces el registro en `clinica_consultas` tiene `generadaConIA: true` y la `transcripcion` enviada; una consulta guardada sin generar no tiene esos campos. Se mantiene la regla de `CLAUDE.md`: solo se registra consulta en citas programadas o confirmadas.

## Tareas
- [x] T1. modificar `SETUP.md`: tabla de capas (`components/` puede importar `data`, como ya hace `DiagnosticosField`; `utils/` puede importar otros `utils`); árbol: comentario de `components/asistente/` con `PanelAsistente`, `ControlesDictado`, `MarcaSugerido`; "Estado actual" con el spec 017.
- [x] T2. crear `src/utils/anonimizar.js`: `anonimizar(paciente)` → `{ edad, sexo, alergias, antecedentes }`, reutilizando `calcularEdad` de `formato.js`.
- [x] T3. modificar `src/services/iaService.js`: exportar `asistenteActivo()` (misma lectura y `trim` de la URL que `generarSugerencias`).
- [x] T4. modificar `src/services/consultasService.js`: `registrarConsulta` guarda `generadaConIA` y `transcripcion` solo si `datos.generadaConIA` es verdadero (RF10).
- [x] T5. crear `src/composables/useDictado.js`: soporte, estado (inactivo, dictando, en pausa, detenido), error y acciones de RF3 sobre un texto recibido por referencia.
- [x] T6. crear `src/composables/useAsistenteConsulta.js`: transcripción, activo, cargando, error, advertencias, sugerencia (ya con la forma del formulario: signos como texto, sin claves `null` ni arrays vacíos), transcripción enviada y `generar(paciente)`.
- [x] T7. crear `src/components/asistente/MarcaSugerido.vue`: marca "Sugerido por IA" de la maqueta.
- [x] T8. crear `src/components/asistente/ControlesDictado.vue`: usa `useDictado`; botones, punto rojo, estado en texto y error de RF3.
- [x] T9. crear `src/components/asistente/PanelAsistente.vue`: cabecera con indicador, aviso, `ControlesDictado`, "Transcripción" (v-model), "Generar con IA"/"Volver a generar", "Usar ejemplo", carga, error y advertencias (RF1, RF2, RF4, RF5, RF7, RF9).
- [x] T10. modificar `src/components/consultas/ConsultaForm.vue`: prop `sugerencia`; al cambiar, aplica sus campos, los marca a lápiz con `MarcaSugerido` y cuenta los sugeridos en la cabecera; editar un campo le quita la marca (RF6).
- [x] T11. modificar `src/components/consultas/DiagnosticosField.vue`: marca "código a verificar" (RF8) y quitar por posición, no por código, para admitir diagnósticos sin código.
- [x] T12. modificar `src/views/medico/AtencionView.vue`: usa `useAsistenteConsulta`, monta `PanelAsistente` sobre `UltimaConsulta` cuando la cita es editable, pasa la sugerencia a `ConsultaForm` y agrega `generadaConIA` y `transcripcion` al guardar (RF10).

## Fuera de alcance
- Borrador de receta desde la IA, `alergias.js` y su alerta, e `indicacionesPaciente` en el PDF: spec 018. De la maqueta quedan fuera por eso la hoja "Receta", la línea de alergia del panel de advertencias, el bloqueo de "Guardar consulta" y el contador "sugerencias por revisar" de la barra inferior.
- El cronómetro del estado de dictado ("01:15" de la maqueta).
- Agregar al panel de advertencias una línea local por cada código CIE-10 a verificar: la marca en el diagnóstico ya lo avisa.
- Enlace a `/medico/configuracion` desde el botón deshabilitado: la ruta no existe desde el spec 016.

## Decisiones abiertas
- **"Volver a generar"** sobrescribe sin confirmar los campos que la nueva respuesta trae con valor, aunque el médico ya los haya editado. Alternativa: pedir confirmación con `ConfirmDialog` si hay campos editados.
- **`transcripcion` guardada**: se propone la enviada en la última generación correcta, no la que haya en el área de texto al guardar (que el médico pudo seguir editando). Alternativa: guardar la del área de texto.
- **Diagnósticos sugeridos**: reemplazan la lista del formulario (no se suman a los ya cargados a mano). Alternativa: agregarlos sin duplicar códigos.
