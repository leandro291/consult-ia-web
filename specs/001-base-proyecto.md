# 001 — Base del proyecto: Vite + Vue 3, login, roles y datos semilla

Estado: implementado
Fase: 1
Depende de: 000

## Contexto
El repositorio hoy solo tiene documentación (`CLAUDE.md`, `SETUP.md`, `specs/000`, `.claude/agents/`, `.gitignore`). No existe `package.json` ni `src/`. Esta fase deja la base **lógica y estructural** sobre la que se construyen las fases 2 a 7:
- Proyecto Vite + Vue 3 en JavaScript, con ESLint y Vitest configurados según `SETUP.md`.
- Vue Router con **todas** las rutas de `CLAUDE.md` declaradas y un guard global de sesión y rol.
- Pinia y PrimeVue **sin tema**: solo `ToastService`, `ConfirmationService`, `Toast` y `ConfirmDialog` globales, como mecanismo de feedback.
- Capa de persistencia (`storage.js`), login simulado (`authService.js` + store `auth`) y layouts por rol.
- Datos semilla completos, cargados solo si `clinica_version` no existe.

**Sin diseño visual.** El usuario aporta su propio diseño por separado. Esta fase no incluye tema, íconos, CSS, responsive ni decisiones estéticas. `LoginView`, `EnConstruccionView`, `MarcoAplicacion` y los layouts son **cascarones funcionales**: HTML semántico plano, sin bloque `<style>` ni clases de estilo. Después se van a reemplazar o estilizar con el diseño del usuario **sin tocar la lógica**: las props (`titulo`, `items`), los stores, `validarLogin`, el router y el guard se mantienen.

Decisiones tomadas en este spec (desvíos de `SETUP.md` que se documentan en T1):
- **Dependencias:** la fase 1 instala solo lo que usa: `vue`, `vue-router`, `pinia`, `primevue` y `dayjs`. `dayjs` se usa en la semilla para calcular las fechas de la semana. No se instalan `@primeuix/themes` ni `primeicons`, porque no hay tema ni íconos. FullCalendar se instala en la fase 3 y pdfmake en la fase 5, cada una en su spec. `SETUP.md` pasa a mostrar el comando de instalación por fase.
- **Sin `npm create vite`:** el comando de `SETUP.md` corre sobre un directorio no vacío y puede pisar o borrar `CLAUDE.md` y `.gitignore`. Además, todo lo que genera (demo `HelloWorld`, `style.css`, `vue.svg`) se descarta. Los pocos archivos base (`package.json`, `index.html`, `vite.config.js`) se escriben a mano y las dependencias se instalan con `npm install`.
- **`leer(clave, porDefecto = [])`:** se agrega un segundo parámetro opcional al patrón de `CLAUDE.md`. Con un solo argumento el comportamiento es idéntico. Hace falta porque `clinica_version` (número) y `clinica_sesion` (objeto o `null`) no son arrays, y con `?? []` no se puede distinguir "no existe" de "vacío".
- **Semilla en dos archivos:** `data/seed.js` genera los datos (sin imports del proyecto, como exige la tabla de capas) y el nuevo `services/semillaService.js` los persiste vía `storage.js`. `main.js` llama a `inicializarDatos()`. `cargarSemilla()` queda exportada para que la reutilice "Restablecer datos de demo" en la fase 6.
- **Guard testeable:** la decisión del guard vive en una función pura `resolverAcceso()` en `src/router/acceso.js`, así se testea sin montar el router.
- **Vistas de fases futuras:** todas las rutas aún no implementadas usan una única vista `EnConstruccionView.vue`. Cada fase posterior reemplaza el `component` de su ruta.
- **Layouts sin duplicación:** `RecepcionLayout` y `MedicoLayout` solo definen sus enlaces de navegación. Los enlaces, el nombre del usuario, el botón "Cerrar sesión" y el `<RouterView>` van en `components/comunes/MarcoAplicacion.vue`.

