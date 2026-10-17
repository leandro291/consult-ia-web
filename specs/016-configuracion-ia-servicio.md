# 016 — Servicio de IA con URL por variable de entorno

Estado: implementado
Fase: 7
Depende de: 001 (router, `storage.js`), 005 (menú del médico)

## Contexto
Reemplaza por completo la versión anterior de este spec (pantalla "Configuración IA" con `clinica_config`), que se implementó sin commitear ni llegar a `implementado`. Ahora la URL del servidor de IA sale de `import.meta.env.VITE_IA_SERVIDOR_URL` en build time y el asistente está activo solo si tiene valor (sección "iaService.js" de `CLAUDE.md`). En el árbol de trabajo quedan restos del ciclo anterior: `ConfiguracionView.vue`, su ruta, la entrada de menú, `stores/ia.js`, `validarConfigIa`, `CLAVES.config` y un `iaService.js` que lee la URL de `localStorage`. Se conserva de `iaService.js` lo que sigue valiendo (POST, timeout de 30 s, extracción y validación del JSON, errores) y se borra el resto. Sin dependencias nuevas: Vite ya expone las variables `VITE_*`.

Secuencia prevista de la fase 7 (solo se escribe este spec):
- 016 (este): `iaService.js` con la URL por variable de entorno y limpieza de la configuración.
- 017: dictado y asistente en Atender cita (`useDictado.js`, `anonimizar.js`, `useAsistenteConsulta.js`, panel, "Generar con IA" deshabilitado cuando el asistente no está activo, prellenado "Sugerido por IA").
- 018: borrador de receta desde la IA, `alergias.js` e `indicacionesPaciente` en el PDF.

## Requisitos
- RF1. No existe la pantalla de configuración del asistente: ni la ruta `/medico/configuracion`, ni la vista, ni su entrada en el menú del médico. La app no guarda ni lee `clinica_config`.
- RF2. `generarSugerencias(transcripcion, contexto)` envía el POST a la URL de `VITE_IA_SERVIDOR_URL` (sin espacios alrededor). No recibe la URL por parámetro.
- RF3. Si la variable está vacía, tiene solo espacios o no está definida, `generarSugerencias` no hace ninguna petición y lanza `Error("El asistente de IA no está configurado en esta instalación.")`.
- RF4. Se mantienen sin cambios el timeout de 30 s, la extracción del JSON (sin bloques ```json, de la primera `{` a la última `}`), la validación mínima de estructura del contrato y los mensajes de 429, 5xx, otro código no 2xx, timeout y respuesta mal formada que ya tiene `iaService.js`.
- RF5. El mensaje de sin conexión ya no le pide al médico revisar una URL que no puede editar: "No se pudo conectar con el servidor de IA. Revise su conexión o intente más tarde."
- RF6. El repositorio incluye `.env.example` con `VITE_IA_SERVIDOR_URL=` vacío, versionado en git, y `SETUP.md` explica cómo configurar la variable.

## Criterios de aceptación
- [ ] CA1. Dado un médico logueado, entonces su menú lateral solo tiene "Mi agenda", y al escribir `/medico/configuracion` en la barra de direcciones termina en su agenda (no ve "Configuración IA" ni "En construcción").
- [ ] CA2. Dado el proyecto, `npm run build` y `npm run lint` pasan, y no queda ninguna referencia a `ConfiguracionView`, `useIaStore`, `validarConfigIa`, `leerConfig`, `guardarConfig`, `probarConexion` ni `clinica_config` en `src/`.
- [ ] CA3. Dado `VITE_IA_SERVIDOR_URL` vacía o ausente, cuando se llama a `generarSugerencias`, entonces lanza el error de RF3 y `fetch` no se ejecuta.
- [ ] CA4. Dado `VITE_IA_SERVIDOR_URL=https://servidor/ia` y un servidor que responde el JSON del contrato (solo o dentro de un bloque ```json con texto alrededor), cuando se llama a `generarSugerencias('texto', contexto)`, entonces hace un POST a esa URL con `{ transcripcion, contexto }` y devuelve el objeto.
- [ ] CA5. Con la variable definida y el servidor apagado, se obtiene el mensaje de RF5; con 429, 500, 404, timeout o JSON inválido, los mensajes que ya tenía el servicio (RF4).
- [ ] CA6. Dado un clon limpio del repositorio, entonces `.env.example` está versionado y los `.env` y `.env.local` siguen ignorados por git.

## Tareas
- [x] T1. modificar `SETUP.md`:
  - árbol: agregar `.env.example` en la raíz; quitar `stores/ia.js` y `ConfiguracionView` de `views/medico/`; comentario de `iaService.js`: "POST al servidor de IA (URL de `VITE_IA_SERVIDOR_URL`) y validación";
  - nueva subsección "Variables de entorno" en "Instalación y comandos": copiar `.env.example` a `.env`, completar `VITE_IA_SERVIDOR_URL`, reiniciar `npm run dev` o volver a hacer `npm run build` (Vite la fija en build time); vacía = asistente desactivado; la URL queda visible en el JS del build, así que no lleva secretos;
  - "Estado actual": fase 7 iniciada con el spec 016 (servicio de IA con URL por variable de entorno, sin pantalla de configuración).
- [x] T2. modificar `.gitignore`: agregar `!.env.example` después de `.env*`.
- [x] T3. crear `.env.example`: línea `VITE_IA_SERVIDOR_URL=` con un comentario en español que explique su uso.
- [x] T4. eliminar `src/views/medico/ConfiguracionView.vue`.
- [x] T5. modificar `src/router/index.js`: quitar el import de `ConfiguracionView` y la hija `configuracion`.
- [x] T6. modificar `src/layouts/MedicoLayout.vue`: quitar la entrada "Configuración IA" del menú.
- [x] T7. eliminar `src/stores/ia.js`: sin configuración ni prueba de conexión no le queda estado ni caller.
- [x] T8. modificar `src/services/iaService.js`: quitar `leerConfig`, `guardarConfig`, `probarConexion`, sus constantes y los imports de `storage.js` y `validaciones.js`; `generarSugerencias` según RF2, RF3 y RF5.
- [x] T9. modificar `src/utils/validaciones.js`: quitar `validarConfigIa` y `esUrlValida` (solo las usaba la configuración).
- [x] T10. modificar `src/services/storage.js`: quitar `config` de `CLAVES`.

## Fuera de alcance
- Todo lo de los specs 017 y 018 (ver Contexto), incluida la función que le dice a la interfaz si el asistente está activo: la agrega el 017, que es quien la usa.
- "Probar conexión": se elimina `probarConexion` en vez de dejarla exportada. Sin pantalla que la dispare no tiene caller, y gasta una llamada real a la IA. Si hiciera falta, se reescribe en pocas líneas sobre `generarSugerencias`.
- El servidor de IA en sí: es externo. Si no permite CORS, el navegador lo informa como "sin conexión" (RF5).
- Crear un `.env` o `.env.local` con una URL real: lo hace cada instalación.
- Pie de receta en la Historia clínica: queda para un spec posterior.

## Decisiones abiertas
- **Maqueta `docs/diseno/pantallas/ConfiguracionIA.dc.html`**: queda sin pantalla que la implemente. Se propone no tocarla en este spec. La alternativa es borrarla y quitar su fila de `docs/diseno/README.md`, y para eso hace falta permiso para leer ese README.
- **`.env.local` en vez de `.env`** en la guía de T1: los dos quedan ignorados, pero `.env.local` es la convención de Vite para valores locales. Si se prefiere `.env`, solo cambia el texto de `SETUP.md`.
