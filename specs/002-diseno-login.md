# 002 — Diseño del login y base visual (tokens, tema y fuentes)

Estado: implementado
Fase: 1 (complemento visual)
Depende de: 001

## Contexto
El spec 001 dejó `LoginView.vue` como cascarón funcional: HTML plano, sin estilos, sin tema de PrimeVue ni fuentes (RNF7 de 001). El diseño ya existe:
- `DESIGN.md`: tokens (front matter) y reglas visuales.
- `docs/diseno/pantallas/Main.dc.html`: maqueta del login a 1366×768.
- `docs/diseno/pantallas/Componentes.dc.html` y `EstadosVaciosCarga.dc.html`: estados de campo, error y botón deshabilitado.

Este spec implementa **el login y solo lo que el login necesita** para verse como el diseño: tokens globales, tema de PrimeVue y fuentes. Las demás pantallas (layouts, menú, dashboard) quedan para specs siguientes, que reutilizan lo que se crea aquí.

Hoy:
- `src/main.js` registra PrimeVue sin tema y no importa CSS. No existen `src/assets/`, `main.css` ni preset.
- `index.html` no carga fuentes.
- `package.json` no tiene `@primeuix/themes` ni `primeicons`.
- `LoginView.vue`:
  - usa un checkbox "Mostrar contraseña" (`#login-mostrar`);
  - muestra el error de credenciales dos veces: un `<p role="alert">` y un Toast;
  - rotula el sitio como "Consult-IA".
- `tests/views/LoginView.test.js` usa `#login-email`, `#login-password`, el `submit` del `<form>` y `[role="alert"]`. No usa `#login-mostrar`.
- No existen `src/utils/formato.js` ni `src/composables/`.
- `src/data/seed.js` crea los usuarios `recepcion@clinica.com` y `medico1..3@clinica.com`, todos con `123456`. Coinciden con el recuadro "Cuentas de demostración" de la maqueta.

### Decisiones tomadas en este spec
Las marcadas con (*) también aparecen en "Decisiones abiertas" para que el usuario las confirme.

- **PrimeVue tematizado (`InputText`, `Button`) y no elementos nativos.** `SETUP.md` › Diseño dice que los tokens viven en el preset de `@primeuix/themes` y en `src/assets/main.css`, y que se usan los componentes de PrimeVue tematizados, sin reemplazarlos. Se agrega la dependencia `@primeuix/themes`. `SETUP.md` y `DESIGN.md` ya la nombran; falta documentarla en las tablas, lo que hace T1.
- **Sin `Password` de PrimeVue.** En `primevue@5`, el ícono de máscara de `Password` es un `<svg>` con `@click`: no es un botón enfocable ni tiene `aria-label`. El diseño pide un botón de 44 px con `aria-label`. Por eso se usa `InputText` + un `Button` de ícono.
- **Una sola fuente de valores.** Los hex, sombras, radios y espaciados están **solo** en `src/assets/main.css`, como variables CSS. El preset (`src/assets/presetClinica.js`) los referencia con `var(--…)` y no repite hex.
- **Fuentes desde Google Fonts (\*)**, con `<link>` en `index.html` y la misma URL de la maqueta. No agrega dependencias. Sin conexión, cae en `system-ui` y `ui-monospace`, que son los fallback de `DESIGN.md`.
- **Sin `primeicons`.** Los tres íconos del login (ojo, ojo tachado, error) son SVG inline de trazo 1.75, como en la maqueta.
- **Logo inline** en un componente, no en `public/`. Así sus colores usan las variables CSS; un `.svg` servido desde `public/` no puede leerlas.
- **Marca "Clínica Demo" en el login (\*).** Así lo dicen el diseño y `PRODUCT.md`. El título de la pestaña (`index.html`) y `MarcoAplicacion.vue` siguen diciendo "Consult-IA": quedan fuera de alcance.
- **Error de credenciales solo en línea, sin Toast (\*).** Se conserva el `<p role="alert">`: lo usa el test y sigue el patrón de error en línea de `EstadosVaciosCarga.dc.html`. Se quita el Toast del login para no anunciar el error dos veces a los lectores de pantalla. Esto cambia la RF12 y la CA33 de 001, que pedían Toast. El `<Toast />` global de `App.vue` no se toca.
- **Texto del error de credenciales (\*).** Debajo del mensaje del servicio se agrega "Revise el correo y la contraseña, o use una cuenta de demostración.". Es texto de la vista, no del servicio, y cumple la regla de error de `DESIGN.md`: qué pasó y qué hacer.
- **Fecha real con `utils/formato.js`.** `CLAUDE.md` pone el formato de fechas en `utils/formato.js`. Se crea con una sola función, `formatearFecha`. Este spec no configura el locale `es` de Day.js: el formato `DD/MM/YYYY` es numérico y no lo necesita. El locale queda para la fase 2, como decía 001. No se crea `useFecha`, porque no hay estado reactivo que compartir.
- **Cuentas de demostración como texto fijo en la vista (\*).** Las vistas no pueden importar `data/` (tabla de capas de `SETUP.md`). Las credenciales ya están cubiertas por los tests de 001: CA14 (usuarios de la semilla) y CA26 (login con esas cuentas).
- **Componentes extraídos:** solo los que el diseño repite en otras pantallas.
  - `HojaCopias.vue`: aparece en `MedicoLayout`, `MiAgendaMedico` y `AtenderCita`.
  - `MarcaClinica.vue`: aparece en todos los layouts.
  - `CampoError.vue`: aparece en todos los formularios y ya está nombrado en `SETUP.md`.
  - Además, así `LoginView.vue` queda por debajo de ~200 líneas.
  - La etiqueta y el renglón de los campos son estilos globales en `main.css` (clase `.campo`), no un componente.
