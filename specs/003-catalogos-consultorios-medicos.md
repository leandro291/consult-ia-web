# 003 — Catálogos: consultorios y médicos

Estado: implementado
Fase: 2
Depende de: 002

## Contexto
Recepción lista, crea y edita consultorios y médicos (modelos `clinica_consultorios` y `clinica_medicos` de `CLAUDE.md`). Hoy `/recepcion/consultorios` y `/recepcion/medicos` muestran `EnConstruccionView`. Ya existen `storage.js` (`CLAVES`, `leer`, `guardar`), el patrón de store de `stores/auth.js`, `utils/validaciones.js` (`validarLogin`) y `comunes/CampoError.vue`.
Por tamaño, la fase 2 se divide en este spec y el 004 (pacientes).

Decisiones:
- **No se eliminan médicos ni consultorios** (el requerimiento solo pide listar, crear y editar).
- `medicosService` comprueba que el consultorio exista leyendo `CLAVES.consultorios` vía `storage.js`, solo lectura.
- Los servicios validan con las mismas funciones de `validaciones.js` que usa el formulario y, si hay errores, lanzan el primero como `Error`.
- Las vistas cruzan datos con los dos stores (nombre del consultorio en médicos, médico asignado en consultorios).
- El formulario de consultorio (3 campos) vive en `ConsultoriosView`. El de médico es un componente propio porque suma el horario.

## Requisitos
- RF1. Consultorios: tabla con nombre, piso, descripción y médico asignado (o "Sin médico asignado"). Crear y editar en un `Dialog`. Nombre obligatorio; piso y descripción opcionales.
- RF2. Médicos: tabla con nombre y colegiatura, especialidad, "consultorio · Piso n", horario (días de lunes a sábado y rango `HH:mm–HH:mm`) y teléfono. Crear y editar en un `Dialog` con nombres, apellidos, especialidad, colegiatura, teléfono, consultorio (selección entre los existentes) y horario (días 1 a 6, hora de inicio y de fin).
- RF3. Validación de médico: nombres, apellidos, especialidad, colegiatura y consultorio obligatorios; al menos un día; inicio y fin obligatorios, con inicio anterior a fin; el consultorio debe existir. Teléfono opcional.
- RF4. Los registros nuevos llevan `id` (`crypto.randomUUID()`) y `creadoEn` (ISO); editar conserva ambos. Los textos se guardan sin espacios al borde y `horario.dias` en orden ascendente.
- RF5. Los errores de validación se muestran junto a cada campo y el foco va al primer campo con error. Los errores del servicio se muestran dentro del diálogo, que queda abierto. Al guardar bien, el diálogo se cierra y aparece un Toast ("Consultorio guardado." / "Médico guardado.").
- RF6. Composición y textos:
  - Listados: `docs/diseno/pantallas/Consultorios.dc.html` y `Medicos.dc.html` (cabecera con título, subtítulo con conteo y botón primario "Nuevo …"; hoja con la tabla; botón "Editar" por fila).
  - Diálogos: el patrón de `PacienteNuevo.dc.html`, con los títulos "Nuevo consultorio", "Editar consultorio", "Nuevo médico" y "Editar médico".
  - Sin consultorios: el estado vacío "Todavía no hay consultorios" de `EstadosVaciosCarga.dc.html`.
- RF7. `/recepcion/consultorios` y `/recepcion/medicos` usan las vistas nuevas; las demás rutas no cambian.
- RF8. Al crear un médico se crea también su usuario de acceso (`clinica_usuarios`: `rol: 'medico'`, `medicoId`, `nombre` con nombres y apellidos). El formulario de alta pide correo y contraseña (texto plano, limitación documentada en `CLAUDE.md`); en edición no se muestran. Correo con el mismo criterio que `validarLogin`, único entre usuarios; contraseña obligatoria de al menos 6 caracteres. Si algo falla no se guarda ni el médico ni el usuario.

