# 004 — Catálogos: pacientes

Estado: implementado
Fase: 2
Depende de: 003

## Contexto
Recepción lista, busca, crea, edita, elimina y ve el detalle de los pacientes (modelo `clinica_pacientes` de `CLAUDE.md`). Hoy `/recepcion/pacientes` y `/recepcion/pacientes/:id` muestran `EnConstruccionView`. Se reutilizan `storage.js`, `validaciones.js`, `formato.js` (`formatearFecha`), `CampoError.vue` y el patrón de servicio y store del spec 003.

Decisiones:
- `pacientesService` lee `CLAVES.consultas` vía `storage.js`, solo lectura, para dos cosas: bloquear la eliminación y agregar a cada paciente listado `totalConsultas` y `ultimaConsulta`. Estos campos derivados no se guardan. No se crea `consultasService` (fase 4).
- La unicidad del DNI se valida en el servicio (al editar, se excluye el propio registro). Como es el único error de negocio del formulario, el mensaje del servicio se muestra bajo el campo DNI, como en `PacienteNuevo.dc.html`.
- La búsqueda filtra la lista ya cargada. Puede usar el filtro global de `DataTable`.
- Se crea `comunes/AlertaAlergias.vue` (ya listado en `SETUP.md`) para el detalle. Las fases 3 y 5 lo reutilizan.
- `calcularEdad` va en `utils/formato.js`; la usa también el PDF de receta (fase 5).

## Requisitos
- RF1. Listado (`docs/diseno/pantallas/Pacientes.dc.html`):
  - tabla paginada de a 8 filas, ordenada por apellido;
  - columnas: DNI, "Apellidos, Nombres", edad, teléfono, última consulta (`DD/MM/YYYY` o "Sin consultas") y acciones Ver, Editar y Eliminar;
  - un chip de alergia (con ícono de triángulo) junto al nombre por cada alergia;
  - el subtítulo indica el número de pacientes registrados.
- RF2. Búsqueda por DNI, nombres o apellidos, sin distinguir mayúsculas ni tildes. Sin resultados, se muestra el estado vacío de `EstadosVaciosCarga.dc.html` ("Ningún paciente coincide con “…”").
- RF3. Crear y editar en un `Dialog` (`PacienteNuevo.dc.html`) con todos los campos del modelo:
  - alergias como lista: Enter agrega el texto (sin vacíos ni duplicados) y cada alergia tiene su botón "Quitar <alergia>";
  - antecedentes como texto libre;
  - título "Nuevo paciente" o "Editar paciente".
- RF4. Validación:
  - DNI de 8 dígitos y único;
  - nombres, apellidos, fecha de nacimiento (no futura) y sexo obligatorios;
  - correo opcional, pero válido si viene;
  - teléfono, dirección, grupo sanguíneo, alergias y antecedentes opcionales.
  - Los errores se muestran junto a cada campo y el foco va al primero con error.
- RF5. Eliminación en dos pasos:
  - Con consultas: aviso "No se puede eliminar a <nombre>", con la cantidad de consultas y el motivo, y un solo botón "Entendido" (`EstadosVaciosCarga.dc.html`).
  - Sin consultas: `ConfirmDialog` "¿Eliminar a <nombre>?" con "Cancelar" y "Sí, eliminar" (rojo lleno).
  - El servicio también rechaza con `Error` la eliminación de un paciente con consultas.
- RF6. Detalle (`/recepcion/pacientes/:id`), solo lectura:
  - nombre completo, DNI, fecha de nacimiento y edad, sexo, teléfono, correo, dirección y grupo sanguíneo;
  - `AlertaAlergias` si tiene alergias (`role="note"`, `aria-label="Alergias del paciente"`);
  - antecedentes;
  - enlace "Volver a pacientes".
  - Con un id inexistente: "Paciente no encontrado." y el mismo enlace.
- RF7. Los registros nuevos llevan `id` (`crypto.randomUUID()`) y `creadoEn` (ISO); editar conserva ambos. Toast al guardar o eliminar. Las rutas `pacientes` y `pacientes/:id` usan las vistas nuevas.

## Criterios de aceptación
- [ ] CA1. Dada la semilla, cuando se abre `/recepcion/pacientes`, entonces:
  - hay 10 pacientes ordenados por apellido, 8 por página;
  - la edad y la última consulta son correctas;
  - la paciente alérgica muestra el chip "Penicilina" junto a su nombre.
- [ ] CA2. Búsqueda:
  - "4012" deja solo los pacientes cuyo DNI lo contiene;
  - "ramirez" encuentra a "Ramírez";
  - "zzz" muestra "Ningún paciente coincide con “zzz”" y "Revise el apellido o busque por DNI.".