- **Modo oscuro desactivado en el tema.** `DESIGN.md` prohíbe el modo oscuro por defecto. Sin esto, Aura pinta los campos oscuros si el sistema operativo está en modo oscuro.
- **Capas CSS.** `main.css` declara `@layer base, primevue;`. PrimeVue se registra con `cssLayer` en la capa `primevue`. Así:
  - el reset (capa `base`) no pisa a PrimeVue;
  - los ajustes de componentes de `main.css`, sin capa, sí ganan sobre PrimeVue, aunque PrimeVue inyecte sus estilos después.

### Criterio ante diferencias entre la maqueta y `DESIGN.md`
`SETUP.md` dice que `DESIGN.md` manda sobre las pantallas. Se aplica así:
- Los valores que `DESIGN.md` define de forma explícita (colores, sombras, radios, padding de hoja, desplazamiento de las copias, tipografía de inputs) salen de `DESIGN.md`.
- Las medidas de composición que `DESIGN.md` no fija salen de la maqueta: tamaños de títulos, columnas, gaps, alto del botón de 48 px.

Diferencias detectadas (\*):

| Punto | Maqueta `Main.dc.html` | Se aplica (`DESIGN.md`) |
|---|---|---|
| Copias apiladas | amarilla 6/6 px, rosa 12/12 px | amarilla 5/5 px, rosa 10/10 px |
| Sombra de la hoja | `0 1px 2px …08, 0 12px 32px …10` | sombra **Hoja** |
| Padding de la hoja | 36 px 40 px 32 px | 28 px 32 px (`hoja.padding`) |
| Texto de `::selection` | `#16192B` (`texto`) | `texto-copia-amarilla` ("texto encima siempre…") |
| Tamaño del texto de los inputs | 16 px | 15 px (`input.typography: body`) |
| Radio de las muestras de la leyenda | 2 px | `rounded.sm` (4 px): sin radios nuevos |

La maqueta no se modifica: estas diferencias quedan registradas aquí.

### Efectos sobre 001
Este spec deja sin efecto, **solo para los archivos que toca**, lo siguiente de 001:
- RNF7 y CA38: sin estilos ni CSS.
- CA2: la lista exacta de dependencias; se suma `@primeuix/themes`.
- RF12: el checkbox y el Toast.
- RF17: sin tema ni CSS.

001 no se edita.

`MarcoAplicacion`, los layouts y `EnConstruccionView` siguen siendo cascarones. Heredan de forma global la fuente, el fondo `mesa` y el tema de PrimeVue: es un efecto aceptado, no un rediseño.

## Requisitos
### Funcionales
- RF1. `package.json` agrega `@primeuix/themes` a `dependencies`, en una versión compatible con `primevue@5` (sin advertencias de *peer dependency* de npm).
- RF2. `index.html` carga las fuentes antes del script:
  - `<link rel="preconnect">` a `https://fonts.googleapis.com`;
  - `<link rel="preconnect" crossorigin>` a `https://fonts.gstatic.com`;
  - `<link rel="stylesheet">` a `https://fonts.googleapis.com/css2?family=Public+Sans:wght@400;500;600;700;800&family=Courier+Prime:wght@400;700&display=swap`.
  - El título de la pestaña no cambia.
