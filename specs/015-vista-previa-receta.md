# 015 — Vista previa en vivo de la receta

Estado: implementado
Fase: 5
Depende de: 014 (`RecetaView.vue`, `RecetaForm.vue`, `pdf/recetaPdf.js`, `pdf/comunes.js`)

## Contexto
Emitir receta (`/medico/receta/:consultaId`) ya tiene cabecera, franja de alergias, `RecetaForm.vue` y los botones "Descargar" e "Imprimir" con su aviso de bloqueo (spec 014), pero no muestra cómo quedará el PDF. Se agrega la columna "Vista previa" de la maqueta `docs/diseno/pantallas/EmitirReceta.dc.html` (líneas 107-144): el nombre del archivo y una hoja HTML que refleja la receta mientras se edita, con los botones debajo (líneas 146-152). El nombre de archivo ya se calcula en `pdf/recetaPdf.js` (función privada) y los datos del membrete están en `CLINICA` de `pdf/comunes.js`. Como `components/` no puede importar `pdf/` (tabla de capas de `SETUP.md`), la vista le pasa al componente esos dos datos por props. Sin dependencias nuevas.

## Requisitos
- RF1. Junto al formulario, una columna con el título "Vista previa" y a su derecha el nombre del archivo que bajará "Descargar" (`receta_<primer apellido>_<YYYY-MM-DD>.pdf`, fecha de la consulta). Debe salir de la misma función que usa la descarga, no de una copia. Debajo de la hoja van los botones "Descargar" e "Imprimir" y el aviso de bloqueo que ya existen.
- RF2. La hoja muestra el contenido del PDF de receta: membrete ("Clínica Demo", "[Dirección de demo] · Tel. [000 000 000]"), médico (nombre, especialidad, colegiatura), paciente (nombre completo), "DNI · Edad", fecha de la consulta en `DD/MM/YYYY`, el bloque "Rp/" con la tabla medicamento, dosis, frecuencia, duración y vía, las "Indicaciones generales", el recuadro del QR, el id de la receta y la línea "Firma y sello del médico".
- RF3. La hoja se actualiza en vivo con cada cambio del formulario (escribir, agregar o quitar un medicamento, editar las indicaciones generales), sin guardar ni pulsar ningún botón.
- RF4. En cada fila, las indicaciones del medicamento van debajo de su nombre, solo si las tiene. Un campo obligatorio vacío (medicamento, dosis, frecuencia, duración o vía) se muestra como el texto "falta" resaltado, como en la maqueta (línea 130). Sin indicaciones generales, ese bloque no se muestra.
- RF5. Id de la receta: si la consulta ya tiene receta guardada, se muestra su id; si no, "Se asigna al guardar". Tras "Descargar" o "Imprimir" aparece el id real. El recuadro del QR es el marcador "QR" de la maqueta, no un QR real.
- RF6. La hoja es de solo lectura y tiene `aria-label` "Vista previa de la receta en PDF". En tablet, la columna se apila debajo del formulario sin generar scroll horizontal.