Limitación (se documenta en el README en la fase 6): la autenticación es simulada y las contraseñas se guardan en texto plano en `clinica_usuarios`.

## Requisitos
### Funcionales
- RF1. `package.json` con `"private": true`, `"type": "module"` y los scripts `dev`, `build`, `preview`, `lint` y `test` exactamente como en la tabla de `SETUP.md`.
- RF2. `vite.config.js` y `eslint.config.js` con la configuración mínima de `SETUP.md`: alias `@` → `src`, `test.environment: 'jsdom'`, flat config con `js.configs.recommended`, `pluginVue.configs['flat/recommended']` y los globals de browser y node.
- RF3. `index.html` con `lang="es"`, título "Consult-IA — Clínica" y `<div id="app">`. Carga `/src/main.js`.
- RF4. `src/services/storage.js` exporta:
  - `leer(clave, porDefecto = [])`: devuelve el JSON parseado, o `porDefecto` si la clave no existe, vale `null` o el JSON está corrupto.
  - `guardar(clave, datos)`.
  - `CLAVES`: objeto con las claves `clinica_*` del modelo de datos (`usuarios`, `pacientes`, `medicos`, `consultorios`, `citas`, `consultas`, `recetas`, `sesion`, `config`, `version`).
- RF5. `src/data/seed.js` exporta:
  - `VERSION_SEMILLA = 1`.
  - `TRANSCRIPCION_DEMO`: el texto de demo de `CLAUDE.md`, literal.
  - `generarDatosSemilla(hoy = dayjs())`: devuelve `{ consultorios, medicos, usuarios, pacientes, citas, consultas, recetas }` con la forma exacta del modelo de datos de `CLAUDE.md`. Todos los registros tienen `id` (`crypto.randomUUID()`) y `creadoEn` (ISO). Contenido:
    - 3 consultorios.
    - 3 médicos: Medicina General (`dias: [1,2,3,4,5,6]`), Pediatría y Cardiología, cada uno en un consultorio distinto y con horario propio.
    - 4 usuarios: `recepcion@clinica.com` (rol `recepcion`) y `medico1@clinica.com`, `medico2@clinica.com`, `medico3@clinica.com` (rol `medico`, con `medicoId`), todos con la contraseña `123456`.
    - 10 pacientes con DNI de 8 dígitos únicos. Al menos uno tiene `alergias` que incluye `'Penicilina'`.
    - Entre 12 y 18 citas en la ventana de `hoy − 3` a `hoy + 3` días, respetando el horario de cada médico y sin solapamientos. Estados: las citas pasadas son `atendida`, `no_asistio` o `cancelada`; las de hoy y futuras son `programada`, `confirmada` o `cancelada`. `duracionMin: 30`. `consultorioId` igual al del médico.
    - Una consulta por cada cita `atendida` (con signos vitales, diagnósticos con código CIE-10 y plan) y al menos una receta ligada a una de esas consultas.
- RF6. `src/services/semillaService.js` exporta:
  - `cargarSemilla()`: escribe todas las colecciones de `generarDatosSemilla()` y `clinica_version = VERSION_SEMILLA`, pisando lo que haya. No toca `clinica_sesion` ni `clinica_config`.
  - `inicializarDatos()`: llama a `cargarSemilla()` solo si `clinica_version` no existe.
- RF7. `src/services/authService.js` exporta:
  - `login(email, password)`: compara el email sin distinguir mayúsculas y sin espacios al borde, y la contraseña de forma exacta. Si coincide, guarda y devuelve (copia) `{ usuarioId, rol, nombre }` en `clinica_sesion`. Si no, lanza `Error('Correo o contraseña incorrectos.')` sin guardar nada.
  - `logout()`: deja `clinica_sesion` en `null`.
  - `sesionActual()`: devuelve una copia de la sesión, o `null`.
  - Un comentario breve indica que la autenticación es simulada, con contraseñas en texto plano.