- RF3. `src/assets/main.css`, en este orden:
  1. `@layer base, primevue;`
  2. `:root` con **todos** los tokens de `DESIGN.md` como variables CSS, con los nombres de la convención de RF10:
     - los 27 colores (`--color-tinta` … `--color-perforacion`);
     - `--radio-sm`, `--radio-md` y `--radio-pill`;
     - `--espacio-xs` … `--espacio-3xl`;
     - `--fuente-texto` (`'Public Sans', system-ui, sans-serif`) y `--fuente-dato` (`'Courier Prime', ui-monospace, monospace`);
     - `--sombra-hoja`, `--sombra-toast`, `--sombra-dialogo` y `--velo-dialogo` (de "Shadow Vocabulary");
     - `--sombra-boton` (`0 1px 2px rgba(21,46,128,.25)`, la "sombra corta" del botón primario de la maqueta).
  3. `@layer base { … }` con:
     - `box-sizing: border-box` para todos los elementos;
     - `body`:
       - `margin: 0`;
       - fondo `--color-mesa` y color `--color-texto`;
       - tipografía `body` (15px/1.55, 400, `--fuente-texto`);
       - `font-variant-numeric: tabular-nums`;
     - `::selection` con fondo `--color-copia-amarilla` y texto `--color-texto-copia-amarilla`;
     - `caret-color: var(--color-tinta)` en `input` y `textarea`;
     - `font: inherit` en `button`, `input`, `textarea` y `select`;
     - `:focus-visible` con `outline: 2px solid var(--color-tinta)` y `outline-offset: 2px`;
     - `a` en tinta con `text-underline-offset: 3px`, y en hover `tinta-oscura`.
  4. Clases globales, sin capa:
     - Tipografía: `.tipo-display`, `.tipo-headline`, `.tipo-title`, `.tipo-label` (con `text-transform: uppercase`) y `.tipo-dato`, con los valores de `typography` de `DESIGN.md`.
     - `.icono`: `fill: none; stroke: currentColor; stroke-width: 1.75`, con extremos y uniones redondeados.
     - `.campo`: columna con un gap de 8 px.
       - `.campo > label`: estilo Label en `--color-texto-secundario`.
       - Con foco dentro (`.campo:focus-within > label`): la etiqueta pasa a `--color-tinta`.
       - `.campo.campo-con-error > label`: la etiqueta pasa a `--color-alerta`.
  5. Ajustes de PrimeVue que el preset no puede expresar (sin capa):
     - `.p-inputtext`:
       - alto de 44 px, padding `0 12px`, 15 px;
       - fondo `--color-campo` y texto `--color-texto`;
       - sin bordes laterales ni superior;
       - renglón inferior de 1.5 px en `--color-texto`;
       - radio `--radio-sm --radio-sm 0 0`.
     - `.p-inputtext` con foco: renglón de 2 px en `--color-tinta`, más el anillo de foco global.
     - `.p-inputtext.p-invalid`: fondo `--color-alerta-suave` y renglón de 2 px en `--color-alerta`.
     - `.p-button:disabled`: fondo `--color-apagado`, texto `--color-lapiz`, borde de 1.5 px punteado en `--color-perforacion` y cursor `not-allowed`.
- RF4. `src/assets/presetClinica.js` exporta por defecto un preset hecho con `definePreset(Aura, …)` de `@primeuix/themes`. Todos sus valores de color son `var(--color-…)` de `main.css`; no tiene hex, `rgb` ni `rgba`. Mapea:
  - **Escala `primary`:**
    - 50 → `tinta-suave`;
    - 100 a 400 → `tinta-clara`;
    - 500 → `tinta`;
    - 600 a 950 → `tinta-oscura`.
  - **`colorScheme.light.primary`:** `color` tinta, `contrastColor` papel, `hoverColor` y `activeColor` tinta-oscura.
  - **Escala `surface` (claro):**
    - 0 → `papel`;
    - 50 → `lapiz-suave`;
    - 100 → `campo`;
    - 200 → `apagado`;
    - 300 → `renglon`;
    - 400 → `linea`;
    - 500 → `perforacion`;
    - 600 → `lapiz`;
    - 700 → `texto-secundario`;
    - 800 a 950 → `texto`.
  - **`focusRing`:** 2 px, `solid`, tinta, `offset` 2 px, sin sombra. Vale también para los campos de formulario.
  - **Campos de formulario:** fondo `campo`, texto `texto`, borde `texto` (también en hover), borde con foco `tinta`, borde inválido `alerta`, radio `var(--radio-sm)`, fondo deshabilitado `apagado` y texto deshabilitado `lapiz`.
  - **Botón primario:** fondo tinta, hover y activo `tinta-oscura`, texto papel, radio `var(--radio-md)`, peso 700.
- RF5. `src/main.js`:
  - importa `@/assets/main.css` antes de crear la app;
  - registra PrimeVue con:
    - `theme: { preset: presetClinica, options: { darkModeSelector: <valor que desactiva el modo oscuro en la versión instalada>, cssLayer: { name: 'primevue', order: 'base, primevue' } } }`.
  - El orden de 001 se mantiene: primero `inicializarDatos()` y después la app.
