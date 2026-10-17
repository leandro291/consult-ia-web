# CLAUDE.md — Sistema de Clínica y Consultorios

## Descripción del proyecto

Aplicación web académica para gestionar una clínica con varios consultorios. Cubre:
- **Citas**: agenda por médico y consultorio.
- **Historias clínicas**: registro de consultas por paciente.
- **Recetas médicas**: se generan como PDF descargable e imprimible.

Es un proyecto **académico**: se priorizan código claro, bien organizado y fácil de explicar por encima de optimizaciones complejas.

## Metodología: Spec-Driven Development (SDD)

### Regla de orquestación

**Todo requerimiento del usuario se delega primero al subagente `orquestador`.** El hilo principal no implementa por su cuenta. El orquestador decide el flujo:

| Flujo | Cuándo |
|---|---|
| **SDD** (spec → aprobación → implementación → revisión) | Feature nueva, cambios en 3 o más archivos, cambios de arquitectura, modelo de datos o rutas. |
| **Implementación directa** (solo `developer`, sin spec) | Bugfix acotado, textos, estilos, renombres, cambios en un solo archivo. |
| **Pregunta al hilo principal** | Hay duda sobre el flujo o sobre el requerimiento. |

### Flujo SDD

1. `orquestador` → `spec-writer`: genera `specs/NNN-nombre.md` con `Estado: borrador`.
2. **STOP.** El orquestador devuelve la ruta del spec al hilo principal. Los subagentes no pueden preguntar al usuario.
3. El hilo principal muestra el spec al usuario. Si el usuario pide cambios, se vuelve al paso 1.
4. Con la aprobación, el hilo principal cambia el spec a `Estado: aprobado` y retoma al orquestador:
   - Preferido: `SendMessage` al mismo orquestador con "Spec NNN aprobado".
   - Alternativa: una nueva invocación con la ruta del spec aprobado.
5. `developer` implementa → `reviewer` revisa → si hay errores, vuelve al `developer` → `reviewer` … **máximo 3 ciclos**.
6. Si tras 3 ciclos siguen los errores, el orquestador se detiene y reporta al hilo principal.
7. Con `APROBADO`, el hilo principal:
   - Muestra el resultado al usuario.
   - **Si el spec toca una pantalla o componente visual**, el usuario prueba el flujo principal en el navegador antes de seguir: el `reviewer` no ejecuta la app (no hay herramienta de navegador headless disponible), solo lee código, lintea, buildea y corre los tests existentes. Bugs de interacción o de estilo que aparezcan ahí vuelven al `developer` como una corrección más, dentro del límite de 3 ciclos (o uno nuevo si ya se agotó).
   - Con el visto bueno, cambia el spec a `Estado: implementado`.
   - Hace el commit **solo con el OK del usuario**. Ningún subagente hace commits.

### Agentes

Definidos en `.claude/agents/`. Cada archivo tiene el detalle técnico de su rol.

| Agente | Rol | Tools | Modelo |
|---|---|---|---|
| [`orquestador`](.claude/agents/orquestador.md) | Punto de entrada. Clasifica y delega. | Agent(spec-writer, developer, reviewer), Read, Glob, Grep, Skill | sonnet |
| [`spec-writer`](.claude/agents/spec-writer.md) | Escribe el spec. No escribe código. | Read, Glob, Grep, Write, Edit, Skill | opus |
| [`developer`](.claude/agents/developer.md) | Implementa las tareas del spec aprobado. | Read, Glob, Grep, Write, Edit, Bash, Skill | sonnet |
| [`reviewer`](.claude/agents/reviewer.md) | Revisa en modo solo lectura. Devuelve APROBADO o errores. | Read, Glob, Grep, Bash, Skill | sonnet |

### Specs

- Ubicación: `specs/NNN-nombre-en-kebab.md`, con numeración correlativa de 3 dígitos (`001`, `002`, …).
- Campo `Estado:` con uno de estos valores: `borrador`, `aprobado` o `implementado`.
- Estructura obligatoria y límites de tamaño (máximo 120 líneas): la plantilla de [`spec-writer`](.claude/agents/spec-writer.md).

