# 019 — Puesta a punto de las pruebas unitarias

Estado: implementado
Fase: 8
Depende de: 001 (base del proyecto: Vitest, jsdom y los primeros tests), 018

## Contexto
Arranca la fase 8. La infraestructura de pruebas ya existe desde el spec 001: `vitest` y `jsdom` instalados, `test: { environment: 'jsdom' }` y alias `@/` en `vite.config.js`, script `test` en `package.json` y 8 archivos en `tests/` (`services/` storage, authService y semillaService; `utils/` formato y validaciones; `data/seed`; `router/acceso`; `views/LoginView`). En cambio, `SETUP.md` dice que "todavía no existen archivos de test" y que "hasta la fase 8 no se escriben ni corren tests", y su árbol de `tests/` no tiene carpeta para `src/pdf/`, que el catálogo `docs/funciones-testeables.md` incluye. Este spec no agrega dependencias, configuración ni casos de prueba: deja la documentación alineada con el repo y deja la suite actual pasando (corrigiendo lo que falle) como base para los specs de tests.

## Requisitos
- RF1. `SETUP.md` describe el estado real de las pruebas: ya hay tests y la fase 8 está en curso, sin frases que digan lo contrario.
- RF2. El árbol de `tests/` de `SETUP.md` espeja todas las capas con funciones del catálogo `docs/funciones-testeables.md`, incluida `pdf/`.
- RF3. `SETUP.md` nombra `docs/funciones-testeables.md` como el catálogo de funciones a probar y `.claude/skills/test-unit/SKILL.md` como la guía para escribir los casos, con el orden de prioridad de `CLAUDE.md` si chocan (por ejemplo, el `fetch` simulado de `iaService.js` que pide `SETUP.md` frente a "nada de piezas falsas" de la skill: manda `SETUP.md`).
- RF4. La suite existente corre y pasa con el comando de una sola pasada de `SETUP.md` (`npm run test -- --run`). Si algún test falla, se corrige en este spec, atacando la causa real.

## Criterios de aceptación
- [ ] CA1. Al leer la sección "Organización de los tests" de `SETUP.md`, no dice que no hay archivos de test, y el bloque "Qué se testea" no dice que hasta la fase 8 no se escriben ni corren tests.
- [ ] CA2. El árbol de `SETUP.md` muestra `tests/pdf/` (con `comunes.test.js` como ejemplo) junto a las carpetas que ya lista.
- [ ] CA3. "Organización de los tests" de `SETUP.md` enlaza `docs/funciones-testeables.md` y `.claude/skills/test-unit/SKILL.md`, y aclara que ante un conflicto manda `SETUP.md`.
- [ ] CA4. El párrafo "Estado actual" de `SETUP.md` menciona que la fase 8 empezó con el spec 019.
- [ ] CA5. Con `npm run test -- --run`, Vitest encuentra los 8 archivos de `tests/` y todos pasan.
- [ ] CA6. `npm run lint` y `npm run build` terminan sin errores.
- [ ] CA7. Dado que un test falló en la primera corrida, cuando el developer lo corrige, entonces el cambio está solo en el código de producción o solo en el test (según dónde esté la causa real), el reporte nombra el test, el mensaje y la causa, y la suite completa vuelve a pasar (CA5).

## Tareas
- [x] T1. modificar `SETUP.md`: corregir "Qué se testea" y "Organización de los tests" (CA1 y CA3), agregar `tests/pdf/` al árbol (CA2) y la fase 8 al "Estado actual" (CA4).
- [x] T2. verificar: correr `npm run test -- --run`, `npm run lint` y `npm run build` (CA5 y CA6). Si todo pasa, no se toca ningún archivo.
- [ ] T3. (condicional, solo si T2 falla en los tests) por cada test que falla: determinar la causa real y corregir **uno** de los dos lados, el archivo de `src/` si el código rompe un comportamiento esperado por `CLAUDE.md` o por un spec implementado, o el archivo de `tests/` si lo que quedó desactualizado es el test (por ejemplo, un spec posterior cambió la semilla o el login a propósito). Nunca ambos "por si acaso". Volver a correr T2 hasta que pase, dentro del límite de ciclos developer↔reviewer de `CLAUDE.md` (CA7).

## Fuera de alcance
- Escribir o ampliar casos de prueba, que van en los specs siguientes, uno por grupo del catálogo.
- Exportar las funciones internas que el catálogo marca como **(interna)** (`limpiarSugerencia` y `limpiarBorradorReceta` de `useAsistenteConsulta.js`, `documentoReceta` de `recetaPdf.js`): lo decide el spec que las pruebe.
- Cobertura (`@vitest/coverage-v8`), `setupFiles` globales, `test.include` explícito, `globals: true` y scripts nuevos en `package.json`: lo actual alcanza (Vitest ya encuentra `tests/**/*.test.js` y los tests limpian `localStorage` en `beforeEach`, según el spec 001).
- Archivos de test vacíos o esqueletos con `it.todo` por función o archivo del catálogo: cada carpeta se crea con su primer test real (decisión del usuario).
- Cambios en `CLAUDE.md`.