- RF6. `src/utils/formato.js` exporta `formatearFecha(fecha)`: recibe un `Date`, un string ISO o un objeto Day.js y devuelve `DD/MM/YYYY` con Day.js. Es una función pura: solo importa `dayjs`.
- RF7. Componentes nuevos en `src/components/comunes/`, con `<script setup>`, `<style scoped>` y solo variables CSS (sin hex):
  - **`MarcaClinica.vue`** (sin props):
    - el SVG del logo de la maqueta, 36×36, `aria-hidden="true"`;
    - los colores del SVG se aplican con CSS y variables: rosa y amarilla en las copias, papel en la hoja, `texto` en los contornos y `tinta` en los renglones;
    - al lado, el texto "Clínica Demo" en 18 px, peso 700 y `letter-spacing: -0.01em`.
  - **`HojaCopias.vue`** (sin props, con un `<slot>`):
    - un contenedor `position: relative`;
    - detrás, dos capas `aria-hidden="true"` con `--radio-sm`: la copia rosa desplazada 10/10 px y la amarilla desplazada 5/5 px;
    - delante, la hoja: fondo `--color-papel`, `--radio-sm`, `--sombra-hoja` y padding 28 px 32 px, con el slot adentro.
  - **`CampoError.vue`**:
    - props: `id` (String, obligatoria) y `mensaje` (String, por defecto `''`);
    - si `mensaje` está vacío, no renderiza nada;
    - si no, renderiza `<p :id>` con el ícono de error (círculo con "!", 14 px, `.icono`, `aria-hidden`) y el mensaje;
    - estilo: 13 px, peso 600, color `--color-alerta`, gap de 6 px.
- RF8. `src/views/LoginView.vue` reproduce la estructura de `Main.dc.html`, con las diferencias del Contexto. Dentro de un `<main>`:
  - **Columna de presentación**, en este orden:
    - `MarcaClinica`;
    - `<h1 class="tipo-display">` con "Una consulta,", un `<br>` y "tres copias.";
    - `<p>` de 18 px en `texto-secundario`, con un máximo de 44ch: "El médico dicta y la hoja se llena. Al confirmar, la historia clínica y la receta salen de la misma atención.";
    - una `<ul>` sin viñetas con tres `<li>`. Cada uno tiene una muestra de 26×32 px (`aria-hidden`, borde de 1.5 px en `texto`, `--radio-sm`) y un texto:
      - muestra `papel`: "**Original** · la consulta: motivo, signos vitales, diagnóstico y plan";
      - muestra `copia-amarilla`: "**Copia amarilla** · la historia clínica, en orden cronológico";
      - muestra `copia-rosa`: "**Copia rosa** · la receta en PDF, para descargar o imprimir".
      - La parte que sigue a "·" va en `texto-secundario`.
  - **Columna del formulario:** `HojaCopias`, que contiene el `<form novalidate @submit.prevent="enviar">` con este contenido, en orden:
    1. **Encabezado:**
       - `<h2>` "Ingresar", de 26 px, peso 800 y `letter-spacing: -0.02em`;
       - a la derecha, la fecha de hoy con `formatearFecha(new Date())` en `.tipo-dato` de 14 px y `texto-secundario`;
       - separados del resto por una perforación: `border-bottom: 2px dotted var(--color-perforacion)` y 18 px de padding inferior.
    2. **Campo de correo** (`.campo`):
       - `<label for="login-email">` "Correo electrónico";
       - `InputText` con `id="login-email"`, `type="email"`, `autocomplete="username"`, `v-model="formulario.email"` e `:invalid="!!errores.email"`;
       - `aria-describedby="login-email-error"` solo cuando hay error;
       - `CampoError` con `id="login-email-error"`;
       - el campo recibe la clase `campo-con-error` cuando hay error.
    3. **Campo de contraseña** (`.campo`):
       - `<label for="login-password">` "Contraseña";
       - un contenedor `position: relative` con:
         - `InputText` con `id="login-password"`, `:type="mostrarPassword ? 'text' : 'password'"`, `autocomplete="current-password"`, padding derecho de 48 px, `:invalid` y `aria-describedby="login-password-error"` cuando hay error;
         - un `Button` de 44×44 px, `type="button"`, variante texto, alineado arriba a la derecha y de color `texto-secundario`:
           - `aria-controls="login-password"`;
           - `aria-label` "Mostrar contraseña" cuando está oculta, u "Ocultar contraseña" cuando está visible;
           - un SVG de 20 px (`.icono`, `aria-hidden`): el ojo de la maqueta cuando está oculta, o el ojo con la diagonal `M3 3l18 18` cuando está visible;
           - al hacer clic alterna `mostrarPassword`;
       - `CampoError` con `id="login-password-error"`.
    4. **Error de credenciales**, si `authStore.error` tiene valor:
       - un `<p role="alert">` con el ícono de error de 18 px (`aria-hidden`), `authStore.error` y, debajo, "Revise el correo y la contraseña, o use una cuenta de demostración.";
       - estilo: color `--color-alerta`, peso 600 y gap de 8 px.
    5. **Botón `Button`** `type="submit"`:
       - texto "Ingresar", ancho completo, alto de 48 px, 16 px, peso 700, `--sombra-boton`, 4 px de margen superior;
       - `:disabled="authStore.cargando"`.
    6. **Recuadro de cuentas de demostración:**
       - fondo `--color-campo`, `--radio-md`, padding 14 px 16 px, gap de 6 px;
       - "Cuentas de demostración" en 13 px y peso 700;
       - en `.tipo-dato` de 14 px: "recepcion@clinica.com", un salto y "medico1@clinica.com · medico2 · medico3";
       - "Contraseña de todas: " en 13 px y `texto-secundario`, seguido de `123456` en `.tipo-dato`, color `texto` y peso 700.
    7. **Pie:** `<p>` de 12 px en `texto-secundario`: "Proyecto académico. Los datos se guardan solo en este navegador."
  - **Espaciado:** gap de 22 px entre los bloques del formulario, 32 px entre los bloques de la presentación y 14 px entre los ítems de la leyenda.
  - **Composición** (según el ancho de la ventana):
    - **1366 px o más:** grilla de dos columnas `minmax(0, 1fr) 452px`, gap de 104 px, padding 72 px 120 px, `align-items: center` y `min-height: 100vh`. La presentación tiene un ancho máximo de 560 px.
    - **De 1200 a 1365 px:** dos columnas. El padding lateral baja lo necesario, hasta un mínimo de 48 px, para que "Una consulta," no parta en dos renglones.
    - **Menos de 1200 px:** una columna centrada, con un ancho máximo de 560 px y padding 48 px 24 px. La presentación va arriba y la hoja abajo, con un ancho máximo de 452 px. El orden del DOM y el visual es el mismo.
