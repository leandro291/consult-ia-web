# 018 — Borrador de receta con IA y alerta de alergias

Estado: implementado
Fase: 7
Depende de: 014 (Emitir receta), 015 (vista previa), 017 (asistente en Atender cita)

## Contexto
Cierra la fase 7: la `receta` y las `indicacionesPaciente` que devuelve la IA (contrato de `CLAUDE.md`) llegan como "Borrador de receta" en Atender cita, y `alergias.js` cruza cada medicamento con las alergias del paciente (pasos 7 a 9 de "Asistente de consulta por voz (IA)" de `CLAUDE.md`). Hoy existen `useAsistenteConsulta.js` (solo sugiere la consulta), `RecetaForm.vue` y `RecetaView.vue` (arranca con la receta guardada o una fila vacía), `useRecetasStore`, `MarcaSugerido.vue` y `normalizarTexto` en `utils/formato.js`; no existe `utils/alergias.js`. El PDF y la vista previa ya imprimen `indicacionesGenerales`. Maqueta: `docs/diseno/pantallas/AtenderCita.dc.html` (hoja "Borrador de receta" y barra inferior).

## Requisitos
- RF1. Si la última generación correcta trae `receta` no vacía o `indicacionesPaciente`, bajo la hoja "Consulta" de Atender cita aparece la hoja "Borrador de receta" con los medicamentos sugeridos como filas editables (agregar y quitar, como en Emitir receta) y el campo "Indicaciones para el paciente" con la ayuda "Se imprime en la receta, en lenguaje sencillo.". Los valores `null` quedan vacíos; una vía que no está en `VIAS_RECETA` queda sin elegir. "Volver a generar" reemplaza el borrador. Sin receta ni indicaciones en la respuesta, la hoja no aparece.
- RF2. En el borrador, cada medicamento sugerido y las indicaciones se ven a lápiz con "Sugerido por IA" hasta que el médico los edita (regla lápiz → tinta de `DESIGN.md`). La marca no se persiste ni aparece en Emitir receta.
- RF3. `alergias.js` (lógica local, sin IA) define las familias de `CLAUDE.md` (penicilina, AINEs, sulfas) y detecta conflicto cuando, sin tildes ni mayúsculas, el nombre del medicamento contiene el nombre de una alergia del paciente o el de un integrante de la familia que se llama como esa alergia.
- RF4. Cada medicamento en conflicto, en el borrador de Atender cita y en el formulario de Emitir receta (sugerido o cargado a mano), muestra la fila resaltada y el bloque rojo "Posible reacción alérgica" con el texto de la maqueta, la casilla "Revisé la alergia y mantengo la indicación", "Mantener" (habilitado solo con la casilla marcada) y "Quitar medicamento". La cabecera de la hoja muestra el chip "Alergia: <alergia>". "Mantener" resuelve la alerta; editar otros campos no la reabre, cambiar el nombre del medicamento sí.
- RF5. Con alertas sin resolver: en Atender cita, "Guardar consulta" queda deshabilitado con el aviso "N alerta(s) de alergia sin resolver"; en Emitir receta, "Descargar" e "Imprimir" quedan deshabilitados con el mismo aviso.
- RF6. Al guardar la consulta con borrador, este (medicamentos e indicaciones) queda en memoria asociado a la cita. Al abrir Emitir receta de esa consulta sin receta guardada, el formulario arranca con el borrador en vez de la fila vacía. La receta se guarda recién con "Descargar" o "Imprimir" (spec 014); si ya hay receta guardada, manda ella. Las alertas resueltas en Atender cita se vuelven a pedir en Emitir receta, antes de recetar.
- RF7. "Indicaciones para el paciente" se guarda como `indicacionesGenerales` de la receta: sale en la vista previa y en el PDF por el camino existente, sin campo nuevo en `clinica_recetas`.

## Criterios de aceptación
- [ ] CA1. Dada Ana María Quispe (alérgica a la penicilina, `seed.js`) y una respuesta de la IA con amoxicilina, paracetamol e `indicacionesPaciente`, cuando se genera con IA, entonces aparece "Borrador de receta" con las dos filas y las indicaciones a lápiz con "Sugerido por IA", el chip "Alergia: Penicilina" y la fila de amoxicilina resaltada con "Posible reacción alérgica".
- [ ] CA2. Dada esa alerta sin resolver, entonces "Guardar consulta" está deshabilitado con "1 alerta de alergia sin resolver" y "Mantener" está deshabilitado hasta marcar la casilla; tras "Mantener" o "Quitar medicamento", "Guardar consulta" se habilita.
- [ ] CA3. Dada la amoxicilina mantenida, cuando el médico edita su dosis, entonces la alerta sigue resuelta y la fila pierde la marca "Sugerido por IA"; cuando cambia su nombre, entonces la alerta vuelve a exigir confirmación.
- [ ] CA4. Dado un paciente sin alergias, o una respuesta sin `receta` ni `indicacionesPaciente`, entonces no aparece ninguna alerta (ni la hoja, en el segundo caso) y la consulta se guarda como en el spec 017.
- [ ] CA5. Dada la consulta guardada con borrador, cuando el médico pulsa "Generar receta PDF", entonces Emitir receta muestra los medicamentos y las indicaciones del borrador como filas editables, sin marcas de IA, la vista previa los refleja y `clinica_recetas` no tiene receta de esa consulta hasta "Descargar" o "Imprimir".
- [ ] CA6. Dada María Isabel Torres (alérgica a sulfas), cuando en Emitir receta el médico agrega a mano "Sulfametoxazol + trimetoprima", entonces, además de la franja de alergias ("Antes de recetar, mostrar un aviso visible con las alergias" de `CLAUDE.md`), aparece la alerta y "Descargar" e "Imprimir" quedan deshabilitados con el aviso hasta mantener o quitar el medicamento.
- [ ] CA7. Dado un borrador con indicaciones para el paciente, cuando el médico descarga la receta, entonces el PDF y la vista previa muestran ese texto en las indicaciones generales, y el registro guardado no lleva marcas de IA.
- [ ] CA8. Dada una receta ya guardada con un medicamento en conflicto, cuando se vuelve a abrir Emitir receta, entonces la alerta aparece de nuevo sin resolver.

