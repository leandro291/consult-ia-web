# SETUP.md — Arquitectura, instalación y convenciones

Este archivo es la **fuente de verdad** para lo siguiente:
- Estructura de carpetas.
- Dependencias.
- Comandos.
- Nombres.
- Diseño visual, que delega en `DESIGN.md` y `docs/diseno/` (ver [Diseño](#diseño)).

Toda nueva carpeta, archivo o dependencia debe seguirlo. Si algo nuevo no encaja, se actualiza este archivo antes de implementarlo.

---

## Arquitectura

### Capas y dependencias permitidas

```
┌─────────────┐
│   views/    │  Pantallas por ruta. Delgadas: arman componentes y llaman stores/composables.
└──────┬──────┘
       │ usa
┌──────▼──────┐     ┌──────────────┐
│ components/ │     │ composables/ │  Lógica reactiva reutilizable (useXxx).
└──────┬──────┘     └──────┬───────┘
       │ usa               │ usa
┌──────▼───────────────────▼──────┐
│            stores/              │  Pinia, uno por dominio. Estado + cargando + error.
└──────────────┬──────────────────┘
               │ llama
┌──────────────▼──────────────────┐
│           services/             │  CRUD y reglas de negocio. Lanzan Error en español.
└──────────────┬──────────────────┘
               │ único acceso
┌──────────────▼──────────────────┐
│   services/storage.js → localStorage
└─────────────────────────────────┘
```

| Capa | Puede importar | No puede importar |
|---|---|---|
| `views/` | components, composables, stores, utils, pdf | services, storage |
| `layouts/` | components, composables, stores, utils | services, storage |
| `components/` | components, composables, stores, utils, data | services, storage |
| `composables/` | stores, services, utils | views, components |
| `stores/` | su propio servicio, utils | `localStorage` directo, views, components |
| `services/` | `storage.js`, otros services, utils, data | Vue, Pinia, `localStorage` directo (salvo `storage.js`) |
| `utils/` | otros utils (funciones puras) | todo lo demás |
| `data/` | nada del proyecto (funciones puras) | todo lo demás |
| `pdf/` | utils, data | stores, services |
| `router/` | views, layouts, stores, utils, `acceso.js` | services |
| `assets/` | nada del proyecto | todo lo demás (solo lo importa `main.js`) |
| `main.js` | todo (raíz de composición) | — |

> ¿Por qué? Si en el futuro hay un backend HTTP, solo se reemplaza `services/`. Lo demás no cambia.

### Árbol de carpetas

```
consult-ia/
├── SETUP.md                  # Este archivo: arquitectura, instalación, convenciones
├── README.md                 # Presentación del proyecto y limitaciones (fase 6)
├── PRODUCT.md                # Contexto de producto (usuarios, propósito, principios)
├── DESIGN.md                 # Sistema de diseño: tokens, componentes y reglas visuales
├── index.html                # Punto de entrada de Vite
├── package.json
├── vite.config.js            # Config de Vite + Vitest (environment: jsdom)
├── eslint.config.js          # ESLint flat config (JS + Vue)
├── .gitignore
├── .env.example               # Plantilla de variables de entorno (VITE_IA_SERVIDOR_URL)
│
├── docs/
│   └── diseno/               # Guía de pantallas y fuentes del canvas de diseño (referencia, no se compila)
│
├── public/                   # Archivos estáticos servidos tal cual (favicon, logo)
│
├── src/
│   ├── main.js               # Crea la app, registra Router, Pinia, PrimeVue; ejecuta la semilla
│   ├── App.vue               # Raíz: <RouterView>, Toast y ConfirmDialog globales
│   │
│   ├── router/
│   │   ├── index.js          # Rutas con meta.rol + guard global de sesión y rol
│   │   └── acceso.js         # resolverAcceso(): decisión pura del guard
│   │
│   ├── services/             # Única capa con persistencia
│   │   ├── storage.js        # leer(clave, porDefecto = []), guardar(clave, datos), CLAVES: único acceso a localStorage
│   │   ├── semillaService.js # cargarSemilla, inicializarDatos (persiste los datos de seed.js)
│   │   ├── pacientesService.js
│   │   ├── medicosService.js
│   │   ├── consultoriosService.js
│   │   ├── citasService.js   # Incluye validación de solapamiento y horario
│   │   ├── consultasService.js
│   │   ├── recetasService.js
│   │   ├── authService.js    # login (la sesión incluye medicoId cuando el usuario es médico), logout, sesionActual
│   │   ├── respaldoService.js # Exportar e importar JSON
│   │   └── iaService.js      # POST al servidor de IA (URL de `VITE_IA_SERVIDOR_URL`) y validación
│   │
│   ├── stores/               # Pinia, uno por dominio + auth
│   │   ├── auth.js
│   │   ├── pacientes.js
│   │   ├── medicos.js
│   │   ├── consultorios.js
│   │   ├── citas.js
│   │   ├── consultas.js
│   │   └── recetas.js
│   │
│   ├── composables/          # Lógica reactiva reutilizable
│   │   ├── useCatalogo.js    # Estado y ejecución comunes de los stores de catálogo, de citas, de consultas y de recetas
│   │   ├── useEliminarPaciente.js # Eliminación de paciente en dos pasos (confirmar o avisar)
│   │   ├── useFecha.js
│   │   ├── useConfirmacion.js
│   │   ├── useDictado.js     # Envoltura de Web Speech API
│   │   ├── useAsistenteConsulta.js # Orquesta dictado → IA → formulario y arma el borrador de receta
│   │   └── useAlertasAlergias.js # Cruza medicamentos y alergias del paciente y maneja la confirmación de "Mantener"
│   │
│   ├── layouts/              # Un layout por rol (menú + <RouterView>)
│   │   ├── RecepcionLayout.vue
│   │   └── MedicoLayout.vue
│   │
│   ├── views/                # Una vista por ruta
│   │   ├── LoginView.vue
│   │   ├── EnConstruccionView.vue # Provisional para rutas de fases futuras
│   │   ├── recepcion/        # ConsultoriosView, MedicosView, DashboardView, PacientesView, PacienteDetalleView, AgendaView, ...
│   │   └── medico/           # MiAgendaView, AtencionView, HistoriaView, RecetaView, ...
│   │
│   ├── components/           # Componentes reutilizables, agrupados por dominio
│   │   ├── comunes/          # Piezas genéricas: MarcoAplicacion (esqueleto por rol: menú lateral + contenido), AlertaAlergias, FranjaAlergias (franja roja de todo el ancho de Atender cita y Emitir receta), ChipAlergia, MarcaClinica (logo + nombre), HojaCopias (hoja con copias apiladas), CampoError, CampoTexto (etiqueta + InputText + error), PaginaListado (cabecera, hoja con tabla, slot de pie y estado vacío), DialogoFormulario, BotonEditar, ...
│   │   ├── pacientes/        # PacienteForm.vue, AlergiasField.vue, AvisoNoEliminable.vue, BuscadorPacientes.vue, FichaPaciente.vue (ficha de la Historia clínica: datos y antecedentes)
│   │   ├── medicos/          # MedicoForm.vue, HorarioFields.vue
│   │   ├── citas/            # ResumenSemana.vue (hoja "Resumen de la semana" del panel del día: dos gráficos con Chart de PrimeVue), CalendarioCitas.vue (FullCalendar; con soloLectura para la agenda del médico: sin filtro, sin arrastrar ni crear, abre en Día), EventoCita.vue (contenido de la cita en modo soloLectura, con "Atender" y, si está atendida, "Ver historia"), BarraCalendario.vue, FiltroMedico.vue, LeyendaCitas.vue, IconoAlergia.vue, DetalleCita.vue (popover de solo lectura), DetalleCitaDialog.vue (diálogo de detalle de una cita en la agenda del médico, vista Semana, con "Ver historia clínica" y la acción según estado: "Atender cita" o "Generar receta PDF"), CitaForm.vue (diálogo de alta), ChipEstadoCita.vue (chip de estado), FiltroEstadosCita.vue (pestañas-filtro por estado)
│   │   ├── consultas/        # CabeceraAtencion.vue (cabecera de Atender cita), UltimaConsulta.vue (panel con la consulta más reciente del paciente), ConsultaForm.vue (formulario de consulta y barra "Guardar consulta"), DiagnosticosField.vue (lista editable de diagnósticos con buscador CIE-10), TarjetaConsulta.vue (tarjeta de una consulta en la línea de tiempo de la Historia clínica, expandida o resumida)
│   │   ├── recetas/          # RecetaForm.vue (filas de medicamentos, agregar/quitar e indicaciones generales; título, etiqueta de indicaciones y marcas "Sugerido por IA" configurables por prop para reutilizarse como borrador de receta en "Atender cita"), AlertaAlergiaMedicamento.vue (bloque rojo "Posible reacción alérgica" con la casilla de confirmar, "Mantener" y "Quitar medicamento"), VistaPreviaReceta.vue (hoja de vista previa de la receta, en vivo)
│   │   └── asistente/        # PanelAsistente.vue (panel de "Atender cita"), ControlesDictado.vue (usa useDictado), MarcaSugerido.vue (marca "Sugerido por IA")
│   │
│   ├── pdf/                  # Plantillas pdfmake
│   │   ├── comunes.js        # Membrete, estilos y pie compartidos
│   │   ├── recetaPdf.js
│   │   └── historiaPdf.js
│   │
│   ├── data/                 # Datos estáticos
│   │   ├── seed.js           # Datos semilla + transcripción de demo
│   │   └── cie10.js          # Lista reducida de códigos CIE-10
│   │
│   ├── utils/                # Funciones puras, sin Vue ni persistencia
│   │   ├── validaciones.js
│   │   ├── estadisticas.js   # Conteos de citas por día y estado y por médico (gráficos del panel del día)
│   │   ├── formato.js        # formatearFecha, calcularEdad, ETIQUETAS_ESTADO_CITA, ESTADOS_MOVIBLES_CITA, formatearHorarioCita
│   │   ├── roles.js          # Ruta de inicio por rol
│   │   ├── alergias.js       # Cruce receta vs alergias (local, sin IA)
│   │   └── anonimizar.js     # Quita datos identificables antes de llamar a la IA
│   │
│   └── assets/               # Estilos globales y tema: main.css (tokens de DESIGN.md, base) y presetClinica.js (preset de PrimeVue)
│
└── tests/                    # Vitest, replica la estructura de src/
    ├── services/             # citasService.test.js, pacientesService.test.js, ...
    ├── router/               # acceso.test.js
    ├── views/                # LoginView.test.js
    ├── data/                 # seed.test.js
    ├── utils/                # alergias.test.js, anonimizar.test.js, estadisticas.test.js, validaciones.test.js
    ├── pdf/                  # comunes.test.js, ...
    └── composables/          # Solo cuando la lógica no dependa del navegador real
```

**Qué se testea (mínimo, fase 8, en curso):**
- Reglas de negocio de `services/`:
  - Solapamiento de citas.
  - Horario.
  - DNI único.
  - Estados de cita.
  - Receta ligada a consulta.
- Todas las funciones de `utils/`.
- Validación y extracción del JSON de `iaService.js`, con `fetch` simulado.

Los componentes visuales no requieren tests unitarios.

### Organización de los tests

- Viven en `tests/` en la raíz, con una carpeta por cada capa de `src/` que tenga funciones testeables (`services/`, `utils/`, `data/`, `router/`, `pdf/`, `composables/`, `views/`), espejando el nombre de la carpeta de origen.
- Cada archivo se llama igual que el archivo que testea, con el sufijo `.test.js` (ej. `src/services/citasService.js` → `tests/services/citasService.test.js`), según la tabla de [Convenciones de nombres](#convenciones-de-nombres).
- El catálogo de funciones a probar es [`docs/funciones-testeables.md`](docs/funciones-testeables.md).

---

## Instalación y comandos

> Estado actual: el proyecto Vue está generado y funcional (Vite + Vue 3 en JS, Router, Pinia, PrimeVue tematizado con el preset de la clínica, login simulado, layouts por rol con menú lateral y datos semilla). Incluye la agenda de recepción (FullCalendar), el panel del día con el resumen de la semana, Mi agenda del médico, Atender cita con asistente de consulta por voz e IA, historia clínica, recetas en PDF y alertas de alergias. Las rutas de funciones futuras usan `EnConstruccionView.vue`.

### Instalación por fase

No se usa `npm create vite`: los archivos base se escriben a mano y cada fase instala solo lo que usa.

```bash
# Fase 1
npm install vue vue-router pinia primevue dayjs
npm install -D vite @vitejs/plugin-vue eslint @eslint/js eslint-plugin-vue globals vitest jsdom

# Fase 1 — diseño del login
npm install @primeuix/themes

# Fase 3 (agenda)
npm install @fullcalendar/vue3 @fullcalendar/core @fullcalendar/daygrid \
  @fullcalendar/timegrid @fullcalendar/interaction

# Fase 5 (PDFs)
npm install pdfmake

# Fase 6 (gráficos del panel del día)
npm install chart.js
```

Íconos: SVG inline de trazo 1.75; no se usa `primeicons`.

### Variables de entorno

Copiar `.env.example` a `.env` y completar `VITE_IA_SERVIDOR_URL` con la URL del servidor de IA. Reiniciar `npm run dev` o volver a hacer `npm run build` para que tome el cambio: Vite fija las variables `VITE_*` en build time, no en tiempo de ejecución. Vacía o ausente, el asistente de IA queda desactivado y la atención médica funciona igual, de forma manual. Como la variable queda visible en el JS del build, no lleva secretos.

### Scripts de `package.json`

| Script | Comando | Uso |
|---|---|---|
| `dev` | `vite` | Servidor de desarrollo |
| `build` | `vite build` | Build de producción en `dist/` |
| `preview` | `vite preview` | Sirve el build localmente |
| `lint` | `eslint .` | Lint de JS y Vue |
| `test` | `vitest` | Tests en modo watch; `npm run test -- --run` para una sola pasada |

### Configuración mínima esperada

**`eslint.config.js`**

```js
import js from '@eslint/js'
import pluginVue from 'eslint-plugin-vue'
import globals from 'globals'

export default [
  { ignores: ['dist/**'] },
  js.configs.recommended,
  ...pluginVue.configs['flat/recommended'],
  { languageOptions: { globals: { ...globals.browser, ...globals.node } } }
]
```

**`vite.config.js`**

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { fileURLToPath, URL } from 'node:url'

export default defineConfig({
  plugins: [vue()],
  resolve: { alias: { '@': fileURLToPath(new URL('./src', import.meta.url)) } },
  test: { environment: 'jsdom' }
})
```

Los imports internos usan el alias `@/` (ej. `import { leer } from '@/services/storage.js'`).

---

## Dependencias

### Runtime

| Paquete | Propósito |
|---|---|
| `vue` | Framework (Composition API, `<script setup>`) |
| `vue-router` | Rutas y guard de roles |
| `pinia` | Estado global, un store por dominio |
| `primevue` | Componentes UI (Toast, ConfirmDialog, DataTable, Dialog, …), tematizado con `src/assets/presetClinica.js` |
| `@primeuix/themes` | Preset del tema de PrimeVue (base Aura) con los tokens de `DESIGN.md` |
| `@fullcalendar/vue3`, `@fullcalendar/core` | Calendario de citas |
| `@fullcalendar/daygrid`, `@fullcalendar/timegrid` | Vistas de mes, semana y día |
| `@fullcalendar/interaction` | Arrastrar para reprogramar y hacer clic para crear |
| `dayjs` | Manejo de fechas (locale `es`) |
| `pdfmake` | PDF de receta e historia clínica; el QR de la receta usa su nodo `qr` nativo, sin la dependencia `qrcode` |
| `chart.js` | Gráficos del panel del día, usado vía el componente `Chart` de PrimeVue |

**Sin dependencia**, se usan APIs nativas:
- Web Speech API para el dictado.
- `fetch` para el servidor de IA.
- `crypto.randomUUID()` para los ids.
- `localStorage` para la persistencia.
- Fuentes Public Sans y Courier Prime desde Google Fonts (`<link>` en `index.html`).

### Desarrollo

| Paquete | Propósito |
|---|---|
| `vite`, `@vitejs/plugin-vue` | Build y servidor de desarrollo |
| `eslint`, `@eslint/js`, `globals` | Lint de JavaScript |
| `eslint-plugin-vue` | Reglas de lint para archivos `.vue` |
| `vitest` | Tests unitarios (usa la config de Vite) |
| `jsdom` | Simula el navegador (`localStorage`, `window`) en los tests |

Para agregar una dependencia nueva hace falta:
1. Justificarla en la descripción del cambio.
2. Agregarla a estas tablas.

---

## Convenciones de nombres

| Elemento | Convención | Ejemplo |
|---|---|---|
| Componente | `PascalCase.vue`, dos palabras o más | `PacienteForm.vue`, `AlertaAlergias.vue` |
| Vista (ruta) | `PascalCase` + `View.vue` | `AgendaView.vue`, `AtencionView.vue` |
| Layout | `PascalCase` + `Layout.vue` | `MedicoLayout.vue` |
| Composable | `use` + `PascalCase.js`, exporta función homónima | `useDictado.js` → `useDictado()` |
| Servicio | `camelCase` + `Service.js` | `citasService.js` |
| Store | `camelCase.js` en plural, exporta `use<Dominio>Store` | `citas.js` → `useCitasStore` |
| Util / data / pdf | `camelCase.js` | `anonimizar.js`, `recetaPdf.js` |
| Test | mismo nombre que el archivo + `.test.js` | `citasService.test.js` |
| Clave de localStorage | `clinica_` + entidad en plural, `snake_case` | `clinica_citas` |
| Variable CSS de diseño | `--` + grupo en español + nombre del token de `DESIGN.md` | `--color-tinta`, `--radio-md`, `--espacio-lg`, `--sombra-hoja`, `--fuente-dato` |
| Clase CSS global | kebab-case en español | `.campo`, `.campo-con-error`, `.tipo-label`, `.icono` |
| Ruta | minúsculas, segmentos en español | `/medico/atencion/:citaId` |
| Variables y funciones | `camelCase` **en español** | `citasDelDia`, `validarSolapamiento()` |
| Constantes | `UPPER_SNAKE_CASE` | `DURACION_CITA_MIN` |
| Eventos de componente | `kebab-case` en español | `@guardar`, `@cita-cancelada` |

**Idioma:** todo en español (identificadores, textos, errores, comentarios y commits). Las excepciones son las palabras reservadas y las APIs de librerías.

---

## Diseño

### Fuente de verdad

El diseño que se implementa es el que creamos para este proyecto. No se inventan estilos ni pantallas nuevas.

| Qué | Dónde | Manda sobre |
|---|---|---|
| Tokens (colores, tipografía, radios, espaciado), componentes y reglas visuales | [`DESIGN.md`](DESIGN.md) | todo lo visual |
| Composición de cada pantalla (qué va dónde, textos, estados) | [`docs/diseno/pantallas/`](docs/diseno/pantallas/) (guía en [`docs/diseno/README.md`](docs/diseno/README.md)) | la maqueta de cada ruta |
| Contexto de producto (usuarios, principios) | [`PRODUCT.md`](PRODUCT.md) | decisiones de UX |

- **Si hay conflicto:** `DESIGN.md` manda sobre las pantallas y ambos mandan sobre cualquier guía de diseño genérica.
- **El canvas en línea es una copia.** Si difiere del repo, vale el repo.
- **Datos de ejemplo:** los nombres de pacientes y médicos de las pantallas son ilustrativos; la app usa los de `src/data/seed.js`.

### Cómo se implementa

- Los hex de `DESIGN.md` viven en un solo lugar: las variables `--color-*` de `src/assets/main.css`. El preset de `@primeuix/themes` (`src/assets/presetClinica.js`) las referencia con `var(--color-...)` y no repite hex; los componentes tampoco escriben colores hex sueltos.
- `main.css` declara `@layer base, primevue;`. PrimeVue se registra en `main.js` con `cssLayer: { name: 'primevue', order: 'base, primevue' }`, así el reset de la capa `base` no pisa a PrimeVue, y los ajustes de componentes sin capa (`.p-inputtext`, `.p-button:disabled`) sí ganan sobre los estilos que PrimeVue inyecta.
- Se usan los componentes de PrimeVue tematizados para que se vean como en `DESIGN.md`; no se reemplazan por componentes propios.
- Cambiar el diseño implica actualizar primero `DESIGN.md` (y la pantalla en `docs/diseno/pantallas/` si cambia la composición).

### Antes de commitear cambios de interfaz

Se verifica, además de lint:

1. La pantalla sigue su archivo en `docs/diseno/pantallas/`: estructura, jerarquía, textos y estados (vacío, carga y error).
2. Solo se usan tokens de `DESIGN.md`; no aparecen colores, fuentes ni radios nuevos.
3. Se cumplen las reglas de `DESIGN.md`:
   - El rojo solo para alergias, errores y grabación.
   - Lo sugerido por la IA a lápiz y lo revisado en tinta.
   - Acciones destructivas en dos pasos.
   - Ningún estado indicado solo con color.
4. Accesibilidad: labels reales, foco visible, contraste AA y `aria-label` con contexto en las acciones repetidas.

Si algo del diseño no se puede implementar tal cual, se anota y se decide con el responsable del producto; no se improvisa.