- RF9. Comportamiento de `enviar()`:
  - Se mantiene igual que en 001: valida con `validarLogin` y, si es válido, llama a `authStore.iniciarSesion` y hace `router.push(INICIO_POR_ROL[rol])`.
  - Cambios:
    - ya no llama a `useToast` (se quita el import);
    - si la validación falla, mueve el foco al primer campo con error: el correo antes que la contraseña.
  - No se cambian `validarLogin`, sus mensajes ni el mensaje del servicio.
- RF10. `SETUP.md` documenta lo que agrega este spec (ver T1).

### No funcionales
- RNF1. JavaScript puro: ningún `.ts` ni `lang="ts"`. `<script setup>` y Composition API.
- RNF2. Sin colores literales fuera de `src/assets/main.css`: ningún `.vue` ni `presetClinica.js` contiene hex, `rgb(` ni `rgba(`.
- RNF3. Capas de `SETUP.md`: `LoginView` y los componentes nuevos no importan `@/services/*` ni `@/data/*`. `utils/formato.js` solo importa `dayjs`. `src/assets/*` solo se importa desde `main.js`.
- RNF4. Textos de interfaz y comentarios en español. Imports con `@/`.
- RNF5. Accesibilidad AA:
  - cada campo tiene un `<label for>` real;
  - foco visible en todos los controles (anillo de 2 px en tinta);
  - ningún estado depende solo del color: los errores llevan ícono y texto, y el botón deshabilitado lleva borde punteado;
  - solo se usan pares de color de `DESIGN.md`;
  - las áreas táctiles miden 44 px o más.
- RNF6. El rojo (`alerta*`) solo aparece en los errores.
- RNF7. `LoginView.vue` tiene menos de ~200 líneas. Cada componente nuevo tiene una sola responsabilidad.
- RNF8. Sin animaciones nuevas.

## Criterios de aceptación
**Herramientas y dependencias**
- [ ] CA1. `npm run lint` termina sin errores. `npm run test -- --run` pasa todos los tests. `npm run build` genera `dist/` sin errores.
- [ ] CA2. `dependencies` es la lista de 001 más `@primeuix/themes`. `npm ls @primeuix/themes primevue` no muestra errores. No hay `primeicons`, `typescript` ni `@fontsource/*`.
- [ ] CA3. `git diff --stat main` no muestra cambios en:
  - `src/services/`, `src/stores/`, `src/router/`, `src/data/`;
  - `src/utils/validaciones.js`, `src/utils/roles.js`;
  - `src/App.vue`, `src/layouts/`, `src/components/comunes/MarcoAplicacion.vue`;
  - `CLAUDE.md`, `DESIGN.md`, `docs/diseno/`.

**Restricciones (grep sobre `src/`)**
- [ ] CA4. `grep -rnE "#[0-9A-Fa-f]{3}\b|#[0-9A-Fa-f]{6}\b|rgba?\(" src --include=*.vue --include=*.js` no devuelve nada. Los colores literales solo están en `src/assets/main.css`.
- [ ] CA5. `grep -rn "@/services\|@/data" src/views src/components` no devuelve nada. `src/utils/formato.js` solo importa `dayjs`. `grep -rn "assets/" src` solo encuentra coincidencias en `src/main.js`.
- [ ] CA6. `find src tests -name "*.ts"` no devuelve nada, y todo `.vue` tiene `<script setup>`.
- [ ] CA7. `src/assets/main.css` define una variable `--color-*` por cada uno de los 27 colores del front matter de `DESIGN.md`, con el mismo valor hex, y además los radios, espaciados, fuentes y sombras de RF3.
- [ ] CA8. `index.html` contiene los dos `preconnect` y el `stylesheet` de RF2. El `<title>` sigue siendo "Consult-IA — Clínica".

**formato** (`tests/utils/formato.test.js`)
- [ ] CA9. `formatearFecha('2026-09-26')` devuelve `'26/09/2026'`. `formatearFecha(new Date(2026, 0, 5))` devuelve `'05/01/2026'`.

