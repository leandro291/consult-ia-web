# 005 — Menú lateral de los layouts por rol

Estado: implementado
Fase: 1 (layouts por rol, pendientes de diseño)
Depende de: 003

## Contexto
Hoy `MarcoAplicacion.vue` arma una cabecera simple (marca, rol, nombre, "Cerrar sesión") y un `nav` de `RouterLink` sin estilo; `RecepcionLayout` y `MedicoLayout` solo le pasan título e ítems. Este spec reemplaza eso por el menú lateral diseñado: `docs/diseno/pantallas/RecepcionLayout.dc.html`, `docs/diseno/pantallas/MedicoLayout.dc.html` y las secciones "Layout" y "Navigation" de `DESIGN.md`. Solo el `aside`: el contenido de esas maquetas (panel del día, lista del día) es de otros specs.
La cabecera de cada pantalla (título, fecha, acciones) es de cada vista según `DESIGN.md`, no del layout. Por eso la cabecera global actual se quita: lo que mostraba pasa al menú.

## Requisitos
- RF1. Esqueleto común para ambos roles: menú lateral fijo a la izquierda y área de contenido con la vista de la ruta.
- RF2. El menú muestra, de arriba abajo:
  - la marca de la clínica con la etiqueta del rol ("Recepción" o "Médico");
  - la lista de navegación del rol;
  - el pie con el usuario de la sesión y el botón "Cerrar sesión".
- RF3. Ítems de recepción, en este orden y con los íconos de la maqueta: Panel del día (`/recepcion/dashboard`), Agenda, Pacientes, Médicos, Consultorios y Respaldo.
- RF4. Ítems del médico, con los íconos de la maqueta: Mi agenda (`/medico/agenda`) y Configuración IA (`/medico/configuracion`). Las rutas con parámetro no van en el menú.
- RF5. El ítem de la sección actual se marca como activo, también en sus subrutas (`/recepcion/pacientes/:id` → Pacientes). El estado activo no depende solo del color.
- RF6. El pie muestra las iniciales y el nombre del usuario de la sesión. "Cerrar sesión" cierra la sesión y lleva a `/login`.
- RF7. En anchos de tablet (menos de 1024px) el menú se reduce a solo íconos, cada uno con nombre accesible, y el contenido gana ese ancho.

## Criterios de aceptación
- [ ] CA1. Dado un usuario de recepción logueado, cuando entra a cualquier ruta `/recepcion/*`, entonces ve el menú con la marca, la etiqueta "Recepción" y los 6 ítems de RF3 en ese orden, y cada ítem navega a su ruta.
- [ ] CA2. Dado un médico logueado, cuando entra a `/medico/agenda`, entonces ve la etiqueta "Médico" y solo los ítems "Mi agenda" y "Configuración IA".
- [ ] CA3. Dado que estoy en `/recepcion/pacientes/:id`, entonces "Pacientes" se ve activo, con `aria-current="page"` y peso de texto distinto al resto; ningún otro ítem aparece activo.
- [ ] CA4. Dado cualquier rol, entonces el pie muestra las iniciales y el nombre de la sesión, y ya no aparece la cabecera global anterior (sin nombre ni "Cerrar sesión" duplicados).
- [ ] CA5. Dado un usuario logueado, cuando presiona "Cerrar sesión", entonces llega a `/login`; y si vuelve con "Atrás" a una ruta protegida, el guard lo redirige a `/login`.
- [ ] CA6. Solo con teclado, Tab recorre los ítems y "Cerrar sesión" con foco visible. El `nav` tiene `aria-label` "Menú de recepción" o "Menú del médico", según el rol.
- [ ] CA7. Con la ventana a 800px de ancho, el menú muestra solo íconos, cada ítem conserva su nombre accesible y no aparece scroll horizontal en la página.
- [ ] CA8. La pantalla de login sigue mostrando la marca igual que antes de este spec.

## Tareas
- [x] T1. modificar `SETUP.md`:
  - en `components/comunes/`, describir `MarcoAplicacion` como "esqueleto por rol: menú lateral + contenido";
  - en "Estado actual", reemplazar "Los layouts y las demás pantallas siguen sin diseño" por "Los layouts tienen el menú lateral diseñado (spec 005); las demás pantallas siguen sin diseño".
- [x] T2. modificar `src/components/comunes/MarcaClinica.vue`: prop opcional con la etiqueta del rol debajo del nombre. Sin la prop se ve igual que hoy (CA8).
- [x] T3. modificar `src/components/comunes/MarcoAplicacion.vue`: menú lateral + área de contenido, sin la cabecera actual. Cada ítem recibe su ícono. Incluye el pie de usuario y el colapso a íconos (RF1, RF2, RF5, RF6, RF7).
- [x] T4. modificar `src/layouts/RecepcionLayout.vue`: ítems, orden, etiquetas e íconos de RF3; `aria-label` del `nav`.
- [x] T5. modificar `src/layouts/MedicoLayout.vue`: ítems e íconos de RF4; `aria-label` del `nav`.

## Fuera de alcance
- Cabeceras de cada pantalla y contenido del panel del día o de la lista del día del médico.
- Bloque "Siguiente paciente" del menú del médico: depende de las citas (fases 3 y 4).
- Menú de íconos de 72px en Atender cita y Emitir receta: se hace con esas pantallas (fases 4 y 5).
- Cajón (drawer) con botón para tablet.

## Decisiones abiertas
- Segunda línea del pie: la maqueta muestra el email en recepción y "especialidad · colegiatura" en el médico. `clinica_sesion` (modelo en `CLAUDE.md`) solo guarda `usuarioId`, `rol` y `nombre`, así que este spec muestra solo las iniciales y el nombre. Para mostrar esa línea hay que agregar `email` y `medicoId` a la sesión: cambiaría `CLAUDE.md` y `src/services/authService.js`, y la fase 4 igual va a necesitar el `medicoId`. ¿Se agrega ahora?
- Tablet: `DESIGN.md` deja abierto "íconos o cajón". Este spec elige íconos por debajo de 1024px, porque se hace solo con CSS. ¿Se confirma el corte de 1024px?
- La etiqueta del ítem cambia de "Dashboard" a "Panel del día", como en la maqueta; la ruta sigue siendo `/recepcion/dashboard`. ¿Se confirma?