### Skills globales

Si una skill instalada globalmente (`~/.claude/skills/` o un plugin) aplica a la tarea, **úsala**. Esto vale para el hilo principal y para todos los agentes. Si la skill contradice algo, manda este orden de prioridad:

1. Este `CLAUDE.md`.
2. `SETUP.md`.
3. El spec aprobado.
4. La skill.

### SETUP.md (obligatorio)

[`SETUP.md`](SETUP.md) define:
- La arquitectura y el árbol de carpetas.
- Las dependencias y los comandos.
- Las convenciones de nombres.

**Toda nueva carpeta, archivo o dependencia debe seguir `SETUP.md`.** Si algo nuevo no encaja, el spec debe incluir primero la tarea de actualizar `SETUP.md`.

## Restricciones obligatorias

- **Sin base de datos ni backend.** Toda la persistencia se hace con `localStorage` del navegador.
- **JavaScript puro, NO TypeScript.** No crear archivos `.ts` ni usar `lang="ts"`.
- Vue 3 con **Composition API** y `<script setup>`.
- Todo el acceso a `localStorage` pasa por la capa de servicios (`src/services/`). Los componentes y stores **nunca** llaman a `localStorage` directamente.
- Todos los textos de la interfaz, mensajes y comentarios del código van en **español**.

## Stack tecnológico

- Vue 3 + Vite, Vue Router, Pinia y PrimeVue.
- FullCalendar, Day.js y pdfmake.
- ESLint y Vitest como herramientas de desarrollo.