**LoginView** (`tests/views/LoginView.test.js`)
- [ ] CA10. Los tres casos existentes siguen pasando sin cambios en sus aserciones:
  - recepción → `/recepcion/dashboard`;
  - médico → `/medico/agenda`;
  - credenciales incorrectas → sigue en `/login`, con un `[role="alert"]` no vacío.
- [ ] CA11. Dado el formulario vacío, cuando se envía:
  - la ruta sigue en `/login`;
  - `#login-email` tiene `aria-invalid="true"` y `aria-describedby="login-email-error"`, y `#login-email-error` contiene "El correo es obligatorio.";
  - `#login-password` tiene `aria-invalid="true"` y `#login-password-error` contiene "La contraseña es obligatoria.";
  - `document.activeElement` es `#login-email`.
- [ ] CA12. Dado el login montado:
  - al inicio, `#login-password` tiene `type="password"` y el botón `[aria-controls="login-password"]` tiene `aria-label="Mostrar contraseña"`;
  - al hacer clic en ese botón, el campo pasa a `type="text"` y el `aria-label` a "Ocultar contraseña";
  - un segundo clic vuelve al estado inicial.
- [ ] CA13. Dado un envío válido o inválido, `#login-email` y `#login-password` no tienen `aria-invalid` cuando no hay error de su campo.

**Interfaz** (inspección manual con `npm run dev`, a 1366×768, al lado de `docs/diseno/pantallas/Main.dc.html` abierto en el navegador)
- [ ] CA14. **Estructura y textos:**
  - la estructura y los textos coinciden con la maqueta: marca "Clínica Demo" con logo, titular en dos renglones, bajada, leyenda de tres copias, hoja con copias apiladas, encabezado "Ingresar" con fecha y perforación punteada, dos campos, botón "Ingresar", recuadro de cuentas y pie;
  - las únicas diferencias son las de la tabla del Contexto;
  - no hay scroll vertical ni horizontal.
- [ ] CA15. **Fecha e inputs:**
  - la fecha del encabezado es la de hoy en `DD/MM/YYYY`, en Courier Prime;
  - los inputs empiezan vacíos;
  - en DevTools › Computed, los textos usan "Public Sans" y la fecha y las cuentas usan "Courier Prime".
- [ ] CA16. **Foco:**
  - al navegar con Tab, el orden es: correo → contraseña → botón del ojo → "Ingresar";
  - cada control muestra un anillo de foco de 2 px en tinta separado 2 px;
  - en los campos, el renglón pasa a 2 px en tinta y la etiqueta a tinta.
- [ ] CA17. **Errores de validación:** dado el formulario vacío, cuando se presiona "Ingresar", entonces los dos campos muestran:
  - fondo `alerta-suave`, renglón de 2 px en `alerta` y etiqueta en `alerta`;
  - debajo, el ícono y el mensaje de `validarLogin`;
  - y el foco queda en el correo.
- [ ] CA18. **Error de credenciales:** dado `recepcion@clinica.com` con una contraseña incorrecta, cuando se envía, entonces:
  - aparece en línea, con ícono, "Correo o contraseña incorrectos." y "Revise el correo y la contraseña, o use una cuenta de demostración.";
  - no aparece ningún Toast.
- [ ] CA19. **Contraseña visible:** el botón del ojo alterna entre la contraseña visible y la oculta, y cambia de ícono (ojo / ojo tachado).
- [ ] CA20. **Botón deshabilitado:** en DevTools, al agregar `disabled` al botón "Ingresar", se ve con fondo `apagado`, texto `lapiz` y borde punteado.
- [ ] CA21. **Redirección por rol:** con `recepcion@clinica.com` / `123456` se llega a `/recepcion/dashboard`, y con `medico1@clinica.com` / `123456` a `/medico/agenda`.
- [ ] CA22. **Modo oscuro:** con el sistema operativo en modo oscuro (o `prefers-color-scheme: dark` emulado en DevTools), el login se ve igual que en modo claro.
- [ ] CA23. **Responsive:**
  - a 1280×800 hay dos columnas y "Una consulta," ocupa un solo renglón;
  - a 1024×768 y 768×1024 hay una columna, con la presentación arriba y la hoja abajo;
  - en ninguno de los tres tamaños hay scroll horizontal.
- [ ] CA24. **Selección y caret:** al seleccionar texto, el fondo es `copia-amarilla` y el texto `texto-copia-amarilla`. El caret de los inputs es tinta.
- [ ] CA25. **Rojo reservado:** el rojo no aparece en el login fuera de los estados de error de CA17 y CA18.

**Documentación**
- [ ] CA26. `SETUP.md` refleja todo lo que pide T1.