- RF8. `src/stores/auth.js` (`useAuthStore`, setup store) expone:
  - `sesion`, inicializada con `sesionActual()`;
  - `cargando` y `error`;
  - `iniciarSesion(email, password)`: devuelve `true` o `false`; si falla, `error` queda con el mensaje del servicio;
  - `cerrarSesion()`.
  - Solo importa `authService.js`.
- RF9. `src/utils/validaciones.js` exporta `validarLogin({ email, password })`. Devuelve un objeto con un mensaje por campo inválido, o `{}` si todo es válido:
  - email vacío → `'El correo es obligatorio.'`;
  - email con formato inválido → `'Ingrese un correo válido.'`;
  - contraseña vacía → `'La contraseña es obligatoria.'`.
- RF10. `src/router/acceso.js` exporta:
  - `INICIO_POR_ROL = { recepcion: '/recepcion/dashboard', medico: '/medico/agenda' }`;
  - `resolverAcceso(destino, sesion)`: devuelve `true` o la ruta a la que redirigir. Una sesión es válida si existe y su `rol` está en `INICIO_POR_ROL`. Casos:
    - destino público (`meta.publica`) + sesión válida → inicio de su rol;
    - destino público sin sesión válida → `true`;
    - destino privado sin sesión válida → `'/login'`;
    - `meta.rol` distinto del rol de la sesión → inicio de su rol;
    - en cualquier otro caso → `true`.
- RF11. `src/router/index.js` (`createWebHistory`):
  - `/login` → `LoginView`, con `meta: { publica: true }`.
  - `/` redirige a `/login`. El guard lleva al inicio del rol si hay sesión.
  - `/recepcion` → `RecepcionLayout` con `meta: { rol: 'recepcion' }` y redirección a `/recepcion/dashboard`. Hijas: `dashboard`, `pacientes`, `pacientes/:id`, `medicos`, `consultorios`, `agenda`, `respaldo`.
  - `/medico` → `MedicoLayout` con `meta: { rol: 'medico' }` y redirección a `/medico/agenda`. Hijas: `agenda`, `atencion/:citaId`, `historia/:pacienteId`, `receta/:consultaId`, `configuracion`.
  - Las hijas heredan `meta.rol` por el merge de `to.meta` de Vue Router y usan `EnConstruccionView`.
  - Cualquier otra ruta redirige a `/`.
  - `beforeEach` aplica `resolverAcceso(to, useAuthStore().sesion)`.
- RF12. `LoginView.vue` (cascarón funcional, HTML plano):
  - `<form>` con `<input type="email">` para el correo y `<input type="password">` para la contraseña, cada uno con su `<label for>` asociado.
  - Un `<input type="checkbox">` "Mostrar contraseña" alterna el `type` del campo de contraseña entre `password` y `text`.
  - `<button type="submit">` "Ingresar", deshabilitado mientras `authStore.cargando` es `true`. Enter en cualquier campo envía el formulario (comportamiento nativo del `submit`).
  - Valida con `validarLogin` y muestra cada error en un elemento de texto debajo de su campo, sin llamar al store.
  - Si el login falla, muestra un Toast de error (`useToast()` de PrimeVue) con `authStore.error`. Si funciona, navega a `/` y el guard lleva al inicio del rol.
- RF13. `MarcoAplicacion.vue` (cascarón funcional, HTML plano) recibe `titulo` e `items` (`[{ etiqueta, ruta }]`) y muestra:
  - un `<header>` con "Consult-IA", el `titulo`, el `sesion.nombre` y un `<button>` "Cerrar sesión" (llama a `cerrarSesion()` y navega a `/login`);
  - un `<nav>` con un `RouterLink` por ítem;
  - un `<main>` con el `<RouterView>`.
- RF14. Los layouts usan `MarcoAplicacion` con estos enlaces:
  - `RecepcionLayout`: Dashboard, Pacientes, Médicos, Consultorios, Agenda y Respaldo.
  - `MedicoLayout`: Mi agenda y Configuración IA. Las rutas con parámetro no van a la navegación.