## Tareas
- [x] T1. modificar `SETUP.md`: árbol, `composables/useAlertasAlergias.js` y `AlertaAlergiaMedicamento.vue` en el comentario de `components/recetas/`; "Estado actual" con el spec 018.
- [x] T2. crear `src/utils/alergias.js`: familias de RF3 y `buscarConflictos(items, alergias)` → por medicamento en conflicto, su índice, nombre y la alergia; reutiliza `normalizarTexto`.
- [x] T3. crear `src/composables/useAlertasAlergias.js`: recibe items y alergias reactivos; expone conflictos, pendientes (sin resolver), `mantener` y el aviso de RF5. La confirmación se liga al nombre del medicamento (RF4).
- [x] T4. modificar `src/composables/useAsistenteConsulta.js`: expone `borradorReceta` (`{ items, indicacionesGenerales }` o `null`) armado desde `receta` e `indicacionesPaciente` con la forma de fila de `RecetaForm` (RF1); el composable no importa componentes, así que arma la fila él mismo.
- [x] T5. modificar `src/stores/recetas.js`: borrador en memoria por cita, `fijarBorrador(citaId, borrador)` y `tomarBorrador(citaId)` (devuelve y limpia) (RF6).
- [x] T6. crear `src/components/recetas/AlertaAlergiaMedicamento.vue`: bloque rojo de RF4 con casilla, "Mantener" y "Quitar medicamento"; emite `mantener` y `quitar`.
- [x] T7. modificar `src/components/recetas/RecetaForm.vue`: título y etiqueta/ayuda de indicaciones por prop (defaults actuales), chip de alergia, fila resaltada y `AlertaAlergiaMedicamento` bajo cada medicamento en conflicto pendiente, y marcas "Sugerido por IA" opcionales (RF2).
- [x] T8. modificar `src/components/consultas/ConsultaForm.vue`: prop con el aviso de bloqueo; si viene, deshabilita "Guardar consulta" y lo muestra ligado por `aria-describedby` (RF5).
- [x] T9. modificar `src/views/medico/AtencionView.vue`: monta `RecetaForm` como "Borrador de receta" bajo `ConsultaForm` cuando hay borrador, usa `useAlertasAlergias`, pasa el bloqueo y, al guardar con éxito, llama a `fijarBorrador(cita.id, …)`.
- [x] T10. modificar `src/views/medico/RecetaView.vue`: inicializa desde `tomarBorrador(consulta.citaId)` si no hay receta guardada, usa `useAlertasAlergias` y suma su aviso al bloqueo de "Descargar" e "Imprimir".

## Fuera de alcance
- La línea de alergia en el panel "Revise antes de guardar" y el contador "N sugerencias por revisar" de la barra inferior de la maqueta: la alerta de la fila y el contador de la hoja "Consulta" ya lo avisan.
- La maqueta dibuja el borrador como tabla con botón "Editar" por fila; se reutiliza el formulario de filas de `RecetaForm` (ver Decisiones abiertas).
- Persistir el borrador: recargar la página antes de emitir la receta lo pierde y Emitir receta arranca con la fila vacía.
- Cambios en `recetaPdf.js`, `VistaPreviaReceta.vue` o en el modelo de `clinica_recetas` (RF7).
- Tabla farmacológica real: solo las familias de `CLAUDE.md` (limitación ya documentada).

## Decisiones abiertas
- **Diseño del borrador:** reutilizar `RecetaForm` (filas con campos siempre visibles) en vez de la tabla con "Editar" por fila de la maqueta. Evita un segundo formulario de medicamentos; alternativa: componente nuevo fiel a la maqueta (+1 archivo, más líneas).
- **`indicacionesPaciente` = `indicacionesGenerales`:** el borrador tiene un solo campo de indicaciones, así que se reutiliza el existente. Alternativa: campo nuevo `indicacionesPaciente` en `clinica_recetas`, con sección propia en el PDF y la vista previa.
- **Paso del borrador entre vistas:** en memoria en `useRecetasStore`, por `citaId` (Emitir receta conoce `consulta.citaId`). Alternativas: `history.state` del router o guardar la receta al guardar la consulta (no sirve: `guardarReceta` valida dosis, frecuencia, duración y vía, que la IA puede traer en `null`).
- **Confirmación de alergia repetida:** se vuelve a pedir en Emitir receta aunque se haya resuelto en Atender cita. Alternativa: pasar las confirmaciones dentro del borrador.
- **Hoja visible solo con sugerencia de la IA:** sin IA la receta se sigue armando en Emitir receta. Alternativa: mostrar siempre el borrador vacío en Atender cita.