## Archivos afectados
| Acción | Ruta | Propósito |
|---|---|---|
| modificar | SETUP.md | Árbol (`assets/`, componentes comunes), capa `assets/`, dependencia `@primeuix/themes`, fuentes, instalación, convención de variables CSS y estado actual |
| modificar | package.json | Agregar `@primeuix/themes` |
| modificar | package-lock.json | Lo actualiza `npm install` |
| modificar | index.html | `<link>` a Google Fonts (Public Sans y Courier Prime) |
| crear | src/assets/main.css | Tokens de `DESIGN.md`, base, clases globales y ajustes de PrimeVue |
| crear | src/assets/presetClinica.js | Preset de `@primeuix/themes` (Aura) que referencia las variables de `main.css` |
| modificar | src/main.js | Importar `main.css` y registrar el tema (modo oscuro desactivado, `cssLayer`) |
| crear | src/utils/formato.js | `formatearFecha` (`DD/MM/YYYY`) |
| crear | tests/utils/formato.test.js | CA9 |
| crear | src/components/comunes/MarcaClinica.vue | Logo de tres copias y "Clínica Demo" |
| crear | src/components/comunes/HojaCopias.vue | Hoja con copias amarilla y rosa apiladas (slot) |
| crear | src/components/comunes/CampoError.vue | Mensaje de error de campo con ícono e `id` |
| modificar | src/views/LoginView.vue | Diseño de `Main.dc.html`, botón del ojo, errores accesibles, sin Toast |
| modificar | tests/views/LoginView.test.js | CA10–CA13 |

## Tareas
- [x] T1. Actualizar `SETUP.md`:
  - **Árbol:**
    - `src/assets/` pasa a describirse como "Estilos globales y tema: `main.css` (tokens de `DESIGN.md`, base) y `presetClinica.js` (preset de PrimeVue)";
    - agregar `utils/formato.js` con el comentario "`formatearFecha`";
    - en `components/comunes/`, listar `MarcaClinica`, `HojaCopias` y `CampoError`.
  - **Tabla de capas:** nueva fila `assets/`: no importa nada del proyecto y solo lo importa `main.js`.
  - **Dependencias (Runtime):**
    - nueva fila `@primeuix/themes`: "Preset del tema de PrimeVue (base Aura) con los tokens de `DESIGN.md`";
    - la fila `primevue` pasa a decir "tematizado con `src/assets/presetClinica.js`".
  - **"Sin dependencia":** agregar "Fuentes Public Sans y Courier Prime desde Google Fonts (`<link>` en `index.html`)".
  - **Instalación por fase:** agregar el bloque `# Fase 1 — diseño del login (spec 002)` con `npm install @primeuix/themes`, y reemplazar la línea "Si se incorpora un tema o íconos de PrimeVue…" por "Íconos: SVG inline de trazo 1.75; no se usa `primeicons`."
  - **Convenciones de nombres:**
    - fila "Variable CSS de diseño": `--` + grupo en español + nombre del token de `DESIGN.md` (`--color-tinta`, `--radio-md`, `--espacio-lg`, `--sombra-hoja`, `--fuente-dato`);
    - fila "Clase CSS global": kebab-case en español (`.campo`, `.campo-con-error`, `.tipo-label`, `.icono`).
  - **Diseño › Cómo se implementa:** los hex viven solo en `main.css`, el preset los referencia con `var()`, y se describen las capas CSS (`base`, `primevue`).
  - **Estado actual:** fase 1 implementada, con el login diseñado (spec 002); los layouts siguen sin diseño.
- [x] T2. Instalar `@primeuix/themes` con `npm install @primeuix/themes` (RF1). Si npm reporta un conflicto de *peer* con `primevue@5`, detenerse y reportarlo; no forzar con `--force` ni `--legacy-peer-deps`.
- [x] T3. Agregar los `<link>` de fuentes a `index.html` (RF2).
- [x] T4. Crear `src/assets/main.css` con `@layer`, las variables de tokens y la capa `base` (RF3, puntos 1 a 3).
- [x] T5. Agregar a `main.css` las clases globales (`.tipo-*`, `.icono`, `.campo`) y los ajustes de `.p-inputtext` y `.p-button:disabled` (RF3, puntos 4 y 5).
- [x] T6. Crear `src/assets/presetClinica.js` (RF4). Si alguna variable del preset no funciona con `var(--…)` en la versión instalada, detenerse y reportarlo; no copiar hex al preset.
- [x] T7. Modificar `src/main.js`: importar `main.css` y registrar el tema con el modo oscuro desactivado y `cssLayer` (RF5).
- [x] T8. Crear `src/utils/formato.js` con `formatearFecha` (RF6).
- [x] T9. Tests: `tests/utils/formato.test.js` (CA9).
- [x] T10. Crear `src/components/comunes/CampoError.vue` (RF7).
- [x] T11. Crear `src/components/comunes/HojaCopias.vue` (RF7).
- [x] T12. Crear `src/components/comunes/MarcaClinica.vue` (RF7).
- [x] T13. Reescribir la plantilla y los estilos de `src/views/LoginView.vue` (RF8), y ajustar `enviar()`: quitar el Toast y mover el foco al primer campo con error (RF9).
- [x] T14. Tests: actualizar `tests/views/LoginView.test.js`. Se pueden separar el montaje y el envío en helpers. Se conservan las tres aserciones existentes (CA10) y se agregan los casos CA11 a CA13.
- [x] T15. Verificar:
  - `npm run lint`, `npm run test -- --run` y `npm run build`;
  - `npm ls @primeuix/themes primevue`;
  - los greps de CA3 a CA8;
  - la inspección manual de CA14 a CA25 contra `docs/diseno/pantallas/Main.dc.html`, que se usa como referencia y **no se modifica**.

