# 014 — Emitir receta y PDF de receta

Estado: implementado
Fase: 5
Depende de: 012 (`AtencionView.vue`, `FranjaAlergias.vue`), 011 (`consultasService.js`, `useConsultasStore`)

## Contexto
Se construye `/medico/receta/:consultaId` ("Emitir receta"): formulario de medicamentos ligado a una consulta, guardado en `clinica_recetas` y exportado a PDF con pdfmake. Maqueta: `docs/diseno/pantallas/EmitirReceta.dc.html` (cabecera: líneas 35-45; franja de alergias: 47-51; formulario: 55-105; botones y aviso de bloqueo: 146-152). Hoy la ruta apunta a `EnConstruccionView`, no existen `recetasService.js`, `stores/recetas.js` ni `pdf/`, y `AtencionView` vuelve a Mi agenda al guardar. Se instala `pdfmake` (ya prevista en `SETUP.md` para la fase 5); el QR se hace con el nodo `qr` nativo de pdfmake, así que **no** se instala `qrcode`. La semilla ya trae una receta por cada consulta atendida.

## Requisitos
- RF1. En Atender cita, al guardar la consulta la pantalla no navega a Mi agenda: se queda y muestra que la cita está atendida. Toda cita atendida que tenga consulta muestra el botón "Generar receta PDF", que abre `/medico/receta/:consultaId`.
- RF2. Emitir receta solo abre consultas del médico logueado. Consulta inexistente o ajena: "La consulta no existe o no pertenece a su agenda." con enlace a Mi agenda. Estados de carga y de error de carga con "Reintentar", como en `AtencionView`.
- RF3. Cabecera: enlace "Atender cita" (vuelve a la atención de esa cita), título "Emitir receta", subtítulo con nombre del paciente, "Consulta del DD/MM/YYYY" y código y descripción del primer diagnóstico, y el sello "Consulta guardada" con la fecha y hora de registro de la consulta.
- RF4. Aviso de alergias: la franja roja de todo el ancho (`FranjaAlergias.vue`) sobre el formulario, solo si el paciente tiene alergias.
- RF5. Formulario "Medicamentos": una fila por medicamento con medicamento, dosis, frecuencia, duración, vía (lista fija: oral, sublingual, tópica, inhalatoria, intramuscular, endovenosa, rectal, oftálmica, ótica) e indicaciones; botón "Quitar" por fila, "Agregar medicamento" y el campo "Indicaciones generales". Si la consulta ya tiene receta, el formulario se carga con ella; si no, arranca con una fila vacía.
- RF6. Validación: al menos un medicamento; en cada uno son obligatorios medicamento, dosis, frecuencia, duración y vía (indicaciones es opcional). Los errores se muestran junto a cada campo. Mientras haya errores, "Descargar" e "Imprimir" quedan deshabilitados y debajo se indica el primer dato faltante.
- RF7. "Descargar" e "Imprimir" primero guardan la receta (una sola por consulta: crea la primera vez, actualiza las siguientes; paciente, médico y fecha salen de la consulta) y después generan el PDF. "Descargar" baja `receta_<primer apellido>_<YYYY-MM-DD>.pdf` (apellido en minúsculas y sin tildes, fecha de la receta); "Imprimir" abre el PDF en una pestaña nueva. Si el guardado falla, se muestra un Toast de error y no se genera el PDF.
- RF8. El PDF tiene el contenido de "PDFs (pdfmake) → Receta médica" de `CLAUDE.md`, con el QR incluido. Membrete: "Clínica Demo", "[Dirección de demo]" y "Tel. [000 000 000]", como en la maqueta (líneas 114-120). Membrete, estilos y pie viven en `pdf/comunes.js` para que los reuse el PDF de historia clínica.

## Criterios de aceptación
- [ ] CA1. Dada una cita programada en Atender cita, cuando el médico guarda la consulta, entonces sigue en la pantalla, la cita figura como atendida y "Generar receta PDF" lo lleva a la receta de esa consulta. Una cita atendida de la semilla también muestra el botón.
- [ ] CA2. Dada la consulta de la paciente semilla alérgica a la penicilina, cuando se abre Emitir receta, entonces la franja roja de alergias se ve antes del formulario (regla de negocio "Antes de agendar o recetar…" de `CLAUDE.md`). Un paciente sin alergias no la muestra.
- [ ] CA3. Dada una consulta de la semilla con receta, cuando se abre Emitir receta, entonces el formulario trae sus medicamentos y sus indicaciones generales; dada una consulta nueva sin receta, arranca con una fila vacía.
- [ ] CA4. Dado un medicamento sin duración, entonces el campo muestra su error, "Descargar" e "Imprimir" están deshabilitados y el aviso de abajo nombra el dato faltante. Al quitar todas las filas, el aviso pide agregar al menos un medicamento. Cada "Quitar" tiene un `aria-label` que nombra su medicamento.
- [ ] CA5. Dado un formulario válido, cuando se pulsa "Descargar", entonces se baja `receta_<apellido>_<fecha>.pdf` con membrete, médico (nombre, especialidad, colegiatura), paciente (nombre, DNI, edad, fecha), tabla de medicamentos, indicaciones generales, línea de firma, pie con el id de la receta y QR. Al recargar la página, el formulario conserva lo guardado.
- [ ] CA6. Dado un formulario válido, cuando se pulsa "Imprimir", entonces el PDF se abre en una pestaña nueva. Pulsar "Descargar" y después "Imprimir" deja una sola receta para esa consulta en `clinica_recetas`.
- [ ] CA7. Dada una URL con un `consultaId` inexistente o de otro médico, entonces se ve el mensaje de RF2 con enlace a Mi agenda. Regla "Una receta siempre pertenece a una consulta existente": el servicio lanza un `Error` en español si la consulta no existe y no guarda nada.