La tabla completa con el propósito de cada dependencia está en [`SETUP.md`](SETUP.md#dependencias). No agregar otras dependencias sin una razón clara; si se agrega una, se documenta en `SETUP.md` dentro del mismo spec.

## Roles y vistas

Es una sola aplicación con login. Según el rol del usuario se muestra un layout y un menú distintos.

### Rol: `recepcion` (Recepción / Administración)
- **Dashboard**: citas del día, conteo por estado (programada, atendida, cancelada, no asistió).
- **Pacientes**: listar, buscar por DNI o nombre, crear, editar y ver detalle.
- **Médicos**: listar, crear y editar (especialidad, colegiatura, consultorio, horario).
- **Consultorios**: listar, crear y editar.
- **Agenda**: calendario con todas las citas, filtrable por médico. Permite crear, reprogramar y cancelar citas.
- **Respaldo**: exportar todos los datos a un archivo JSON e importarlos de vuelta.

### Rol: `medico`
- **Mi agenda**: solo las citas del médico logueado (vista del día y de la semana).
- **Atender cita**: muestra datos del paciente, alergias e historial, junto con el formulario de consulta. Al guardar, la cita pasa a estado `atendida`.
  - Incluye el **Asistente de consulta por voz**: dictado → IA → formulario y borrador de receta prellenados. La URL del servidor de IA es una variable de entorno de build (`VITE_IA_SERVIDOR_URL`), no una pantalla de configuración.
- **Historia clínica**: línea de tiempo de todas las consultas de un paciente, con opción de exportarla a PDF.
- **Emitir receta**: formulario de medicamentos ligado a una consulta. Genera el PDF.

## Rutas

```
/login
/recepcion/dashboard
/recepcion/pacientes
/recepcion/pacientes/:id
/recepcion/medicos
/recepcion/consultorios
/recepcion/agenda
/recepcion/respaldo
/medico/agenda
/medico/atencion/:citaId
/medico/historia/:pacienteId
/medico/receta/:consultaId
```

- Un **navigation guard** global revisa la sesión:
  - Sin sesión → redirigir a `/login`.
  - Rol incorrecto para la ruta → redirigir al inicio de su propio rol.
- Cada ruta declara su rol con `meta: { rol: 'recepcion' | 'medico' }`.

## Modelo de datos (claves de localStorage)

Cada entidad se guarda como un array JSON en su propia clave, con prefijo `clinica_`. Todos los registros tienen:
- `id`, generado con `crypto.randomUUID()`
- `creadoEn`, fecha ISO

Las relaciones se hacen por id.

- **`clinica_usuarios`**: `{ id, email, password, nombre, rol, medicoId? }`
  - La contraseña va en texto plano: es simulado y académico. Documentarlo como limitación.
- **`clinica_pacientes`**: `{ id, dni, nombres, apellidos, fechaNacimiento, sexo, telefono, email, direccion, grupoSanguineo, alergias: [], antecedentes }`
- **`clinica_medicos`**: `{ id, nombres, apellidos, especialidad, colegiatura, telefono, consultorioId, horario: { dias: [1..6], inicio: '08:00', fin: '14:00' } }`
- **`clinica_consultorios`**: `{ id, nombre, piso, descripcion }`
- **`clinica_citas`**: `{ id, pacienteId, medicoId, consultorioId, fecha: 'YYYY-MM-DD', hora: 'HH:mm', duracionMin: 30, motivo, estado: 'programada' | 'confirmada' | 'atendida' | 'cancelada' | 'no_asistio' }`
- **`clinica_consultas`**: `{ id, citaId, pacienteId, medicoId, fecha, motivo, signosVitales: { presion, frecuenciaCardiaca, temperatura, peso, talla }, examenFisico, diagnosticos: [{ codigo, descripcion }], plan, observaciones }`
- **`clinica_recetas`**: `{ id, consultaId, pacienteId, medicoId, fecha, items: [{ medicamento, dosis, frecuencia, duracion, via, indicaciones }], indicacionesGenerales }`
- **`clinica_sesion`**: `{ usuarioId, rol, nombre, medicoId? }`, el usuario logueado actualmente. `medicoId` solo está presente cuando `rol` es `medico`.
- Las consultas creadas con ayuda del asistente llevan además:
  - `generadaConIA: true`
  - `transcripcion`: el texto dictado original
- **`clinica_version`**: número de versión de los datos semilla.

## Reglas de negocio

- Un médico **no puede tener dos citas que se solapen** en el mismo horario. Validar al crear y al reprogramar.
- Solo se agendan citas **dentro del horario de atención** del médico y en fechas no pasadas.
- El DNI del paciente es **único** (8 dígitos).
- Solo se puede registrar una consulta para una cita en estado `programada` o `confirmada`.
- Una receta siempre pertenece a una consulta existente.
- No se eliminan pacientes que tengan consultas registradas; se muestra un mensaje explicando el motivo.
- Antes de agendar o recetar, mostrar un aviso visible con las alergias del paciente.

## Estructura de carpetas

El árbol completo de carpetas, con el propósito de cada una, está en [`SETUP.md`](SETUP.md#arquitectura).

Resumen de las capas:

```
views → stores → services → storage.js → localStorage
```

- Las vistas usan componentes y composables.

## Capa de servicios (patrón a seguir)

```js
// services/storage.js
export function leer(clave) {
  try {
    return JSON.parse(localStorage.getItem(clave)) ?? []
  } catch {
    return []
  }
}

export function guardar(clave, datos) {
  localStorage.setItem(clave, JSON.stringify(datos))
}
```

- Cada servicio de entidad expone funciones CRUD que usan `leer` y `guardar`.
- Los servicios devuelven copias de los datos y lanzan `Error` con mensajes en español cuando se rompe una regla de negocio.
- Los stores de Pinia llaman a los servicios y manejan los estados `cargando` y `error`.

La intención es que, si en el futuro hay un backend, solo se reemplacen los servicios.

## Datos semilla

Al iniciar la app, si `clinica_version` no existe, `seed.js` carga:
- 3 consultorios.
- 3 médicos de distintas especialidades: Medicina General, Pediatría y Cardiología.
- 1 usuario de recepción: `recepcion@clinica.com` / `123456`.
- 1 usuario por médico: `medico1@clinica.com` / `123456`, etc.
- ~10 pacientes.
- ~15 citas distribuidas en la semana actual, en distintos estados.
- Algunas consultas y recetas ya registradas, para que las historias no aparezcan vacías.

Agregar un botón **"Restablecer datos de demo"** en la sección de Respaldo.

## PDFs (pdfmake)

### Receta médica
- **Membrete**: nombre de la clínica, dirección y teléfono.
- **Médico**: nombre, especialidad y número de colegiatura.
- **Paciente**: nombre, DNI, edad y fecha.
- **Medicamentos**: tabla con medicamento, dosis, frecuencia, duración y vía.
- **Cierre**:
  - Indicaciones generales.
  - Línea de firma del médico.
  - Pie con el id de la receta.
  - QR opcional con el id de la receta.
- Nombre de archivo: `receta_<apellido>_<YYYY-MM-DD>.pdf`.
- Ofrecer dos botones: **Descargar** e **Imprimir** (abrir en una pestaña nueva).

### Historia clínica
- Datos del paciente, alergias y antecedentes.
- Todas las consultas en orden cronológico, cada una con sus diagnósticos y signos vitales.

## Asistente de consulta por voz (IA)

Es la función destacada del proyecto. Imita a un "escriba clínico con IA": el médico dicta la consulta en lenguaje natural y la app prellena el formulario de consulta y un borrador de receta. **La IA solo sugiere; el médico siempre revisa y confirma.**

### Flujo

1. En `/medico/atencion/:citaId`, el médico presiona el botón de micrófono y dicta.
2. `useDictado.js` transcribe en tiempo real con Web Speech API.
   - Idioma `es-PE`, `continuous: true`, `interimResults: true`.
   - La transcripción se muestra en un área de texto editable.
   - Botones: iniciar, pausar, detener y limpiar.
   - Si el navegador no soporta la API, ocultar el micrófono y dejar solo el área de texto. Así el médico puede escribir o pegar la nota.
3. El médico presiona **"Generar con IA"**.
4. `anonimizar.js` prepara el contexto: edad, sexo, alergias y antecedentes del paciente. **Nunca** se envían nombre, DNI, teléfono, email ni dirección.
5. `iaService.js` envía la transcripción y el contexto a la IA y recibe un JSON (ver formato abajo).
6. La respuesta se valida:
   - Si el JSON es inválido o faltan campos, mostrar un error amigable y dejar la transcripción intacta para reintentar.
7. Los campos se cargan en el formulario de consulta y en el borrador de receta, resaltados como "Sugerido por IA". Todos son editables.
8. `alergias.js` cruza cada medicamento sugerido con las alergias del paciente. Esta verificación es **lógica local, no de la IA**.
   - Usar una tabla simple de familias: por ejemplo, penicilina → amoxicilina, ampicilina; AINEs → ibuprofeno, naproxeno, diclofenaco; sulfas → sulfametoxazol.
   - Si hay coincidencia, mostrar una alerta roja bloqueante que exige confirmación explícita.
9. El médico revisa y presiona **"Guardar consulta"**, lo que habilita **"Generar receta PDF"**.

### Formato de respuesta esperado de la IA

El servidor de IA responde **solo con JSON válido** con esta forma exacta (es el contrato entre el front y el servidor). Los campos sin información van como `null` o como array vacío. La IA **nunca inventa** datos que no estén en la transcripción.

```json
{
  "motivo": "string|null",
  "signosVitales": {
    "presion": "string|null",
    "frecuenciaCardiaca": "number|null",
    "temperatura": "number|null",
    "peso": "number|null",
    "talla": "number|null"
  },
  "examenFisico": "string|null",
  "diagnosticos": [
    { "codigo": "string|null", "descripcion": "string" }
  ],
  "plan": "string|null",
  "receta": [
    {
      "medicamento": "string",
      "dosis": "string|null",
      "frecuencia": "string|null",
      "duracion": "string|null",
      "via": "string|null",
      "indicaciones": "string|null"
    }
  ],
  "indicacionesPaciente": "string|null",
  "advertencias": ["datos faltantes o dudas que el médico debería revisar"]
}
```

- Si `diagnosticos` trae un código, verificarlo contra `data/cie10.js`. Si no existe en la lista, marcarlo como "código a verificar".
- Mostrar `advertencias` en un panel visible.
- `indicacionesPaciente` se agrega al PDF de la receta.

### iaService.js

- Hace `fetch` POST a la URL de `import.meta.env.VITE_IA_SERVIDOR_URL` (variable de entorno de build, no se guarda en `localStorage` ni se muestra en ninguna pantalla), enviando `{ texto, contexto }` (el cuerpo usa la clave `texto` para la transcripción, según el contrato del servidor de IA del proyecto; el `contexto` es la salida de `anonimizar.js`; nunca nombre, DNI, teléfono, email ni dirección).
- Recibe como respuesta el JSON con el formato del contrato (ver arriba).
- El asistente se considera activo si `VITE_IA_SERVIDOR_URL` tiene valor. Si está vacía, el botón "Generar con IA" queda deshabilitado con un texto que explica que el asistente no está disponible.
- Parámetros de la llamada:
  - Timeout de 30 s con `AbortController`.
- Extraer el JSON de la respuesta de forma robusta: quitar bloques ```json si vienen incluidos.
- Manejar los errores con mensajes en español: sin conexión o servidor no disponible, error del servidor (5xx), límite de uso (429), timeout y respuesta mal formada.

### Interfaz

- Aviso fijo en el panel del asistente: *"Sugerencias generadas por IA. Verifique toda la información antes de guardar."*
- Estado de carga claro mientras la IA responde (spinner + "Analizando consulta…").
- Indicador visual de micrófono activo (punto rojo pulsante).
- Si la IA está desactivada o falla, la atención médica funciona igual de forma manual.

### Texto de demo

Incluir en `data/seed.js` un ejemplo de transcripción para probar sin micrófono, cargable con el botón "Usar ejemplo":

> "Paciente refiere fiebre desde hace tres días y dolor de garganta al tragar. Temperatura 38.5, presión 120 sobre 80, frecuencia cardiaca 92. Amígdalas inflamadas con placas blanquecinas, sin tos. Impresión diagnóstica faringitis bacteriana. Indico amoxicilina 500 miligramos cada 8 horas por 7 días y paracetamol 500 miligramos cada 8 horas si hay fiebre. Control en una semana."

Para demostrar la alerta de alergias, uno de los pacientes semilla debe ser alérgico a la penicilina.

## Convenciones de código

Las convenciones de nombres de archivos, carpetas e identificadores están en [`SETUP.md`](SETUP.md#convenciones-de-nombres).

**Vue 3:**
- `<script setup>` en JS, siempre con Composition API.
- Un componente por archivo, **chico y con una sola responsabilidad**.
  - Si pasa de ~200 líneas o mezcla responsabilidades, se divide.
- **Vistas delgadas**:
  - Orquestan componentes y llaman a stores o composables.
  - No contienen reglas de negocio.
- **Composables** (`useXxx`): lógica reactiva reutilizable entre componentes (dictado, confirmaciones, formato de fechas).
- **Un store de Pinia por dominio**:
  - El store llama a su servicio y expone el estado, `cargando` y `error`.
  - Un store no importa otro servicio que no sea el suyo, salvo que el spec lo justifique.
- **Servicios**:
  - Son la única capa con persistencia, a través de `storage.js`.
  - Son funciones puras de dominio: sin Vue ni Pinia.
- Props hacia abajo y eventos hacia arriba. Estado compartido solo en stores.

**SOLID y DRY aplicados:**
- **S**: cada archivo tiene un motivo de cambio. El servicio persiste, el store maneja el estado, el componente muestra.
- **O/D**: stores y vistas dependen de la interfaz de los servicios (nombres de funciones), no de `localStorage`. Así se puede cambiar a un backend sin tocar otras capas.
- **DRY**:
  - Las validaciones van en `utils/validaciones.js`.
  - El formato de fechas y monedas va en `utils/formato.js`.
  - Membrete y estilos de PDF van en `pdf/comunes.js`.
  - Antes de crear un helper, se busca si ya existe.

**Otras reglas:**
- Fechas:
  - Guardarlas en formato ISO.
  - Formatearlas solo al mostrarlas, con Day.js en español (`DD/MM/YYYY`).
- Validar formularios antes de guardar y mostrar los errores junto a cada campo.
- Usar los componentes de PrimeVue:
  - `DataTable` para listados.
  - `Dialog` para formularios modales.
  - `Toast` para notificaciones.
  - `ConfirmDialog` para acciones destructivas.
- Diseño responsive, usable en laptop y tablet.
- Comentarios breves en español solo donde la lógica no sea obvia.

## Plan de desarrollo (por fases)

Completar y verificar cada fase antes de pasar a la siguiente. Cada fase se ejecuta como uno o más specs del flujo SDD.

1. **Base**: proyecto Vite + Vue 3 (JS), configuración de ESLint y Vitest (según `SETUP.md`), Router, Pinia, PrimeVue, layouts por rol, `storage.js`, datos semilla, login y guard de roles.
2. **Catálogos**: CRUD de pacientes, médicos y consultorios, con búsqueda y validaciones.
3. **Citas**:
   - Agenda con FullCalendar.
   - Crear, reprogramar (arrastrar) y cancelar citas.
   - Validación de solapamiento y de horario.
4. **Atención médica**: agenda del médico, pantalla de atención, registro de consulta con CIE-10 y línea de tiempo de la historia clínica.
5. **Recetas y PDFs**: formulario de receta, PDF de receta y PDF de historia clínica.
6. **Cierre**: dashboard con indicadores, exportar/importar JSON, restablecer demo, pulido visual y README.
7. **Asistente de consulta por voz (IA)**:
   - `useDictado.js`
   - `anonimizar.js`
   - `iaService.js` con validación de respuesta
   - `alergias.js`
   - `VITE_IA_SERVIDOR_URL` como variable de entorno de build (sin pantalla de configuración)
   - Integración en "Atender cita" (formulario y borrador de receta)
   - `indicacionesPaciente` en el PDF
   - Texto de demo
   
   Probar primero con el texto de demo y después con el micrófono.
8. **Tests**: con todo construido, tests de Vitest según [`SETUP.md`](SETUP.md). Hasta esta fase, `developer` y `reviewer` no escriben tests. El `reviewer` sí corre `npm run build` y los tests que ya existan (dentro del límite de 3 ciclos con el `developer`) para confirmar que lo implementado funciona; la falta de tests no es un error antes de esta fase.

## Limitaciones conocidas (documentar en el README)

- Los datos solo existen en el navegador donde se usa la app. Si se borran los datos del navegador, se pierden, salvo que se haya exportado un respaldo.
- localStorage tiene un límite aproximado de 5 MB.
- La autenticación es simulada, con contraseñas sin cifrar. No es apta para datos médicos reales.
- No hay sincronización entre dispositivos ni usuarios simultáneos.
- Límites del asistente IA:
  - El asistente depende de un servidor de IA externo a este proyecto; si no está disponible o no está configurado, la atención médica funciona igual de forma manual.
  - El dictado por voz depende del navegador (funciona mejor en Chrome o Edge) y requiere permiso de micrófono.
  - Las sugerencias de la IA pueden contener errores; el médico es siempre responsable de verificarlas.
  - La detección de alergias usa una tabla simplificada de familias de medicamentos y no reemplaza una base farmacológica real.

## Comandos

Instalación, scripts (`dev`, `build`, `lint`, `test`) y configuración: ver [`SETUP.md`](SETUP.md#instalación-y-comandos).
