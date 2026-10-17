# 021 — Gráficos del panel del día

Estado: implementado
Fase: 6
Depende de: 010 (Panel del día)

## Contexto
El Panel del día (`/recepcion/dashboard`, `src/views/recepcion/DashboardView.vue`) muestra hoy, dentro de `PaginaListado`, las pestañas-filtro por estado y la tabla de citas de hoy, y ya carga los stores de citas y médicos. Este spec agrega debajo una segunda hoja, "Resumen de la semana", con dos gráficos: citas por día y estado, y carga por médico. Se agrega la dependencia `chart.js` (hoy listada como opcional junto a `vue-chartjs`), usada solo a través del componente `Chart` de PrimeVue (`primevue/chart`, ya instalado con `primevue`, que importa `chart.js/auto`); no se usa `vue-chartjs`. Ninguna maqueta de `docs/diseno/pantallas/` trae gráficos: el diseño lo aprobó el usuario y se documenta en `DESIGN.md` (T3). No se tocan servicios ni stores.

## Requisitos
- RF1. Debajo de la hoja de la tabla del día, como hoja hermana (nunca dentro de la hoja de `PaginaListado`), aparece la hoja simple "Resumen de la semana" con los dos gráficos: lado a lado desde 1024px de ancho y apilados por debajo (mismo corte de 1023px que el resto de las vistas).
- RF2. La semana es la actual, de lunes a sábado (días 1..6, como `horario.dias`); el lunes se obtiene con Day.js en locale `es` (semana que empieza en lunes), así que un domingo muestra la semana lunes-sábado que acaba de terminar. Las citas fuera de esos 6 días no cuentan en ningún gráfico.
- RF3. Gráfico "Citas de la semana": barras apiladas, una por día (etiquetas con los nombres de `DIAS_SEMANA` de `utils/formato.js`) y una serie por estado: "Programadas" (suma `programada` y `confirmada`, igual que las pestañas del spec 010), "Atendidas", "Canceladas" y "No asistió".
- RF4. Colores de las series desde los tokens de `DESIGN.md` leídos en ejecución de las variables CSS (`getComputedStyle`), sin hex en el código: Programadas `--color-tinta`, Atendidas `--color-sello`, Canceladas `--color-apagado` con borde `--color-texto-secundario` (el relleno solo no contrasta con el papel), No asistió `--color-texto-secundario`.
- RF5. Gráfico "Carga por médico": barras horizontales, una por médico de la lista (incluidos los que tienen 0), con la cantidad de citas de la semana **sin contar las canceladas**; ordenadas de mayor a menor carga. Etiqueta `Dr. <primer apellido> · <especialidad>` (ej. "Dr. Salas · Medicina General"). Color `--color-tinta`. Las citas de un médico que no está en la lista se ignoran.
- RF6. Accesibilidad (ningún estado indicado solo con color): el gráfico 1 muestra leyenda con el texto de cada serie; ambos tienen tooltips con serie y cantidad; cada gráfico expone `role="img"` con un `aria-label` que resume los totales (ej. "Citas de la semana: 12 programadas, 5 atendidas, 2 canceladas, 1 no asistió" y "Carga por médico: Dr. Salas 6, Dr. Paredes 4, Dr. Montoya 3").
- RF7. Si la semana no tiene citas, en lugar de cada gráfico se lee "No hay citas esta semana.". Sin tarjetas de métricas (Don't de `DESIGN.md`).
- RF8. Los conteos salen de dos funciones puras nuevas en `src/utils/estadisticas.js`:
  - `citasPorDiaYEstado(citas, lunes)`: `lunes` en `YYYY-MM-DD`; devuelve `{ programadas, atendidas, canceladas, noAsistio }`, cada uno un array de 6 números (índice 0 = lunes … 5 = sábado); ignora citas fuera de esos 6 días.
  - `citasPorMedico(citas, medicos)`: recibe las citas ya limitadas a la semana; devuelve `[{ medicoId, etiqueta, total }]` según RF5 (sin canceladas, todos los médicos, orden descendente).
- RF9. Los gráficos se recalculan al cambiar las citas del store (por ejemplo, al crear una cita con "Nueva cita" del mismo panel), sin recargar la página.

## Criterios de aceptación
- [ ] CA1. Dado un usuario de recepción en el Panel del día, cuando la pantalla carga, entonces debajo de la tabla del día ve la hoja "Resumen de la semana" con "Citas de la semana" y "Carga por médico" lado a lado a 1366px de ancho y apilados a 900px, y la tabla, las pestañas y "Nueva cita" funcionan igual que antes.
- [ ] CA2. Dada una cita `confirmada` del miércoles de esta semana, cuando se mira "Citas de la semana", entonces suma en la serie "Programadas" del miércoles; una cita del domingo o de la semana siguiente no aparece en ningún gráfico.
- [ ] CA3. Dado un médico con 3 citas esta semana, una de ellas `cancelada`, cuando se mira "Carga por médico", entonces su barra marca 2 con la etiqueta "Dr. <apellido> · <especialidad>", y un médico sin citas en la semana aparece con 0.
- [ ] CA4. Dado cualquiera de los dos gráficos, cuando se pasa el mouse por una barra se ve un tooltip con la serie y la cantidad, el gráfico 1 tiene leyenda con los cuatro textos de serie, y un lector de pantalla anuncia el `aria-label` con el resumen de totales.
- [ ] CA5. Dado el panel abierto, cuando se crea una cita de hoy con "Nueva cita", entonces "Citas de la semana" y "Carga por médico" reflejan la nueva cita sin recargar.
- [ ] CA6. Dado un `localStorage` sin citas en la semana actual, cuando se abre el panel, entonces cada gráfico se reemplaza por "No hay citas esta semana.".
- [ ] CA7. `tests/utils/estadisticas.test.js` cubre RF8 (suma de programada y confirmada, exclusión de fechas fuera de lunes-sábado, exclusión de canceladas por médico, médico con 0, orden, etiqueta) y pasa; `npm run lint`, `npm run build` y `npm run test -- --run` terminan sin errores.
- [ ] CA8. `package.json` tiene `chart.js` en `dependencies` y no tiene `vue-chartjs`; ningún archivo de `src/services/` ni `src/stores/` cambia.

## Tareas
- [x] T1. modificar `SETUP.md`: fila de Dependencias → `chart.js` (no opcional) "Gráficos del panel del día, usado vía el componente `Chart` de PrimeVue"; en "Instalación por fase" reemplazar el bloque de opcionales por `# Fase 6 (gráficos del panel del día)` + `npm install chart.js`; árbol: `utils/estadisticas.js` (conteos de citas por día y estado y por médico), `ResumenSemana.vue` en `components/citas/` y `estadisticas.test.js` en `tests/utils/`; mencionar el spec 021 en "Estado actual".
- [x] T2. modificar `package.json` (y `package-lock.json`): `npm install chart.js`.
- [x] T3. modificar `DESIGN.md`: en Components, subsección breve "Gráficos (panel del día)" con la hoja simple aparte bajo la tabla, colores por estado de RF4, leyenda y tooltip con texto y `aria-label` de resumen; en Layout, aclarar que el panel del día suma esta hoja secundaria a la principal.
- [x] T4. crear `src/utils/estadisticas.js`: `citasPorDiaYEstado` y `citasPorMedico` (RF8).
- [x] T5. crear `src/components/citas/ResumenSemana.vue`: props `citas` y `medicos`; calcula el lunes, filtra la semana, usa `estadisticas.js` y arma los dos `Chart` de `primevue/chart` con colores de tokens, leyenda, tooltips, `aria-label`, estado vacío y disposición responsive (RF1 a RF7, RF9).
- [x] T6. modificar `src/views/recepcion/DashboardView.vue`: agregar `<ResumenSemana>` después de `<PaginaListado>`, pasando `citas.lista` y `medicos.lista`.
- [x] T7. modificar `docs/funciones-testeables.md`: sección `estadisticas.js` con las dos funciones.
- [x] T8. crear `tests/utils/estadisticas.test.js`: casos de CA7, según `SETUP.md` y la skill `test-unit`.

## Fuera de alcance
- Otros gráficos o indicadores (por consultorio, por especialidad, históricos o de otras semanas), selector de semana y exportar los gráficos.
- Filtrar los gráficos con las pestañas de estado o con el médico: las pestañas siguen filtrando solo la tabla del día.
- Clic en una barra para navegar a la agenda.
- Actualizar la maqueta `docs/diseno/pantallas/RecepcionLayout.dc.html` (ver Decisiones abiertas).
- Cambios en servicios, stores, `PaginaListado.vue` o en el resto del dashboard de la fase 6 (exportar/importar, restablecer demo).

## Decisiones (resueltas por el usuario)
- Canceladas: relleno `--color-apagado` con borde `--color-texto-secundario` (RF4).
- "Carga por médico" se ordena de mayor a menor carga (RF5).
- Solo se actualiza `DESIGN.md` (T3); la maqueta `RecepcionLayout.dc.html` no se toca.