## Criterios de aceptación
- [ ] CA1. Dada la semilla, cuando recepción abre `/recepcion/consultorios`, entonces ve los 3 consultorios con su piso, su descripción y su médico asignado, y el subtítulo indica cuántos consultorios hay y en cuántos pisos.
- [ ] CA2. Dado "Nuevo consultorio" con el nombre vacío, cuando se guarda, entonces aparece "El nombre es obligatorio." junto al campo y no se guarda nada. Con un nombre válido, la fila aparece y sigue ahí al recargar, con `id` UUID y `creadoEn`.
- [ ] CA3. Dado "Editar" en un consultorio, cuando se cambia el nombre y se guarda, entonces la fila se actualiza, `id` y `creadoEn` no cambian, y en `/recepcion/medicos` el médico de ese consultorio muestra el nombre nuevo.
- [ ] CA4. Dada la semilla, cuando se abre `/recepcion/medicos`, entonces se ven los 3 médicos con colegiatura, especialidad, consultorio con piso y horario. Los días activos se distinguen de los inactivos por algo más que el color, y el grupo de días tiene un `aria-label` con sus nombres (ej. "Lunes, miércoles, viernes y sábado").
- [ ] CA5. Dado "Nuevo médico" vacío, cuando se guarda, entonces cada campo obligatorio muestra su error, sin días aparece "Seleccione al menos un día." y el foco va al primer error. Con inicio 14:00 y fin 08:00 aparece "La hora de fin debe ser posterior a la de inicio.". En ningún caso se guarda.
- [ ] CA6. Consultorio referenciado debe existir: dado un `consultorioId` que no está en `clinica_consultorios`, cuando se crea o actualiza un médico, entonces el servicio lanza `Error('El consultorio seleccionado no existe.')`, no se guarda nada y el diálogo muestra el mensaje.
- [ ] CA7. Dado "Editar" en un médico, cuando se cambian el consultorio, los días y el horario y se guarda, entonces la tabla lo refleja y sigue igual al recargar, con `id` y `creadoEn` intactos.
- [ ] CA8. En ambas pantallas no hay acción de eliminar. Cada "Editar" tiene un `aria-label` con el nombre de la fila ("Editar Consultorio 101"). Cerrar el diálogo con "Cancelar" o Escape no guarda cambios.
- [ ] CA9. Dado "Nuevo médico" con datos válidos, correo `nuevo@clinica.com` y contraseña `123456`, cuando se guarda, entonces existe el médico y un usuario con `rol: 'medico'` y su `medicoId`, y ese correo inicia sesión y entra a `/medico/agenda`. Con un correo ya usado aparece "El correo ya está registrado." y no se crea ni el médico ni el usuario.

## Tareas
- [x] T1. modificar `SETUP.md`: en el árbol, agregar `components/medicos/` (`MedicoForm.vue`) y nombrar `ConsultoriosView` y `MedicosView` en `views/recepcion/`. Estado actual: "fase 2 en curso (spec 003: consultorios y médicos)".
- [x] T2. modificar `src/utils/validaciones.js`: `validarConsultorio` y `validarMedico` (RF1, RF3, RF8: correo y contraseña solo en alta), con el mismo formato que `validarLogin`.
- [x] T3. crear `src/services/consultoriosService.js`: `listarConsultorios`, `crearConsultorio` y `actualizarConsultorio` (RF1, RF4).
- [x] T4. crear `src/services/medicosService.js`: `listarMedicos`, `crearMedico` y `actualizarMedico`, con validación y existencia del consultorio; `crearMedico` crea también el usuario vía `storage.js` (RF3, RF4, RF8, CA6, CA9).
- [x] T5. crear `src/stores/consultorios.js`: `useConsultoriosStore` con la lista, `cargando`, `error`, `cargar`, `crear` y `actualizar`.
- [x] T6. crear `src/stores/medicos.js`: `useMedicosStore`, con la misma forma que T5.
- [x] T7. crear `src/components/medicos/MedicoForm.vue`: `Dialog` de alta y edición con el horario; valida con `validarMedico`; en alta suma correo y contraseña; guarda vía store (RF2, RF3, RF5, RF8).
- [x] T8. crear `src/views/recepcion/ConsultoriosView.vue`: listado, estado vacío y diálogo de consultorio (RF1, RF5, RF6).
- [x] T9. crear `src/views/recepcion/MedicosView.vue`: listado de médicos con consultorio y horario, y apertura de `MedicoForm` (RF2, RF6).
- [x] T10. modificar `src/router/index.js`: `consultorios` y `medicos` apuntan a sus vistas (RF7).

## Fuera de alcance
- Eliminar médicos o consultorios.
- Cambiar correo o contraseña de un médico ya creado.
- Columna "Citas hoy" de `Consultorios.dc.html`: depende de citas (fase 3).
- Diseño del menú lateral y la cabecera del layout (`RecepcionLayout.dc.html`).
- Búsqueda en médicos y consultorios; unicidad de colegiatura o de nombre de consultorio.
- Estado de carga: la lectura de `localStorage` es síncrona.
- Locale `es` de Day.js: solo se usan formatos numéricos.

## Decisiones tomadas (aprobadas)
- **Usuario del médico nuevo:** se crea en este spec (RF8).
- **Diálogos sin maqueta:** patrón de `PacienteNuevo.dc.html`; horario con 6 casillas Lu–Sá y dos campos de hora.
- **"Dr./Dra.":** se muestra "nombres apellidos" sin prefijo.
- **Estado vacío de médicos:** "Todavía no hay médicos" y "Cree un consultorio y luego registre al médico con su horario.".
- **Piso:** opcional.