- RF15. `EnConstruccionView.vue` muestra "Sección en construcción" y "Esta pantalla se implementa en una fase posterior."
- RF16. `App.vue` solo contiene `<Toast />`, `<ConfirmDialog />` y `<RouterView />`.
- RF17. `main.js`, en este orden:
  1. `inicializarDatos()`;
  2. `createApp(App)` con Pinia, el Router, PrimeVue sin opción `theme` (sin preset), `ToastService` y `ConfirmationService`;
  3. `mount('#app')`.
  - No importa ningún archivo CSS.
- RF18. `SETUP.md` refleja lo que agrega o quita este spec (ver T1 y T23).

### No funcionales
- RNF1. JavaScript puro: ningún `.ts` ni `lang="ts"`. Todos los `.vue` usan `<script setup>` y Composition API.
- RNF2. `localStorage` solo aparece en `src/services/storage.js`.
- RNF3. Capas según `SETUP.md`:
  - views, layouts y components no importan `@/services/*`;
  - `stores/auth.js` solo importa `authService.js`;
  - `data/` y `utils/` no importan nada del proyecto;
  - `router/` no importa services.
- RNF4. Imports internos con el alias `@/`.
- RNF5. Textos de interfaz, mensajes de error, identificadores y comentarios en español.
- RNF6. Componentes de menos de ~200 líneas y con una sola responsabilidad.
- RNF7. Sin diseño visual: ningún `.vue` tiene bloque `<style>`, no hay archivos `.css` en `src/` y ningún archivo importa CSS. La lógica de los cascarones (props, eventos, llamadas a stores) no depende del marcado, para que el usuario pueda reemplazar el HTML con su diseño.
- RNF8. Los tests importan `describe`, `it` y `expect` desde `vitest` (sin `globals: true`) y limpian `localStorage` en `beforeEach`.

## Criterios de aceptación
**Herramientas y proyecto**
- [ ] CA1. Los scripts de `package.json` coinciden con la tabla de `SETUP.md`.
- [ ] CA2. `dependencies` contiene exactamente `vue`, `vue-router`, `pinia`, `primevue` y `dayjs`. `devDependencies` contiene exactamente `vite`, `@vitejs/plugin-vue`, `eslint`, `@eslint/js`, `eslint-plugin-vue`, `globals`, `vitest` y `jsdom`. No hay `typescript`, `@primeuix/themes`, `primeicons`, `@fullcalendar/*` ni `pdfmake`.
- [ ] CA3. `npm run lint` termina sin errores.
- [ ] CA4. `npm run test -- --run` pasa todos los tests.
- [ ] CA5. `npm run build` genera `dist/` sin errores.
- [ ] CA6. `git diff --stat` no muestra cambios en `CLAUDE.md`, `.gitignore`, `specs/000-base-documental-sdd.md` ni `.claude/`.

**Restricciones (verificables con grep o find sobre `src/`)**
- [ ] CA7. `find src tests -name "*.ts"` y `grep -rn 'lang="ts"' src` no devuelven nada.
- [ ] CA8. Todo `.vue` de `src/` contiene `<script setup>`.
- [ ] CA9. `grep -rn "localStorage" src` solo encuentra coincidencias en `src/services/storage.js`.
- [ ] CA10. `grep -rn "@/services" src/views src/layouts src/components src/router` no devuelve nada. `src/stores/auth.js` no importa otro servicio que `authService.js`. `src/data/seed.js` y `src/utils/validaciones.js` no tienen imports con `@/` ni rutas relativas.

**storage.js** (`tests/services/storage.test.js`)
- [ ] CA11. Dada una clave inexistente, cuando se llama `leer(clave)`, entonces devuelve `[]`; y con `leer(clave, null)` devuelve `null`.
- [ ] CA12. Dado un valor no JSON en la clave, cuando se llama `leer(clave)`, entonces devuelve `[]` sin lanzar.
- [ ] CA13. Dado `guardar(clave, datos)`, cuando se llama `leer(clave)`, entonces devuelve un valor igual (`toEqual`) a `datos`. Dado `guardar(clave, null)`, `leer(clave, null)` devuelve `null`.