## Criterios de aceptación
- [ ] CA1. Dada una consulta de la semilla con receta, cuando se abre Emitir receta, entonces la hoja muestra sus medicamentos, sus indicaciones generales y el id de la receta, y el nombre sobre la hoja coincide con el del archivo que baja "Descargar".
- [ ] CA2. Dada la hoja abierta, entonces el membrete, el médico logueado (nombre, especialidad y colegiatura), el paciente (nombre, DNI, edad) y la fecha de la consulta coinciden con los del PDF generado.
- [ ] CA3. Cuando el médico escribe en dosis o en indicaciones generales, entonces la hoja cambia en el momento, sin pulsar ningún botón. Al agregar o quitar un medicamento, la tabla de la hoja agrega o quita esa fila.
- [ ] CA4. Dado un medicamento sin duración, entonces su celda de duración en la hoja dice "falta" (con texto, no solo con color), el campo muestra su error y los botones siguen bloqueados como en el spec 014. Al completarla, "falta" desaparece.
- [ ] CA5. Dado un medicamento con indicaciones, entonces aparecen debajo de su nombre en la hoja; uno sin indicaciones no muestra esa línea. Si se borran las indicaciones generales, su bloque desaparece de la hoja.
- [ ] CA6. Dada una consulta sin receta, entonces la hoja muestra "Se asigna al guardar" en lugar del id; después de "Descargar" muestra el id guardado.
- [ ] CA7. Dada una pantalla de tablet (menos de 1024 px), entonces la vista previa queda debajo del formulario, con los botones debajo de la hoja, y la página no tiene scroll horizontal.
- [ ] CA8. Regla "Antes de agendar o recetar, mostrar un aviso visible con las alergias" de `CLAUDE.md`: la franja de alergias sigue arriba del formulario y de la vista previa.

## Tareas
- [x] T1. modificar `SETUP.md`: en `components/recetas/`, agregar `VistaPreviaReceta.vue` (hoja de vista previa de la receta); en "Estado actual", el spec 015 agrega la vista previa en vivo de Emitir receta.
- [x] T2. modificar `src/pdf/recetaPdf.js`: exportar la función del nombre de archivo (hoy privada) como `nombreArchivoReceta`, y que la descarga la siga usando.
- [x] T3. crear `src/components/recetas/VistaPreviaReceta.vue`: título, nombre de archivo y hoja de RF2, RF4 y RF5. Recibe por props los medicamentos y las indicaciones generales del formulario, el paciente, el médico, la fecha, el id de la receta (o ninguno), los datos de la clínica y el nombre del archivo. No importa `pdf/`.
- [x] T4. modificar `src/views/medico/RecetaView.vue`: la columna derecha pasa a tener `VistaPreviaReceta` arriba de los botones; le pasa `formulario`, paciente, médico, `consulta.fecha`, el id de `recetaExistente`, `CLINICA` y `nombreArchivoReceta(paciente, consulta.fecha)`; ancho de columna y apilado en tablet según la maqueta (RF6).

## Fuera de alcance
- Pie de receta en la Historia clínica ("Receta", "Ver receta", "Sin receta emitida"; `HistoriaClinica.dc.html`, líneas 116-121 y 138): quedó para el 015 en el spec 014 y pasa a un spec aparte (016).
- QR real en la vista previa (se ve en el PDF) y el render del PDF real dentro de la página (iframe o canvas).
- Paginado, zoom o impresión desde la vista previa.
- PDF de historia clínica, cruce de medicamentos con las alergias y `indicacionesPaciente` de la IA: ver "Fuera de alcance" del spec 014.

## Decisiones abiertas
- **Pie de receta de la Historia clínica fuera de este spec**, aunque el spec 014 lo había asignado al 015. El requerimiento solo pide la vista previa. La alternativa es sumarlo acá (tocaría `TarjetaConsulta.vue` y `HistoriaView.vue`).
- **Hoja HTML, no el PDF real**: se replica el contenido con HTML y el PDF de `recetaPdf.js` sigue siendo la fuente de verdad. Renderizar el PDF en un iframe costaría regenerarlo en cada tecla.
- **Altura de la hoja con muchos medicamentos**: la maqueta usa una hoja de alto fijo (proporción A4). Propuesta: mantener ese alto como mínimo y dejar que crezca si el contenido no entra, sin recortar ni paginar.
- **Id antes de guardar**: se propone "Se asigna al guardar", porque el id se genera al guardar. La alternativa es ocultar la línea del id hasta que exista.
- **`RecetaView.vue` ya pasa las ~200 líneas** (286, más de la mitad son template y estilos). Este spec suma poco. Si querés adelgazarla, el bloque de botones y aviso puede pasar a un componente en este mismo spec, como una T5.