- [ ] CA3. Dado un paciente nuevo válido con dos alergias, cuando se guarda, entonces aparece en la tabla y sigue ahí al recargar, con `id` UUID, `creadoEn`, `alergias` como array y sin `totalConsultas` ni `ultimaConsulta` en `clinica_pacientes`.
- [ ] CA4. DNI único y de 8 dígitos:
  - "1234567" o "1234567a" → "El DNI debe tener 8 dígitos.";
  - el DNI de otro paciente → bajo el campo, "Ya existe un paciente con este DNI (<nombres apellidos>)." y no se guarda;
  - editar un paciente sin cambiar su DNI guarda bien.
- [ ] CA5. Dado el formulario vacío, cuando se guarda, entonces los campos obligatorios muestran su error y el foco va al DNI. Una fecha futura → "La fecha de nacimiento no puede ser futura.". El correo "abc" → "Ingrese un correo válido."; vacío es válido.
- [ ] CA6. No se elimina un paciente con consultas registradas: al pulsar "Eliminar" en un paciente semilla con consultas, aparece el aviso con la cantidad y el motivo, y el paciente sigue en la lista y en `clinica_pacientes`.
- [ ] CA7. Dado un paciente sin consultas, cuando se pulsa "Eliminar", entonces aparece la confirmación. "Cancelar" no borra nada; "Sí, eliminar" lo quita de la tabla y de `clinica_pacientes` y muestra un Toast.
- [ ] CA8. Detalle:
  - "Ver" lleva a `/recepcion/pacientes/:id` con los datos de RF6;
  - la paciente alérgica muestra el aviso rojo con "Penicilina"; un paciente sin alergias no muestra aviso rojo;
  - un id inexistente muestra "Paciente no encontrado.";
  - las acciones de cada fila tienen `aria-label` con el nombre ("Ver a …", "Editar a …", "Eliminar a …").

## Tareas
- [x] T1. modificar `SETUP.md`: comentario de `utils/formato.js` → "`formatearFecha`, `calcularEdad`". Nombrar `PacientesView` y `PacienteDetalleView` en `views/recepcion/`. Estado actual: "fase 2 implementada (specs 003 y 004)".
- [x] T2. modificar `src/utils/validaciones.js`: `validarPaciente` (RF4, sin la unicidad).
- [x] T3. modificar `src/utils/formato.js`: `calcularEdad(fechaNacimiento)` en años cumplidos.
- [x] T4. crear `src/services/pacientesService.js`:
  - `listarPacientes` (con los campos derivados), `obtenerPaciente`, `crearPaciente`, `actualizarPaciente` y `eliminarPaciente`;
  - DNI único y bloqueo por consultas (RF4, RF5, RF7).
- [x] T5. crear `src/stores/pacientes.js`: `usePacientesStore` con la lista, `cargando`, `error` y acciones que recargan la lista después de cada cambio.
- [x] T6. crear `src/components/comunes/AlertaAlergias.vue`: prop `alergias`; no renderiza nada si está vacía (RF6). Sigue el aviso rojo de `Componentes.dc.html`, sin el texto de familias (fase 7).
- [x] T7. crear `src/components/pacientes/PacienteForm.vue`: `Dialog` de alta y edición; valida con `validarPaciente` y guarda vía store (RF3, RF4).
- [x] T8. crear `src/views/recepcion/PacientesView.vue`: listado, búsqueda, estado vacío y eliminación en dos pasos (RF1, RF2, RF5).
- [x] T9. crear `src/views/recepcion/PacienteDetalleView.vue` (RF6).
- [x] T10. modificar `src/router/index.js`: `pacientes` y `pacientes/:id` apuntan a sus vistas (RF7).

## Fuera de alcance
- Editar o eliminar desde el detalle.
- Historial de consultas en el detalle (fase 4).
- Familias de medicamentos en el aviso de alergias (fase 7).
- Diseño del layout lateral (`RecepcionLayout.dc.html`).
- Estado de carga (lectura síncrona) y locale `es` de Day.js.

## Decisiones abiertas
- **Campos obligatorios:** la maqueta solo marca "Correo (opcional)". Se propone además marcar como opcionales teléfono, dirección y grupo sanguíneo. ¿El teléfono debe ser obligatorio?
- **Detalle sin maqueta** ("Pendiente de diseño" en `docs/diseno/README.md`): se propone la composición de RF6, en solo lectura.
- **Botón Eliminar de la tabla:** la maqueta usa un ícono gris de 40 px, mientras que `DESIGN.md` pide un disparador con borde rojo. Se sigue la maqueta (botones de ícono en tablas) y el rojo lleno solo en la confirmación.