**Datos semilla** (`tests/data/seed.test.js`, con `hoy` fijo en un lunes, un miércoles, un sábado y un domingo)
- [ ] CA14. Hay 3 consultorios, 3 médicos con especialidades {Medicina General, Pediatría, Cardiología}, 4 usuarios (1 `recepcion` y 3 `medico` cuyo `medicoId` existe) y 10 pacientes.
- [ ] CA15. Todo registro de cada colección tiene un `id` con formato UUID, único en su colección, y un `creadoEn` que `Date.parse` interpreta.
- [ ] CA16. Los DNI tienen 8 dígitos (`/^\d{8}$/`) y son únicos. Existe al menos un paciente con `'Penicilina'` en `alergias`, y ninguna receta de ese paciente contiene amoxicilina, ampicilina ni penicilina.
- [ ] CA17. Hay entre 12 y 18 citas, todas con `fecha` entre `hoy − 3` y `hoy + 3` (`YYYY-MM-DD`), `hora` en formato `HH:mm` y `duracionMin` igual a 30. `pacienteId` y `medicoId` existen, y `consultorioId` es el del médico.
- [ ] CA18. Cada cita cae en un día incluido en `horario.dias` del médico, con `hora >= inicio` y `hora + 30 min <= fin`. Ningún médico tiene dos citas no canceladas que se solapen en la misma fecha.
- [ ] CA19. Las citas con fecha anterior a `hoy` tienen estado `atendida`, `no_asistio` o `cancelada`. Las de `hoy` o posteriores tienen `programada`, `confirmada` o `cancelada`. Entre todas aparecen al menos 4 estados distintos. Salvo que `hoy` sea domingo, hay al menos una cita con `fecha === hoy`.
- [ ] CA20. Cada cita `atendida` tiene exactamente una consulta, y cada consulta apunta a una cita `atendida` con el mismo `pacienteId`, `medicoId` y `fecha`. Hay al menos 2 consultas.
- [ ] CA21. Hay al menos 1 receta. Cada receta tiene un `consultaId` existente, con el mismo `pacienteId` y `medicoId` que la consulta, e `items` con los campos `medicamento`, `dosis`, `frecuencia`, `duracion`, `via` e `indicaciones`.
- [ ] CA22. `TRANSCRIPCION_DEMO` es idéntico al texto de demo de `CLAUDE.md`.

**semillaService** (`tests/services/semillaService.test.js`)
- [ ] CA23. Dado un `localStorage` vacío, cuando se llama `inicializarDatos()`, entonces existen las 7 colecciones y `leer(CLAVES.version, null) === 1`.
- [ ] CA24. Dado que `clinica_version` ya existe y `clinica_pacientes` fue modificado, cuando se llama `inicializarDatos()`, entonces ninguna clave cambia (idempotente, no pisa datos).
- [ ] CA25. Dada una sesión guardada, cuando se llama `cargarSemilla()`, entonces `clinica_sesion` no cambia.

**authService** (`tests/services/authService.test.js`, con la semilla cargada)
- [ ] CA26. Dado `login('recepcion@clinica.com', '123456')`, entonces devuelve `{ usuarioId, rol: 'recepcion', nombre }` y `sesionActual()` devuelve lo mismo. Con `'  Medico1@Clinica.com '` también funciona, con rol `medico`.
- [ ] CA27. Dada una contraseña incorrecta o un email inexistente, cuando se llama `login`, entonces lanza `Error` con el mensaje `'Correo o contraseña incorrectos.'` y `sesionActual()` sigue siendo `null`.
- [ ] CA28. Dado un login previo, cuando se llama `logout()`, entonces `sesionActual()` devuelve `null`.
- [ ] CA29. Mutar el objeto que devuelve `sesionActual()` no altera la siguiente llamada (devuelve copias).

