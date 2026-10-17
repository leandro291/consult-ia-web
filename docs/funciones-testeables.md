# Funciones candidatas a pruebas unitarias

Catálogo de referencia de las funciones puras / de lógica de negocio del proyecto que son
candidatas razonables para pruebas unitarias (Vitest, fase 8). No incluye
componentes `.vue`, stores de Pinia, handlers de eventos ni lógica atada al ciclo de vida de
Vue o al DOM. Las funciones marcadas **(interna)** no están exportadas por su archivo; para
testearlas directamente haría falta exportarlas (o cubrirlas a través de la función pública que
las usa).

## Services

Capa de persistencia (`storage.js`) y reglas de negocio por dominio.

### storage.js (acceso a localStorage)

| Función | Archivo | Qué hace |
|---|---|---|
| `leer(clave, porDefecto)` | `src/services/storage.js` | Lee y parsea el JSON de `localStorage`; devuelve `porDefecto` si no existe, es `null` o está corrupto |
| `guardar(clave, datos)` | `src/services/storage.js` | Serializa `datos` con `JSON.stringify` y los guarda en `localStorage` |

### Autenticación

| Función | Archivo | Qué hace |
|---|---|---|
| `login(email, password)` | `src/services/authService.js` | Busca el usuario por correo y contraseña, guarda la sesión y la devuelve |
| `logout()` | `src/services/authService.js` | Borra la sesión guardada |
| `sesionActual()` | `src/services/authService.js` | Devuelve una copia de la sesión guardada, o `null` |

### Citas

| Función | Archivo | Qué hace |
|---|---|---|
| `listarCitas()` | `src/services/citasService.js` | Devuelve todas las citas guardadas |
| `crearCita(datos)` | `src/services/citasService.js` | Valida datos, paciente y médico existentes, horario y solapamiento, y crea la cita en estado `programada` |
| `reprogramarCita(id, { fecha, hora })` | `src/services/citasService.js` | Cambia fecha y hora de una cita activa validando horario y solapamiento (ignorándose a sí misma) |
| `cancelarCita(id)` | `src/services/citasService.js` | Pasa una cita activa a estado `cancelada` |
| `atenderCita(id)` | `src/services/citasService.js` | Pasa una cita activa a estado `atendida` |

### Consultas

| Función | Archivo | Qué hace |
|---|---|---|
| `listarConsultas()` | `src/services/consultasService.js` | Devuelve todas las consultas guardadas |
| `registrarConsulta(datos)` | `src/services/consultasService.js` | Valida la consulta, marca la cita como atendida y guarda la consulta (con `generadaConIA`/`transcripcion` si aplica) |

### Consultorios

| Función | Archivo | Qué hace |
|---|---|---|
| `listarConsultorios()` | `src/services/consultoriosService.js` | Devuelve todos los consultorios guardados |
| `crearConsultorio(datos)` | `src/services/consultoriosService.js` | Valida y crea un consultorio |
| `actualizarConsultorio(id, datos)` | `src/services/consultoriosService.js` | Valida y actualiza un consultorio existente |

### Médicos

| Función | Archivo | Qué hace |
|---|---|---|
| `listarMedicos()` | `src/services/medicosService.js` | Devuelve todos los médicos guardados |
| `crearMedico(datos)` | `src/services/medicosService.js` | Valida, evita correos duplicados y crea el médico junto con su usuario de acceso |
| `actualizarMedico(id, datos)` | `src/services/medicosService.js` | Valida y actualiza un médico existente |

### Pacientes

| Función | Archivo | Qué hace |
|---|---|---|
| `listarPacientes()` | `src/services/pacientesService.js` | Devuelve los pacientes ordenados por apellido, con `totalConsultas` y `ultimaConsulta` derivados |
| `obtenerPaciente(id)` | `src/services/pacientesService.js` | Devuelve un paciente por id o lanza error si no existe |
| `crearPaciente(datos)` | `src/services/pacientesService.js` | Valida (incluida unicidad de DNI) y crea un paciente |
| `actualizarPaciente(id, datos)` | `src/services/pacientesService.js` | Valida (incluida unicidad de DNI) y actualiza un paciente existente |
| `eliminarPaciente(id)` | `src/services/pacientesService.js` | Elimina un paciente si no tiene consultas registradas; si tiene, lanza error explicando el motivo |

### Recetas

| Función | Archivo | Qué hace |
|---|---|---|
| `listarRecetas()` | `src/services/recetasService.js` | Devuelve todas las recetas guardadas |
| `guardarReceta(datos)` | `src/services/recetasService.js` | Valida, verifica que la consulta exista y crea o actualiza la receta de esa consulta (una por consulta) |