## Tareas
- [x] T1. modificar `SETUP.md`: en la tabla de capas, `views/` puede importar `pdf/`; quitar `qrcode` (opcional) de la instalación y de las dependencias, y anotar en `pdfmake` que el QR es nativo; en `components/recetas/`, `RecetaForm.vue`; en `useCatalogo.js`, mencionar recetas; en `views/medico/`, `RecetaView`; en "Estado actual", fase 5 en curso con el spec 014.
- [x] T2. modificar `package.json`: `npm install pdfmake`.
- [x] T3. modificar `src/utils/validaciones.js`: `validarReceta` con las reglas de RF6, que devuelve los errores por medicamento y por campo.
- [x] T4. crear `src/services/recetasService.js`: `listarRecetas` y `guardarReceta` (valida con `validarReceta`, exige consulta existente y crea o actualiza la receta de esa consulta), siguiendo el patrón de `consultasService.js`.
- [x] T5. crear `src/stores/recetas.js`: `useRecetasStore` con `useCatalogo`, `cargar` y `guardar`, como `stores/consultas.js`.
- [x] T6. crear `src/pdf/comunes.js`: membrete de la clínica, estilos y pie compartidos.
- [x] T7. crear `src/pdf/recetaPdf.js`: arma el documento de RF8 con `comunes.js`, y expone descargar (con el nombre de RF7) e imprimir (pestaña nueva). Recibe receta, paciente y médico ya resueltos.
- [x] T8. crear `src/components/recetas/RecetaForm.vue`: filas de medicamentos, "Agregar medicamento", "Quitar", indicaciones generales y errores por campo (RF5 y RF6); reutiliza `CampoTexto.vue`.
- [x] T9. crear `src/views/medico/RecetaView.vue`: carga los stores de consultas, pacientes, médicos y recetas; resuelve la consulta del médico logueado; arma la cabecera (RF3), `FranjaAlergias`, `RecetaForm`, los botones con el aviso de bloqueo y los estados de RF2; guarda y llama a `recetaPdf.js` (RF7).
- [x] T10. modificar `src/router/index.js`: `receta/:consultaId` usa `RecetaView`.
- [x] T11. modificar `src/views/medico/AtencionView.vue`: al guardar, se queda en la pantalla y recarga la cita; si la cita está atendida y tiene consulta, muestra "Generar receta PDF" hacia su receta (RF1).

## Fuera de alcance
- **Vista previa** en vivo de la receta y el nombre de archivo sobre ella (maqueta, líneas 107-144): spec 015.
- Pie de receta en la Historia clínica ("Receta", "Ver receta", "Sin receta emitida"; `HistoriaClinica.dc.html`, líneas 116-121 y 138): spec 015.
- PDF de historia clínica (`pdf/historiaPdf.js`) y botón "Exportar PDF" de `HistoriaView`: spec posterior que reusa `pdf/comunes.js`.
- Cruce de cada medicamento con las alergias ("Sin cruce con las alergias registradas", maqueta línea 77) y `utils/alergias.js`: fase 7.
- `indicacionesPaciente` de la IA en el PDF y el borrador de receta dentro de Atender cita (maqueta de Atender cita, líneas 160-217): fase 7.
- Eliminar recetas o emitir varias recetas por consulta.

## Decisiones abiertas
- **Una receta por consulta** (crear o actualizar), como la semilla. Evita duplicados al pulsar "Descargar" e "Imprimir". La alternativa es permitir varias recetas por consulta, cada una con su id.
- **Sin botón "Guardar receta"**: la maqueta no lo tiene, así que guardan "Descargar" e "Imprimir". Si preferís un guardado explícito aparte, se suma a T9.
- **Atender cita ya no vuelve a Mi agenda al guardar** (cambio de comportamiento del spec 012), para habilitar "Generar receta PDF" como dice `CLAUDE.md`. La alternativa es navegar directo a Emitir receta al guardar.
- **Fecha de la receta = fecha de la consulta** (igual que la semilla), no el día en que se emite.
- **Tamaño**: 11 tareas (el límite es ~10). Ya se separó la vista previa al spec 015. Si hace falta recortar más, `pdf/comunes.js`, `recetaPdf.js` y los botones del PDF (T2, T6, T7 y parte de T9) pueden pasar al spec 015, y este spec guardaría con un botón "Guardar receta" temporal.