**validaciones** (`tests/utils/validaciones.test.js`)
- [ ] CA30. `validarLogin({ email: '', password: '' })` devuelve los dos mensajes de RF9. Un email `'abc'` devuelve `'Ingrese un correo válido.'`. Datos válidos devuelven `{}`.

**Guard** (`tests/router/acceso.test.js`)
- [ ] CA31. `resolverAcceso` devuelve:
  - `'/login'` para un destino privado sin sesión;
  - `'/login'` con una sesión de rol desconocido;
  - `true` para `/login` sin sesión;
  - `INICIO_POR_ROL.medico` para `/login` con sesión de médico;
  - `INICIO_POR_ROL.recepcion` para un destino `meta.rol: 'medico'` con sesión de recepción;
  - `true` para un destino con el rol correcto.

**Interfaz funcional** (inspección manual con `npm run dev`; se evalúa el comportamiento, no la apariencia)
- [ ] CA32. Sin sesión, al abrir `/`, `/recepcion/pacientes` o `/medico/agenda`, se redirige a `/login`.
- [ ] CA33. Dado el formulario vacío, cuando se presiona "Ingresar" (o Enter), entonces aparecen los errores debajo de cada campo. Con credenciales incorrectas aparece el texto de un Toast de error en español (sin estilos es aceptable). Al marcar "Mostrar contraseña", la contraseña se ve en texto plano.
- [ ] CA34. Con `recepcion@clinica.com` / `123456` se llega a `/recepcion/dashboard`, con 6 enlaces de navegación y el nombre del usuario visible. Al abrir `/medico/agenda`, se redirige a `/recepcion/dashboard`.
- [ ] CA35. Con `medico1@clinica.com` / `123456` se llega a `/medico/agenda`, con los enlaces "Mi agenda" y "Configuración IA". Al abrir `/recepcion/agenda`, se redirige a `/medico/agenda`.
- [ ] CA36. Con sesión iniciada, al recargar la página se mantiene la sesión, y al abrir `/login` se redirige al inicio del rol. "Cerrar sesión" lleva a `/login` y deja `clinica_sesion` en `null`.
- [ ] CA37. Las 13 rutas de `CLAUDE.md` resuelven, con sesión del rol correcto, a su layout con `EnConstruccionView` (salvo `/login`). Una ruta inexistente redirige a `/`.
- [ ] CA38. Sin diseño visual (RNF7): `grep -rln "<style" src`, `find src -name "*.css"` y `grep -rn "\.css" src` no devuelven nada.
- [ ] CA39. En la primera carga, con `localStorage` vacío, en DevTools aparecen las claves `clinica_usuarios`, `clinica_pacientes`, `clinica_medicos`, `clinica_consultorios`, `clinica_citas`, `clinica_consultas`, `clinica_recetas` y `clinica_version`.

**Documentación**
- [ ] CA40. `SETUP.md` refleja:
  - los comandos de instalación por fase, sin `npm create vite` y sin `@primeuix/themes` ni `primeicons`;
  - la tabla de dependencias sin `@primeuix/themes` ni `primeicons`, con `primevue` descrito sin tema;
  - el árbol sin `assets/main.css`;
  - los archivos nuevos `semillaService.js`, `router/acceso.js`, `views/EnConstruccionView.vue`, `components/comunes/MarcoAplicacion.vue`, `tests/router/` y `tests/data/`;
  - las filas `layouts/`, `router/` y `main.js` en la tabla de capas;
  - la firma `leer(clave, porDefecto = [])`;
  - el "Estado actual" actualizado.