### Semilla

| Función | Archivo | Qué hace |
|---|---|---|
| `cargarSemilla()` | `src/services/semillaService.js` | Genera los datos de demo y los guarda pisando cada colección |
| `inicializarDatos()` | `src/services/semillaService.js` | Carga la semilla solo si `clinica_version` no existe |

### Servicio de IA

| Función | Archivo | Qué hace |
|---|---|---|
| `asistenteActivo()` | `src/services/iaService.js` | Indica si `VITE_IA_SERVIDOR_URL` tiene valor |
| `generarSugerencias(transcripcion, contexto)` | `src/services/iaService.js` | Hace el POST al servidor de IA (timeout 30 s) y valida/devuelve el JSON de respuesta |
| `extraerJson(texto)` **(interna)** | `src/services/iaService.js` | Quita los bloques ```json``` y extrae el primer objeto JSON del texto de respuesta |
| `respuestaValida(json)` **(interna)** | `src/services/iaService.js` | Verifica que el JSON tenga las claves y tipos mínimos del contrato de la IA |
| `urlServidor()` **(interna)** | `src/services/iaService.js` | Lee y limpia `VITE_IA_SERVIDOR_URL` |
| `mensajeErrorHttp(status)` **(interna)** | `src/services/iaService.js` | Traduce un código HTTP de error a un mensaje en español |

## Utils

Funciones puras sin Vue ni persistencia.

### validaciones.js

| Función | Archivo | Qué hace |
|---|---|---|
| `primerError(errores)` | `src/utils/validaciones.js` | Devuelve el primer mensaje de un objeto de errores, o `null` |
| `validarLogin({ email, password })` | `src/utils/validaciones.js` | Valida correo y contraseña del login |
| `validarConsultorio({ nombre })` | `src/utils/validaciones.js` | Valida que el nombre del consultorio no esté vacío |
| `validarMedico(datos, { alta })` | `src/utils/validaciones.js` | Valida datos del médico y, en alta, correo y contraseña del usuario |
| `validarPaciente(datos)` | `src/utils/validaciones.js` | Valida DNI (8 dígitos), datos obligatorios, fecha de nacimiento no futura y correo |
| `validarCita(datos)` | `src/utils/validaciones.js` | Valida campos obligatorios de una cita y que no empiece en el pasado |
| `validarConsulta(datos)` | `src/utils/validaciones.js` | Valida motivo, al menos un diagnóstico y el formato de los signos vitales |
| `validarReceta(datos)` | `src/utils/validaciones.js` | Valida que haya al menos un medicamento y que cada uno tenga sus campos obligatorios |
| `primerErrorReceta(errores)` | `src/utils/validaciones.js` | Extrae el primer mensaje de error de `validarReceta` (string o arreglo por medicamento) |
| `citasSeSolapan(a, b)` | `src/utils/validaciones.js` | Indica si dos citas de la misma fecha se solapan en el tiempo |
| `citaDentroDelHorario(cita, horario)` | `src/utils/validaciones.js` | Indica si una cita cae en un día y dentro del rango horario de atención del médico |

### formato.js

| Función | Archivo | Qué hace |
|---|---|---|
| `formatearFecha(fecha)` | `src/utils/formato.js` | Formatea una fecha como `DD/MM/YYYY` |
| `calcularEdad(fechaNacimiento)` | `src/utils/formato.js` | Calcula la edad en años cumplidos |
| `formatearFechaHora(fecha)` | `src/utils/formato.js` | Formatea fecha y hora como `DD/MM/YYYY · HH:mm` |
| `normalizarTexto(texto)` | `src/utils/formato.js` | Pasa el texto a minúsculas y le quita tildes, para comparar |
| `nombreCompleto({ nombres, apellidos })` | `src/utils/formato.js` | Concatena nombres y apellidos |
| `listarDias(dias)` | `src/utils/formato.js` | Convierte números de día en una lista de nombres en español (ej. "Lunes, miércoles y viernes") |
| `nombreDia(fecha)` | `src/utils/formato.js` | Devuelve el nombre del día de la semana con mayúscula inicial |
| `formatearRango(inicio, fin)` | `src/utils/formato.js` | Formatea un rango de fechas de forma compacta según coincidan día, mes o año |
| `formatearMesAnio(fecha)` | `src/utils/formato.js` | Formatea una fecha como `MM/YYYY` |
| `formatearHorarioCita({ fecha, hora, duracionMin })` | `src/utils/formato.js` | Arma el texto "Sáb 26/09 · 10:30–11:00" de una cita |