## Fuera de alcance
- Diseño de `MarcoAplicacion`, `RecepcionLayout`, `MedicoLayout` y `EnConstruccionView`: menú lateral, cabecera con perforación y variante de `MarcaClinica` de 30 px con subtítulo de rol. Se hacen en specs siguientes con los tokens de este spec.
- Dashboard y demás pantallas.
- Tema específico de `Toast`, `ConfirmDialog`, `Dialog`, `DataTable` y otros componentes: se completa en el preset cuando un spec los use. Hoy toman los valores generales del preset.
- Título de la pestaña, favicon y la marca "Consult-IA" fuera del login.
- Cambios en `authService`, `stores/auth`, el router, el guard, `validarLogin` y sus mensajes.
- Borrar los errores mientras el usuario escribe: se mantiene el comportamiento de 001, en el que los errores se recalculan al enviar.
- Estado de carga visible ("Ingresando…"): el login es síncrono y `cargando` nunca se ve.
- Locale `es` de Day.js y otras funciones de `formato.js` (fase 2). Composable `useFecha`.
- Modo oscuro, animaciones, tests visuales o de captura.
- Modificar `docs/diseno/pantallas/Main.dc.html` o `DESIGN.md`, incluido su comentario "Aún no hay tema de PrimeVue".
- Traducción del `locale` de PrimeVue (fase 2, según 001).

## Decisiones abiertas
- **Fuentes.** Recomendación aplicada: Google Fonts con `<link>` (sin dependencias, igual que la maqueta). Alternativa: `@fontsource/public-sans` y `@fontsource/courier-prime`, que funcionan sin internet en la presentación, pero agregan 2 dependencias. En ese caso cambian RF2, CA2, CA8 y T1.
- **PrimeVue tematizado o nativo.** Recomendación aplicada: `InputText` + `Button` de PrimeVue con el preset, como pide `SETUP.md`. Alternativa: `<input>` y `<button>` nativos con los tokens, sin `@primeuix/themes`. Es menos código ahora, pero contradice `SETUP.md` › Diseño y habría que tematizar PrimeVue igual en la fase 2.
- **Marca.** Recomendación aplicada: "Clínica Demo" en el login (diseño y `PRODUCT.md`). La pestaña y `MarcoAplicacion` siguen con "Consult-IA" hasta el spec de layouts. ¿Se unifica todo en "Clínica Demo"? ¿"Consult-IA" queda como nombre del proyecto (README, repo)?
- **Diferencias entre la maqueta y `DESIGN.md`** (tabla del Contexto). Recomendación aplicada: manda `DESIGN.md`, como dice `SETUP.md`. Punto a confirmar: el texto de los inputs a 15 px. En Safari de iPad, los inputs de menos de 16 px hacen zoom al enfocarlos. Si la tablet es un objetivo real, conviene fijar 16 px (el valor de la maqueta) y actualizar `DESIGN.md` (`input.typography`) en este spec.
- **Toast del login.** Recomendación aplicada: quitarlo y dejar solo el error en línea (`role="alert"`), para no anunciarlo dos veces. Alternativa: mantener los dos, como en 001; en ese caso hay que tematizar `Toast` en el preset (fondo papel, borde de 1 px en alerta, ícono sobre `alerta-suave`, `--sombra-toast`, como en `Componentes.dc.html`).
- **Texto de ayuda del error de credenciales:** "Revise el correo y la contraseña, o use una cuenta de demostración." Es un texto nuevo que no está en el diseño; confirmar o cambiar la redacción.
- **Ícono de "Ocultar contraseña"** (ojo tachado). La maqueta solo muestra el ojo abierto. Se propone el mismo ojo con una diagonal, con el mismo trazo.
- **Fecha visible.** Se muestra la fecha de hoy, calculada al montar la vista. No se actualiza si la página queda abierta después de la medianoche.
- **Cuentas de demostración.** Recomendación aplicada: texto fijo en la vista (las vistas no pueden importar `data/`). Si se prefiere una sola fuente, habría que exponerlas desde un servicio o store, lo que agrega código solo para esto.
- **Responsive.** `DESIGN.md` lo deja "a resolver en implementación". Se proponen los cortes de RF8: dos columnas desde 1200 px y una columna por debajo. En tablet vertical, la hoja queda debajo de la presentación y puede requerir scroll. Alternativa: mostrar primero la hoja por debajo de 1200 px, con orden visual invertido.