## Archivos afectados
| Acción | Ruta | Propósito |
|---|---|---|
| modificar | SETUP.md | Instalación por fase, dependencias sin tema ni íconos, árbol sin `main.css`, archivos nuevos, capas de layouts/router/main.js, firma de `leer`, estado actual |
| crear | package.json | Metadatos, scripts y dependencias |
| crear | package-lock.json | Lo genera `npm install` |
| crear | index.html | Punto de entrada de Vite (`lang="es"`) |
| crear | vite.config.js | Vite + alias `@` + Vitest con jsdom |
| crear | eslint.config.js | ESLint flat config (JS + Vue) |
| crear | src/main.js | Semilla, Pinia, Router, PrimeVue sin tema, servicios de Toast/Confirm |
| crear | src/App.vue | Toast, ConfirmDialog y RouterView globales |
| crear | src/services/storage.js | `leer`, `guardar`, `CLAVES` |
| crear | src/services/semillaService.js | `cargarSemilla`, `inicializarDatos` |
| crear | src/services/authService.js | `login`, `logout`, `sesionActual` |
| crear | src/stores/auth.js | `useAuthStore`: sesión, cargando, error |
| crear | src/data/seed.js | `generarDatosSemilla`, `VERSION_SEMILLA`, `TRANSCRIPCION_DEMO` |
| crear | src/utils/validaciones.js | `validarLogin` |
| crear | src/router/acceso.js | `INICIO_POR_ROL`, `resolverAcceso` |
| crear | src/router/index.js | Las 13 rutas + guard global |
| crear | src/components/comunes/MarcoAplicacion.vue | Cascarón: enlaces por rol, usuario, cerrar sesión, RouterView |
| crear | src/layouts/RecepcionLayout.vue | Enlaces de recepción |
| crear | src/layouts/MedicoLayout.vue | Enlaces de médico |
| crear | src/views/LoginView.vue | Cascarón: formulario de login |
| crear | src/views/EnConstruccionView.vue | Vista provisional de rutas de fases futuras |
| crear | tests/services/storage.test.js | CA11–CA13 |
| crear | tests/data/seed.test.js | CA14–CA22 |
| crear | tests/services/semillaService.test.js | CA23–CA25 |
| crear | tests/services/authService.test.js | CA26–CA29 |
| crear | tests/utils/validaciones.test.js | CA30 |
| crear | tests/router/acceso.test.js | CA31 |

## Tareas
- [x] T1. Actualizar `SETUP.md`:
  - reemplazar "Generar el proyecto (fase 1)" por los comandos de instalación por fase (fase 1: los de CA2; fase 3: FullCalendar; fase 5: pdfmake), sin `npm create vite`;
  - en la tabla de dependencias, quitar las filas o menciones de `@primeuix/themes` y `primeicons`, describir `primevue` como "Componentes UI (Toast, ConfirmDialog, DataTable, Dialog, …), sin tema: el diseño lo aporta el usuario", y quitar "(los instala `create vite`)" de la fila de `vite`;
  - en el árbol, quitar `assets/main.css`;
  - agregar al árbol `services/semillaService.js`, `router/acceso.js`, `views/EnConstruccionView.vue`, `components/comunes/MarcoAplicacion.vue`, `tests/router/` y `tests/data/`;
  - en la tabla de capas, agregar `layouts/` (igual que `views/`), `router/` (puede importar views, layouts, stores y `acceso.js`; no puede importar services) y `main.js` (raíz de composición: puede importar todo);
  - documentar `leer(clave, porDefecto = [])` y `CLAVES`.