### estadisticas.js

| Función | Archivo | Qué hace |
|---|---|---|
| `citasPorDiaYEstado(citas, lunes)` | `src/utils/estadisticas.js` | Cuenta las citas de lunes a sábado por estado (`programadas` suma programada y confirmada); ignora fechas fuera de esos 6 días |
| `citasPorMedico(citas, medicos)` | `src/utils/estadisticas.js` | Cuenta las citas no canceladas por médico (incluye 0), con etiqueta "Dr. <apellido> · <especialidad>" y de mayor a menor |

### alergias.js y anonimizar.js

| Función | Archivo | Qué hace |
|---|---|---|
| `buscarConflictos(items, alergias)` | `src/utils/alergias.js` | Cruza medicamentos con las alergias del paciente (directas o por familia) y devuelve los conflictos |
| `anonimizar(paciente)` | `src/utils/anonimizar.js` | Arma el contexto anónimo (edad, sexo, alergias, antecedentes) que se envía a la IA |

## Composables (lógica pura)

Solo la lógica interna sin estado reactivo de UI ni ciclo de vida de componentes. El resto de
`src/composables/` (`useCatalogo.js`, `useDictado.js`, `useEliminarPaciente.js`,
`useAlertasAlergias.js`) es manejo de estado reactivo o de APIs del navegador/PrimeVue sin
funciones puras separables, y no se lista aquí (ver nota final).

| Función | Archivo | Qué hace |
|---|---|---|
| `limpiarSugerencia(json)` **(interna)** | `src/composables/useAsistenteConsulta.js` | Da forma a la respuesta de la IA como la espera `ConsultaForm` (signos vitales como texto, sin claves vacías) |
| `limpiarBorradorReceta(json)` **(interna)** | `src/composables/useAsistenteConsulta.js` | Arma el borrador de receta a partir de `receta`/`indicacionesPaciente` de la respuesta de la IA, o `null` si no hay nada que sugerir |

## Data

| Función | Archivo | Qué hace |
|---|---|---|
| `generarDatosSemilla(hoy)` | `src/data/seed.js` | Genera consultorios, médicos, usuarios, pacientes, citas, consultas y recetas de demo a partir de una fecha de referencia |

`src/data/cie10.js` solo exporta la lista `CIE10` (sin funciones), así que no aparece en esta tabla.

## PDF

| Función | Archivo | Qué hace |
|---|---|---|
| `membreteClinica()` | `src/pdf/comunes.js` | Arma el bloque de membrete (nombre, dirección y teléfono de la clínica) para pdfmake |
| `lineaDivisoria()` | `src/pdf/comunes.js` | Arma una línea horizontal de separación para pdfmake |
| `piePagina(identificador, etiqueta)` | `src/pdf/comunes.js` | Devuelve la función de pie de página con el identificador del documento y la numeración |
| `nombreArchivoReceta({ apellidos }, fecha)` | `src/pdf/recetaPdf.js` | Arma el nombre `receta_<apellido>_<YYYY-MM-DD>.pdf` |
| `documentoReceta({ receta, paciente, medico })` | `src/pdf/recetaPdf.js` | Arma la estructura de datos (definición de documento) que pdfmake usa para generar el PDF de la receta |

## Router (lógica pura del guard)

| Función | Archivo | Qué hace |
|---|---|---|
| `resolverAcceso(destino, sesion)` | `src/router/acceso.js` | Decide si se permite el acceso a una ruta o a qué ruta redirigir, según la sesión y el rol |

---

**Qué se excluyó y por qué:** no se listan componentes `.vue`, stores de Pinia ni handlers de
eventos (`@click`, `@submit`, etc.), por no ser el foco de pruebas unitarias de lógica pura.
Tampoco se listan `useCatalogo.js`, `useDictado.js`, `useEliminarPaciente.js` ni
`useAlertasAlergias.js` completos: son manejo de estado reactivo (`ref`/`reactive`/`computed`),
de la Web Speech API o de `ConfirmDialog`/`Toast` de PrimeVue, sin funciones puras separables de
esa capa (`useAlertasAlergias.js` solo envuelve `buscarConflictos`, ya listada en `alergias.js`).
`src/services/iaService.js#generarSugerencias` y `src/pdf/recetaPdf.js` (`descargarRecetaPdf`,
`imprimirRecetaPdf`) dependen de `fetch`/pdfmake; se listan igual porque `fetch` se simula en los
tests, pero las funciones que abren o descargan el PDF quedan fuera por ser efectos secundarios
sin lógica que verificar.
