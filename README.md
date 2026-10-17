# Consult-IA

Aplicación web para gestionar una clínica con varios consultorios: **citas**, **historias clínicas** y **recetas médicas** en PDF, con un asistente de consulta por voz asistido por IA.

> **Proyecto académico.** Prioriza código claro y bien organizado por encima de optimizaciones complejas. No usar con datos médicos reales (ver [Limitaciones](#limitaciones-conocidas)).

## Índice

- [Descripción](#descripción)
- [Stack tecnológico](#stack-tecnológico)
- [Instalación y comandos](#instalación-y-comandos)
- [Usuarios de demo](#usuarios-de-demo)
- [Roles, vistas y rutas](#roles-vistas-y-rutas)
- [Arquitectura en capas](#arquitectura-en-capas)
- [Árbol de carpetas](#árbol-de-carpetas)
- [Modelo de datos](#modelo-de-datos)
- [Diseño e interfaz](#diseño-e-interfaz)
- [PDFs](#pdfs)
- [Asistente de consulta por voz (IA)](#asistente-de-consulta-por-voz-ia)
- [Estado actual](#estado-actual)
- [Tests](#tests)
- [Limitaciones conocidas](#limitaciones-conocidas)

## Descripción

Es una sola aplicación con login. Según el rol del usuario logueado (`recepcion` o `medico`) se muestra un layout y un menú distintos:

- **Recepción**: dashboard del día, pacientes, médicos, consultorios y agenda (calendario de citas).
- **Médico**: su propia agenda, atención de citas (con historia clínica del paciente y asistente de voz con IA), historia clínica del paciente y emisión de recetas en PDF.

No hay backend ni base de datos: toda la persistencia vive en `localStorage` del navegador.

## Stack tecnológico

| Paquete | Propósito |
|---|---|
| `vue` | Framework, Composition API + `<script setup>` |
| `vue-router` | Rutas y guard de sesión/rol |
| `pinia` | Estado global, un store por dominio |
| `primevue` | Componentes UI (DataTable, Dialog, Toast, ConfirmDialog, …) |
| `@primeuix/themes` | Preset de tema de PrimeVue (base Aura), tematizado para la clínica |
| `@fullcalendar/vue3`, `@fullcalendar/core` | Calendario de citas |
| `@fullcalendar/daygrid`, `@fullcalendar/timegrid` | Vistas de mes, semana y día |
| `@fullcalendar/interaction` | Arrastrar para reprogramar y clic para crear |
| `dayjs` | Manejo y formato de fechas (locale `es`) |
| `pdfmake` | PDF de receta médica (el QR usa su nodo `qr` nativo) |
| `vite`, `@vitejs/plugin-vue` | Build y servidor de desarrollo |
| `eslint`, `@eslint/js`, `eslint-plugin-vue`, `globals` | Lint de JS y Vue |
| `vitest`, `jsdom` | Tests unitarios y simulación de navegador |

Sin dependencia adicional, se usan APIs nativas: Web Speech API (dictado), `fetch` (servidor de IA), `crypto.randomUUID()` (ids) y `localStorage` (persistencia).

## Instalación y comandos

```bash
npm install
npm run dev       # servidor de desarrollo
npm run build     # build de producción en dist/
npm run preview   # sirve el build localmente
npm run lint      # lint de JS y Vue
npm run test      # tests con Vitest (modo watch; agregar -- --run para una sola pasada)
```

### Variable de entorno: asistente de IA

El asistente de consulta por voz depende del servidor de IA configurado en `VITE_IA_SERVIDOR_URL`:

```bash
cp .env.example .env
# completar VITE_IA_SERVIDOR_URL con la URL del servidor de IA
```

Es una variable de **build** de Vite (`VITE_*`): hay que reiniciar `npm run dev` o rehacer `npm run build` para que tome el cambio. Si está vacía o ausente, el botón "Generar con IA" queda deshabilitado y la atención médica funciona igual, de forma manual. Al quedar visible en el JS del build, no debe llevar secretos.

## Usuarios de demo

Cargados por `src/data/seed.js` la primera vez que se abre la app (mientras no exista `clinica_version` en `localStorage`):

| Usuario | Contraseña | Rol |
|---|---|---|
| `recepcion@clinica.com` | `123456` | Recepción |
| `medico1@clinica.com` | `123456` | Médico (Medicina General) |
| `medico2@clinica.com` | `123456` | Médico (Pediatría) |
| `medico3@clinica.com` | `123456` | Médico (Cardiología) |

También trae ~10 pacientes (una de las pacientes es alérgica a la penicilina, para probar la alerta de alergias), citas de la semana en distintos estados, y algunas consultas y recetas ya registradas.

## Roles, vistas y rutas

**Recepción** (`meta: { rol: 'recepcion' }`):

```
/recepcion/dashboard      citas del día, conteo por estado
/recepcion/pacientes      listar, buscar por DNI o nombre, crear, editar
/recepcion/pacientes/:id  detalle de paciente
/recepcion/medicos        listar, crear, editar
/recepcion/consultorios   listar, crear, editar
/recepcion/agenda         calendario de todas las citas, filtrable por médico
/recepcion/respaldo       exportar/importar JSON (en construcción)
```

**Médico** (`meta: { rol: 'medico' }`):

```
/medico/agenda                solo las citas del médico logueado
/medico/atencion/:citaId      atender cita: paciente, historial y asistente de voz
/medico/historia/:pacienteId  línea de tiempo de consultas del paciente
/medico/receta/:consultaId    emitir receta en PDF
```

`/login` es pública. `/` redirige a `/login`. Un guard global (`src/router/acceso.js`, aplicado en `src/router/index.js`) redirige a `/login` sin sesión, y a la ruta de inicio del propio rol si la sesión no corresponde a la ruta pedida.

## Arquitectura en capas

```
views/ (por ruta, delgadas)
   │  usa
components/  ──┐        composables/ (useXxx)
   │  usa       │  usa       │  usa
   └─────────► stores/ (Pinia, uno por dominio) ◄────┘
                  │  llama
               services/ (CRUD + reglas de negocio, lanzan Error en español)
                  │  único acceso
               services/storage.js
                  │
               localStorage
```

- **Vistas**: orquestan componentes y llaman a stores o composables; no tienen reglas de negocio.
- **Stores de Pinia**: uno por dominio, llaman a su propio servicio y exponen estado, `cargando` y `error`.
- **Servicios**: única capa con persistencia (a través de `storage.js`); son funciones puras de dominio (sin Vue ni Pinia) y lanzan `Error` con mensajes en español cuando se rompe una regla de negocio.
- **`storage.js`**: único punto de contacto con `localStorage` (`leer`/`guardar`).

¿Por qué así? Si en el futuro hubiera un backend HTTP, solo se reemplazaría `services/`; el resto de la app no cambia (principio de sustitución / inversión de dependencias). Cada capa tiene un único motivo de cambio (SRP), y lo compartido (validaciones, formato de fechas, PDF) vive en un solo lugar para no duplicarlo (DRY).

## Árbol de carpetas

```
src/
├── main.js               # Crea la app, registra Router, Pinia, PrimeVue; corre la semilla
├── App.vue                # Raíz: <RouterView>, Toast y ConfirmDialog globales
│
├── router/
│   ├── index.js           # Rutas con meta.rol + guard global
│   └── acceso.js           # resolverAcceso(): decisión pura del guard
│
├── services/               # Única capa con persistencia
│   ├── storage.js
│   ├── semillaService.js
│   ├── pacientesService.js
│   ├── medicosService.js
│   ├── consultoriosService.js
│   ├── citasService.js     # incluye validación de solapamiento y horario
│   ├── consultasService.js
│   ├── recetasService.js
│   ├── authService.js
│   └── iaService.js        # POST al servidor de IA y validación de la respuesta
│
├── stores/                 # Pinia, uno por dominio + auth
│   ├── auth.js, pacientes.js, medicos.js, consultorios.js
│   └── citas.js, consultas.js, recetas.js
│
├── composables/
│   ├── useCatalogo.js       # estado/ejecución comunes de los stores de catálogo, citas, consultas y recetas
│   ├── useEliminarPaciente.js
│   ├── useFecha.js, useConfirmacion.js
│   ├── useDictado.js        # envoltura de Web Speech API
│   ├── useAsistenteConsulta.js  # orquesta dictado → IA → formulario
│   └── useAlertasAlergias.js    # cruza medicamentos y alergias, maneja "Mantener"
│
├── layouts/
│   ├── RecepcionLayout.vue
│   └── MedicoLayout.vue
│
├── views/
│   ├── LoginView.vue
│   ├── EnConstruccionView.vue   # provisional para rutas de fases futuras
│   ├── recepcion/                # DashboardView, PacientesView, PacienteDetalleView, MedicosView, ConsultoriosView, AgendaView
│   └── medico/                    # MiAgendaView, AtencionView, HistoriaView, RecetaView
│
├── components/
│   ├── comunes/     # MarcoAplicacion, AlertaAlergias, FranjaAlergias, ChipAlergia, MarcaClinica, HojaCopias, CampoError, CampoTexto, PaginaListado, DialogoFormulario, BotonEditar, ...
│   ├── pacientes/   # PacienteForm, AlergiasField, AvisoNoEliminable, BuscadorPacientes, FichaPaciente
│   ├── medicos/     # MedicoForm, HorarioFields
│   ├── citas/       # CalendarioCitas (FullCalendar), EventoCita, BarraCalendario, FiltroMedico, LeyendaCitas, IconoAlergia, DetalleCita, DetalleCitaDialog, CitaForm, ChipEstadoCita, FiltroEstadosCita
│   ├── consultas/   # CabeceraAtencion, UltimaConsulta, ConsultaForm, DiagnosticosField (CIE-10), TarjetaConsulta
│   ├── recetas/     # RecetaForm, AlertaAlergiaMedicamento, VistaPreviaReceta
│   └── asistente/   # PanelAsistente, ControlesDictado, MarcaSugerido
│
├── pdf/
│   ├── comunes.js    # membrete, estilos y pie compartidos
│   └── recetaPdf.js  # documentoReceta, descargarRecetaPdf, imprimirRecetaPdf, nombreArchivoReceta
│
├── data/
│   ├── seed.js  # datos semilla + transcripción de demo
│   └── cie10.js # lista reducida de códigos CIE-10
│
├── utils/
│   ├── validaciones.js
│   ├── formato.js    # formatearFecha, calcularEdad, etiquetas de estado de cita, ...
│   ├── roles.js       # ruta de inicio por rol
│   ├── alergias.js    # cruce receta vs. alergias (local, sin IA)
│   └── anonimizar.js  # quita datos identificables antes de llamar a la IA
│
└── assets/
    ├── main.css          # tokens de diseño y estilos base
    └── presetClinica.js  # preset de tema de PrimeVue
```

`tests/` en la raíz replica esta estructura con una carpeta por capa testeada (ver [Tests](#tests)).

> Nota: `src/pdf/` todavía no incluye un `historiaPdf.js`; la exportación de la historia clínica a PDF (prevista pero aún sin implementar) aún no está implementada. La receta sí genera PDF completo (descargar e imprimir).

## Modelo de datos

Cada entidad se guarda como un array JSON en su propia clave de `localStorage`, con prefijo `clinica_`. Todos los registros tienen `id` (`crypto.randomUUID()`) y `creadoEn` (fecha ISO); las relaciones se hacen por id.

| Clave | Contenido |
|---|---|
| `clinica_usuarios` | `{ id, email, password, nombre, rol, medicoId? }` — contraseña en texto plano (simulado, académico) |
| `clinica_pacientes` | `{ id, dni, nombres, apellidos, fechaNacimiento, sexo, telefono, email, direccion, grupoSanguineo, alergias: [], antecedentes }` |
| `clinica_medicos` | `{ id, nombres, apellidos, especialidad, colegiatura, telefono, consultorioId, horario: { dias, inicio, fin } }` |
| `clinica_consultorios` | `{ id, nombre, piso, descripcion }` |
| `clinica_citas` | `{ id, pacienteId, medicoId, consultorioId, fecha, hora, duracionMin, motivo, estado }` — estado: `programada`, `confirmada`, `atendida`, `cancelada`, `no_asistio` |
| `clinica_consultas` | `{ id, citaId, pacienteId, medicoId, fecha, motivo, signosVitales, examenFisico, diagnosticos: [], plan, observaciones }` — más `generadaConIA` y `transcripcion` si se generó con el asistente |
| `clinica_recetas` | `{ id, consultaId, pacienteId, medicoId, fecha, items: [], indicacionesGenerales }` |
| `clinica_sesion` | `{ usuarioId, rol, nombre, medicoId? }` del usuario logueado |
| `clinica_version` | número de versión de los datos semilla |

### Reglas de negocio principales

- Un médico no puede tener dos citas que se solapen en el mismo horario (se valida al crear y al reprogramar).
- Las citas solo se agendan dentro del horario de atención del médico y en fechas no pasadas.
- El DNI del paciente es único (8 dígitos).
- Solo se registra una consulta para una cita en estado `programada` o `confirmada`.
- Una receta siempre pertenece a una consulta existente.
- No se eliminan pacientes con consultas registradas; se muestra un mensaje explicando el motivo.
- Antes de agendar o recetar, se muestra un aviso visible con las alergias del paciente.

## Diseño e interfaz

- Componentes de **PrimeVue** tematizados con `@primeuix/themes` y `src/assets/presetClinica.js`: `DataTable` para listados, `Dialog` para formularios, `Toast` para notificaciones y `ConfirmDialog` para acciones destructivas.
- Un layout por rol (`RecepcionLayout.vue`, `MedicoLayout.vue`), con menú lateral y área de contenido, compartiendo la base `MarcoAplicacion.vue`.
- Los tokens visuales (colores, tipografía, radios, espaciado) viven como variables `--color-*` etc. en `src/assets/main.css`; el preset de PrimeVue las referencia en vez de repetir valores hex.
- El calendario de citas usa FullCalendar (`CalendarioCitas.vue`), con un modo de solo lectura para la agenda del médico.
- Diseño responsive, pensado para laptop y tablet.
- Reglas visuales propias del dominio: el rojo se reserva para alergias, errores y grabación de voz; lo sugerido por la IA se marca "a lápiz" y lo revisado por el médico "en tinta"; las acciones destructivas piden confirmación en dos pasos.
- La fuente de verdad del diseño (tokens, composición de cada pantalla) está en `DESIGN.md` y `docs/diseno/`.

## PDFs

Generados con `pdfmake` (`src/pdf/comunes.js` + `src/pdf/recetaPdf.js`).

### Receta médica

- Membrete de la clínica, datos del médico (nombre, especialidad, colegiatura) y del paciente (nombre, DNI, edad, fecha).
- Tabla de medicamentos: medicamento, dosis, frecuencia, duración y vía.
- Indicaciones generales, línea de firma del médico y pie con el id de la receta (con QR opcional).
- Nombre de archivo: `receta_<apellido>_<YYYY-MM-DD>.pdf`.
- Dos acciones: **Descargar** e **Imprimir** (abre en una pestaña nueva).

La exportación de la historia clínica a PDF está prevista en el diseño del proyecto pero **no está implementada** todavía en el código.

## Asistente de consulta por voz (IA)

Función destacada del proyecto: el médico dicta la consulta y la app prellena el formulario de consulta y un borrador de receta. **La IA solo sugiere; el médico siempre revisa y confirma antes de guardar.**

### Flujo

1. En "Atender cita", el médico presiona el micrófono y dicta (o pega/escribe el texto si el navegador no soporta reconocimiento de voz).
2. `useDictado.js` transcribe con Web Speech API (`es-PE`, `continuous: true`, `interimResults: true`) en un área de texto editable, con acciones de iniciar, pausar, reanudar, detener y limpiar.
3. El médico presiona "Generar con IA". `useAsistenteConsulta.js` orquesta el resto del flujo:
   - `anonimizar.js` arma el contexto del paciente (edad, sexo, alergias, antecedentes). **Nunca** se envían nombre, DNI, teléfono, email ni dirección.
   - `iaService.js` hace `fetch` POST a `VITE_IA_SERVIDOR_URL` con `{ texto, contexto }` y un timeout de 30 s (`AbortController`).
   - La respuesta se extrae de forma robusta (quita bloques ```` ```json ```` si vienen incluidos) y se valida contra el contrato; si el JSON es inválido o le faltan campos, se muestra un error amigable y la transcripción queda intacta para reintentar.
4. Los campos sugeridos se cargan en el formulario de consulta y en el borrador de receta, marcados como "Sugerido por IA" (`MarcaSugerido.vue`); todos son editables. Un código CIE-10 que no existe en `data/cie10.js` se marca "código a verificar".
5. `utils/alergias.js` cruza cada medicamento sugerido con las alergias del paciente (lógica local, no de la IA), usando una tabla simple de familias (penicilina, AINEs, sulfas). Si hay coincidencia, `useAlertasAlergias.js` bloquea con una alerta roja que exige confirmación explícita ("Mantener" o quitar el medicamento).
6. El médico revisa y presiona "Guardar consulta" (deshabilitado mientras haya alertas de alergia sin resolver), lo que habilita "Generar receta PDF".

### Contrato de la respuesta de la IA

```json
{
  "motivo": "string|null",
  "signosVitales": { "presion": "string|null", "frecuenciaCardiaca": "number|null", "temperatura": "number|null", "peso": "number|null", "talla": "number|null" },
  "examenFisico": "string|null",
  "diagnosticos": [{ "codigo": "string|null", "descripcion": "string" }],
  "plan": "string|null",
  "receta": [{ "medicamento": "string", "dosis": "string|null", "frecuencia": "string|null", "duracion": "string|null", "via": "string|null", "indicaciones": "string|null" }],
  "indicacionesPaciente": "string|null",
  "advertencias": ["datos faltantes o dudas que el médico debería revisar"]
}
```

`indicacionesPaciente` se vuelca en el borrador de receta y `advertencias` se muestra en un panel visible.

### Manejo de errores (`iaService.js`)

| Caso | Mensaje |
|---|---|
| Sin conexión / servidor no disponible | "No se pudo conectar con el servidor de IA. Revise su conexión o intente más tarde." |
| Timeout (30 s) | "El servidor de IA tardó más de 30 segundos en responder." |
| Error del servidor (5xx) | "El servidor de IA tuvo un error. Intente de nuevo en unos minutos." |
| Límite de uso (429) | "Se alcanzó el límite de uso del servidor de IA. Espere un momento y reintente." |
| JSON mal formado o incompleto | "La respuesta de la IA no tiene el formato esperado. Puede reintentar." |
| `VITE_IA_SERVIDOR_URL` vacía | El botón "Generar con IA" queda deshabilitado; la atención funciona igual, de forma manual |

### Texto de demo

`src/data/seed.js` incluye una transcripción de ejemplo (`TRANSCRIPCION_DEMO`) cargable con el botón "Usar ejemplo", para probar el asistente sin micrófono. Uno de los pacientes semilla es alérgico a la penicilina, para poder ver la alerta de alergias con la receta sugerida (amoxicilina).

## Estado actual

Fases de construcción del proyecto:

1. **Base** — completa: Vite + Vue 3, ESLint, Vitest, Router, Pinia, PrimeVue, layouts por rol, `storage.js`, semilla, login y guard de roles.
2. **Catálogos** — completa: CRUD de pacientes, médicos y consultorios.
3. **Citas** — completa: agenda con FullCalendar, crear/reprogramar/cancelar, validación de solapamiento y horario.
4. **Atención médica** — completa: agenda del médico, atención de cita, consulta con CIE-10, historia clínica.
5. **Recetas y PDFs** — completa para la receta (formulario + PDF); el PDF de historia clínica queda pendiente.
6. **Cierre** — parcial: el dashboard de recepción ya existe; exportar/importar JSON, restablecer demo y el pulido final de esa fase siguen pendientes (la ruta `/recepcion/respaldo` todavía usa la vista provisional).
7. **Asistente de consulta por voz (IA)** — completa: dictado, anonimización, `iaService.js`, alertas de alergia, integración en "Atender cita" y borrador de receta, texto de demo.
8. **Tests** — en curso: ya hay pruebas de `services/`, `utils/`, `data/`, `router/`, `pdf/` y algún `composable`/`view`; la cobertura se sigue ampliando (ver [Tests](#tests)).

## Tests

Con Vitest (`environment: jsdom`), en `tests/` replicando la estructura de `src/`. Cubren:

- **`services/`**: reglas de negocio (solapamiento de citas, horario, DNI único, estados de cita, receta ligada a una consulta), `storage.js`, `authService`, `semillaService`.
- **`utils/`**: `alergias.js`, `anonimizar.js`, `formato.js`, `validaciones.js`.
- **`router/`**: `acceso.js` (decisión del guard).
- **`data/`**: `seed.js`.
- **`pdf/`**: `comunes.js`, `recetaPdf.js`.
- **`composables/`** y **`views/`**: los casos que no dependen del navegador real (`useAsistenteConsulta`, `LoginView`).
- **`iaService.js`**: validación y extracción del JSON de respuesta, con `fetch` simulado.

Los componentes puramente visuales no tienen tests unitarios.

```bash
npm run test              # modo watch
npm run test -- --run     # una sola pasada
```

## Limitaciones conocidas

- Los datos solo existen en el navegador donde se usa la app; si se borran, se pierden (salvo respaldo exportado — función aún pendiente, ver fase 6).
- `localStorage` tiene un límite aproximado de 5 MB.
- La autenticación es simulada, con contraseñas sin cifrar: no apta para datos médicos reales.
- No hay sincronización entre dispositivos ni usuarios simultáneos.
- El asistente de IA depende de un servidor externo al proyecto; si no está disponible o configurado, la atención médica funciona igual, de forma manual.
- El dictado por voz depende del navegador (funciona mejor en Chrome o Edge) y requiere permiso de micrófono.
- Las sugerencias de la IA pueden contener errores; el médico es siempre responsable de verificarlas.
- La detección de alergias usa una tabla simplificada de familias de medicamentos y no reemplaza una base farmacológica real.