- [x] T2. Crear `package.json` (RF1) e instalar las dependencias de runtime y de desarrollo de CA2 con `npm install` / `npm install -D`.
- [x] T3. Crear `vite.config.js` y `eslint.config.js` (RF2).
- [x] T4. Crear `index.html` (RF3).
- [x] T5. Crear `src/services/storage.js` (RF4).
- [x] T6. Tests: `tests/services/storage.test.js` (CA11–CA13).
- [x] T7. Crear `src/utils/validaciones.js` con `validarLogin` (RF9).
- [x] T8. Tests: `tests/utils/validaciones.test.js` (CA30).
- [x] T9. Crear `src/data/seed.js` (RF5).
- [x] T10. Tests: `tests/data/seed.test.js` (CA14–CA22).
- [x] T11. Crear `src/services/semillaService.js` (RF6).
- [x] T12. Tests: `tests/services/semillaService.test.js` (CA23–CA25).
- [x] T13. Crear `src/services/authService.js` (RF7).
- [x] T14. Tests: `tests/services/authService.test.js` (CA26–CA29).
- [x] T15. Crear `src/stores/auth.js` (RF8).
- [x] T16. Crear `src/router/acceso.js` (RF10).
- [x] T17. Tests: `tests/router/acceso.test.js` (CA31).
- [x] T18. Crear `src/views/EnConstruccionView.vue` (RF15) y `src/views/LoginView.vue` (RF12), en HTML plano sin estilos (RNF7).
- [x] T19. Crear `src/components/comunes/MarcoAplicacion.vue` (RF13), en HTML plano sin estilos (RNF7).
- [x] T20. Crear `src/layouts/RecepcionLayout.vue` y `src/layouts/MedicoLayout.vue` (RF14).
- [x] T21. Crear `src/router/index.js` (RF11).
- [x] T22. Crear `src/App.vue` (RF16) y `src/main.js` (RF17), sin importar CSS.
- [x] T23. Actualizar "Estado actual" en `SETUP.md`: fase 1 implementada, proyecto Vue generado.
- [x] T24. Verificar: correr `npm run lint`, `npm run test -- --run` y `npm run build`, los greps de CA6–CA10 y CA38, y la revisión manual CA32–CA37 y CA39.

## Fuera de alcance
- Diseño visual y estilos: tema de PrimeVue, íconos, CSS, responsive, modo oscuro, disposición de la navegación y resaltado del enlace activo. Lo aporta el usuario y se aplica después sobre los cascarones de esta fase, sin cambiar su lógica.
- CRUD de pacientes, médicos y consultorios, y `utils/formato.js` con el locale `es` de Day.js (fase 2).
- Agenda con FullCalendar, `citasService.js` y validación de solapamiento y horario al crear o reprogramar citas (fase 3). La semilla respeta esas reglas por construcción, pero no se implementa el validador.
- Agenda del médico, atención, `data/cie10.js` e historia clínica (fase 4).
- Recetas, pdfmake y PDFs (fase 5).
- Dashboard con indicadores, respaldo JSON, botón "Restablecer datos de demo" y README con las limitaciones (fase 6).
- Asistente IA, `clinica_config` y pantalla de configuración (fase 7).
- Traducción del `locale` de PrimeVue al español: se hace en la fase 2, cuando se use el primer `ConfirmDialog` o `DatePicker`.
- Redirección a la URL original después del login (`?redirect=`).
- Carpeta `public/` y favicon.
- Tests de componentes, stores y `router/index.js`.

## Decisiones abiertas
- Interpretación de "semana actual" para las citas semilla. **Recomendación (aplicada en RF5):** una ventana móvil de `hoy − 3` a `hoy + 3` días. Así siempre hay citas pasadas (atendidas, con consultas e historias) y citas de hoy y futuras (programadas o confirmadas), sea cual sea el día de la semana. Con la semana calendario de lunes a sábado, un lunes no habría citas pasadas ni consultas. Si se prefiere la semana calendario, hay que ajustar RF5, CA17 y CA19.
- PrimeVue en la fase 1. **Recomendación (aplicada):** instalar `primevue` y registrar solo `ToastService`, `ConfirmationService`, `Toast` y `ConfirmDialog`, sin preset de tema ni CSS, para tener el feedback de errores que usan las fases siguientes. Si el usuario prefiere no usar PrimeVue en esta fase, hay que quitar `primevue` de CA2 y RF17, sacar `Toast` y `ConfirmDialog` de RF16, y en RF12 y CA33 reemplazar el Toast por un mensaje de error en línea (por ejemplo, un `<p role="alert">`).
