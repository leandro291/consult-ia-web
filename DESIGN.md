---
name: Clínica Demo
description: Formulario en triplicado. Una consulta genera tres copias; la IA escribe a lápiz y el médico confirma en tinta.
colors:
  tinta: "#1D3FAF"
  tinta-oscura: "#152E80"
  tinta-suave: "#E9EEFB"
  tinta-clara: "#DCE3FA"
  sello: "#6A2BB0"
  sello-suave: "#F4EEFB"
  sello-oscuro: "#4B1F80"
  alerta: "#B3122E"
  alerta-oscura: "#7A0C1F"
  alerta-suave: "#FCE8EC"
  alerta-clara: "#FFE3E8"
  copia-amarilla: "#FFE14D"
  texto-copia-amarilla: "#3D3400"
  copia-rosa: "#FF9EC4"
  texto-copia-rosa: "#4A0F2A"
  papel: "#FFFFFF"
  mesa: "#E6E9EF"
  campo: "#F4F6FA"
  texto: "#16192B"
  texto-secundario: "#4A5068"
  lapiz: "#5A5F70"
  lapiz-suave: "#F7F7F9"
  apagado: "#EEF0F4"
  pasado: "#F7F8FB"
  linea: "#C5CCDC"
  renglon: "#DDE2EC"
  perforacion: "#AEB6C9"
typography:
  display:
    fontFamily: "Public Sans, system-ui, sans-serif"
    fontSize: "60px"
    fontWeight: 800
    lineHeight: 1.02
    letterSpacing: "-0.035em"
  headline:
    fontFamily: "Public Sans, system-ui, sans-serif"
    fontSize: "30px"
    fontWeight: 800
    lineHeight: 1.1
    letterSpacing: "-0.025em"
  title:
    fontFamily: "Public Sans, system-ui, sans-serif"
    fontSize: "20px"
    fontWeight: 700
    lineHeight: 1.3
    letterSpacing: "-0.01em"
  body:
    fontFamily: "Public Sans, system-ui, sans-serif"
    fontSize: "15px"
    fontWeight: 400
    lineHeight: 1.55
    fontFeature: "tnum"
  label:
    fontFamily: "Public Sans, system-ui, sans-serif"
    fontSize: "12px"
    fontWeight: 600
    lineHeight: 1.3
    letterSpacing: "0.06em"
  dato:
    fontFamily: "Courier Prime, ui-monospace, monospace"
    fontSize: "15px"
    fontWeight: 700
    lineHeight: 1.4
rounded:
  sm: "4px"
  md: "6px"
  pill: "22px"
spacing:
  xs: "4px"
  sm: "8px"
  md: "12px"
  lg: "16px"
  xl: "24px"
  2xl: "32px"
  3xl: "48px"
components:
  button-primary:
    backgroundColor: "{colors.tinta}"
    textColor: "{colors.papel}"
    rounded: "{rounded.md}"
    padding: "0 20px"
    height: "44px"
  button-primary-hover:
    backgroundColor: "{colors.tinta-oscura}"
  button-secondary:
    backgroundColor: "{colors.papel}"
    textColor: "{colors.tinta}"
    rounded: "{rounded.md}"
    padding: "0 20px"
    height: "44px"
  button-ghost:
    textColor: "{colors.tinta}"
    rounded: "{rounded.md}"
    padding: "0 14px"
    height: "44px"
  button-danger:
    backgroundColor: "{colors.alerta}"
    textColor: "{colors.papel}"
    rounded: "{rounded.md}"
    padding: "0 20px"
    height: "44px"
  button-danger-outline:
    backgroundColor: "{colors.papel}"
    textColor: "{colors.alerta}"
    rounded: "{rounded.md}"
    padding: "0 20px"
    height: "44px"
  button-icon-tabla:
    textColor: "{colors.tinta}"
    rounded: "{rounded.md}"
    size: "40px"
  button-disabled:
    backgroundColor: "{colors.apagado}"
    textColor: "{colors.lapiz}"
    rounded: "{rounded.md}"
    height: "44px"
  input:
    backgroundColor: "{colors.campo}"
    textColor: "{colors.texto}"
    typography: "{typography.body}"
    rounded: "{rounded.sm}"
    padding: "0 12px"
    height: "44px"
  input-ia:
    backgroundColor: "{colors.lapiz-suave}"
    textColor: "{colors.lapiz}"
    rounded: "{rounded.sm}"
    height: "44px"
  input-error:
    backgroundColor: "{colors.alerta-suave}"
    textColor: "{colors.texto}"
  chip-programada:
    backgroundColor: "{colors.papel}"
    textColor: "{colors.tinta}"
    rounded: "{rounded.sm}"
  chip-confirmada:
    backgroundColor: "{colors.tinta}"
    textColor: "{colors.papel}"
    rounded: "{rounded.sm}"
  chip-atendida:
    backgroundColor: "{colors.sello-suave}"
    textColor: "{colors.sello}"
    rounded: "{rounded.sm}"
  chip-cancelada:
    backgroundColor: "{colors.apagado}"
    textColor: "{colors.texto-secundario}"
    rounded: "{rounded.sm}"
  chip-no-asistio:
    textColor: "{colors.texto-secundario}"
    rounded: "{rounded.sm}"
  chip-alergia:
    backgroundColor: "{colors.alerta-suave}"
    textColor: "{colors.alerta}"
    rounded: "{rounded.sm}"
  nav-item:
    textColor: "{colors.texto}"
    rounded: "{rounded.md}"
    padding: "0 12px"
    height: "44px"
  nav-item-activo:
    backgroundColor: "{colors.tinta-suave}"
    textColor: "{colors.tinta}"
  evento-confirmado:
    backgroundColor: "{colors.tinta}"
    textColor: "{colors.papel}"
    rounded: "{rounded.sm}"
  evento-atendido:
    backgroundColor: "{colors.sello-suave}"
    textColor: "{colors.sello-oscuro}"
    rounded: "{rounded.sm}"
  hoja:
    backgroundColor: "{colors.papel}"
    rounded: "{rounded.sm}"
    padding: "28px 32px"
---

# Design System: Clínica Demo

<!-- Fuente: canvas de diseño "Clínica: Login y layouts" (https://claude.ai/artifact/LZeFPnEjFHgUwnUvs9HvR2), 17 pantallas. Los nombres de pacientes y médicos del canvas son ilustrativos; la app usa los de src/data/seed.js. -->

## Overview

**Creative North Star: "El formulario en triplicado"**

La app se comporta como el papel autocopiativo de un consultorio: una hoja blanca original y, debajo, la copia amarilla y la copia rosa. Es literal en este producto, porque una atención genera tres documentos: la consulta (original), la entrada de la historia clínica (amarilla) y la receta (rosa). La interfaz es una mesa gris fría con hojas blancas encima, renglones finos, etiquetas de formulario y una perforación punteada que separa la cabecera del contenido.

La tinta cuenta la historia del control clínico. Lo que sugiere la IA se escribe **a lápiz**: gris grafito, borde punteado, cursiva. Lo que el médico revisa pasa a **tinta azul** de bolígrafo. Lo que queda registrado lleva un **sello violeta**. El rojo no decora: solo avisa alergias, errores y que el micrófono está grabando.

Es una interfaz de trabajo (Operate): densidad media, lectura rápida de tablas y campos, cero ornamento que compita con la tarea. El carácter vive en los detalles (sello, perforación, copias asomando), no en fondos ni ilustraciones. Fondo claro por el contexto de uso: consultorio iluminado, laptop, paciente mirando la pantalla.

**Key Characteristics:**
- Hojas blancas sobre mesa gris fría; nada de crema, nada de oscuro.
- Tres copias (blanca, amarilla, rosa) como signo de identidad y como mapa de documentos.
- Lápiz → tinta → sello: el estado de un dato se lee por cómo está "escrito".
- Rojo reservado; el color nunca es la única señal (siempre ícono o texto).
- Public Sans de formulario público y Courier Prime solo para datos.

**Implementación:** los tokens se trasladan a un preset de `@primeuix/themes` (sin librerías nuevas). Los componentes de PrimeVue se tematizan para verse como estos; no se reemplazan.

## Colors

Restringida: neutros fríos de papel y mesa, un acento de tinta, y tres colores con significado fijo (sello, alerta, copias).

### Primary
- **Tinta de bolígrafo** (`tinta`): acción principal, foco, enlaces, ítem de menú activo, datos confirmados por el médico. `tinta-oscura` para hover; `tinta-suave` para fondos de selección; `tinta-clara` para texto secundario sobre fondo de tinta (6,9:1).

### Secondary
- **Sello violeta** (`sello`, `sello-suave`, `sello-oscuro`): solo para lo registrado: cita atendida, "Consulta guardada", toast de éxito. `sello-oscuro` es el texto de los eventos atendidos del calendario.

### Tertiary
- **Copia amarilla** (`copia-amarilla`): historia clínica y selección de texto. Texto encima siempre `texto-copia-amarilla`.
- **Copia rosa** (`copia-rosa`): receta. Texto encima siempre `texto-copia-rosa`.
- **Alerta** (`alerta`, `alerta-oscura`, `alerta-suave`, `alerta-clara`): alergias, errores de validación, errores de IA y el punto de grabación. `alerta-clara` es el texto secundario dentro de la franja roja.

### Neutral
- **Papel** (`papel`): hojas, menú lateral, diálogos.
- **Mesa** (`mesa`): fondo de la app detrás de las hojas.
- **Campo** (`campo`): fondo de inputs y de recuadros informativos.
- **Texto** (`texto`) y **Texto secundario** (`texto-secundario`): contenido y etiquetas. Ambos pasan AA sobre papel, campo y mesa.
- **Lápiz** (`lapiz`, `lapiz-suave`): todo lo sugerido por IA sin confirmar, y el estado deshabilitado.
- **Apagado** (`apagado`): chips cancelados, botones deshabilitados, aviso fijo de IA.
- **Pasado** (`pasado`): días y horas ya transcurridos en el calendario.
- **Línea**, **Renglón**, **Perforación** (`linea`, `renglon`, `perforacion`): bordes, renglones de tabla y bordes punteados de perforación.

### Named Rules
**The Red Is Clinical Rule.** El rojo solo aparece para alergias, errores y grabación. Si algo es rojo y no es uno de esos tres, está mal.

**The Copy Colors Mean Documents Rule.** Amarillo = historia clínica, rosa = receta. No se usan como acento decorativo ni como estado de cita.

**The Never Color Alone Rule.** Todo estado lleva texto o ícono además del color: la cita cancelada va tachada, la no asistida con borde punteado y la alergia con triángulo. En el calendario, cada evento anuncia su estado en su nombre accesible ("10:30 Luis Quispe, confirmada, alérgico a penicilina").

**Contraste verificado.** Todos los pares de texto de la paleta pasan AA (mínimo 5,57:1). `linea` y `perforacion` no llegan a 3:1 y solo pueden ser decorativos: todo campo lleva además su renglón inferior en `texto` o `lapiz`.

## Typography

**Display Font:** Public Sans (con system-ui)
**Body Font:** Public Sans (con system-ui)
**Label/Mono Font:** Courier Prime (con ui-monospace), solo datos

**Character:** Public Sans nació para formularios de gobierno: neutra, legible, seria. Courier Prime pone los datos como si estuvieran tipeados en el formulario.

### Hierarchy
- **Display** (800, 60px, 1.02): solo el titular del login.
- **Headline** (800, 30px, 1.1): título de pantalla en la cabecera.
- **Title** (700, 20px, 1.3): encabezado de hoja o sección.
- **Body** (400, 15px, 1.55): celdas, textos, formularios. Numerales tabulares activados globalmente. Máximo 65–75ch en texto corrido.
- **Label** (600, 12px, 0.06em, MAYÚSCULAS): etiquetas de campo y encabezados de tabla.
- **Dato** (Courier Prime 700, 13–15px): DNI, hora, CIE-10, colegiatura, fechas.

### Named Rules
**The Typed Data Rule.** Courier Prime solo para valores que alguien tipearía en un formulario (DNI, horas, códigos, fechas). Nunca para títulos, botones ni texto corrido.

**The No Kicker Rule.** Nada de etiquetas pequeñas sobre los títulos. La fecha y el contexto van debajo del título o a un lado.

## Layout

- **Esqueleto por rol:** menú lateral fijo de 256px (papel, borde derecho `linea`) y área principal con cabecera y contenido. Mismo esqueleto para médico y recepción; cambian el menú y la etiqueta de rol.
- **Cabecera:** título y fecha a la izquierda, acciones de pantalla a la derecha, separada del contenido por la perforación (2px punteado `perforacion`). Padding 28px 48px 22px.
- **Contenido:** padding 32px 48px 48px sobre la mesa; una hoja principal por pantalla. El panel del día suma la hoja secundaria "Resumen de la semana" bajo la principal.
- **Ritmo:** escala de 4/8 (`spacing`). Grupos internos a 8–12px; entre bloques 20–32px; más aire sobre un título que debajo.
- **Menú de íconos (72px):** en las pantallas donde el médico redacta un documento (Atender cita, Emitir receta) el menú lateral se reduce a íconos con `aria-label` para ganar ancho. En el resto va el menú completo.
- **Bloque "Siguiente paciente":** fijo en el menú completo del médico, en todas sus pantallas.
- **Franja de alergia:** en Atender cita y Emitir receta, franja roja de todo el ancho bajo la cabecera, con `role="note"` y `aria-label="Alergias del paciente"`.
- **Búsqueda:** la búsqueda global de pacientes vive en la cabecera del panel del día; el filtro de un listado vive en la barra superior de su hoja.
- **Referencia:** 1366×768 (laptop). En tablet, el menú lateral se colapsa a íconos o a un cajón; las tablas pasan a filas apiladas antes que a scroll horizontal. [A resolver en implementación.]

## Elevation & Depth

Híbrido: planos a la manera del papel, con sombra solo para lo que está "encima de la mesa". La profundidad principal la dan las copias apiladas, no las sombras.

### Shadow Vocabulary
- **Hoja** (`box-shadow: 0 1px 2px rgba(22,25,43,.08), 0 10px 28px rgba(22,25,43,.08)`): hojas de contenido sobre la mesa.
- **Toast** (`box-shadow: 0 6px 20px rgba(22,25,43,.12)`): notificaciones flotantes.
- **Diálogo** (`box-shadow: 0 20px 48px rgba(22,25,43,.35)`): diálogos sobre un velo `rgba(22,25,43,.55)`.
- **Copias apiladas:** detrás de la hoja, la amarilla desplazada 5px/5px y la rosa 10px/10px, mismo radio, sin sombra propia.

### Named Rules
**The Copies Are Earned Rule.** La hoja con copias solo aparece en pantallas del médico que generan documentos clínicos (agenda del médico, atención, receta) y en el login. Recepción usa la hoja simple.

## Shapes

Esquinas de papel, apenas redondeadas: 4px en hojas, chips y campos (solo esquinas superiores en inputs, para que la línea inferior quede como renglón), 6px en botones, avisos y diálogos, y cápsula (22px) solo para el indicador de grabación. Bordes finos: 1px en estructura, 1.5px en contornos de estado y renglón de input, 2px en avisos de alergia y en el foco. El borde punteado significa "provisional" (lápiz, no asistió, código a verificar) o "perforación".

## Components

### Buttons
- **Shape:** esquinas suaves (6px), alto mínimo 44px, peso 700.
- **Primary:** tinta con texto blanco y sombra corta. Uno solo por vista.
- **Hover / Focus:** hover pasa a `tinta-oscura`. El foco es un anillo de 2px en tinta con 2px de separación, en todos los controles.
- **Secondary:** papel con contorno de 1.5px en tinta y texto en tinta (ej. "Atender" en citas que no son la siguiente).
- **Ghost:** solo texto en tinta, para cancelar o acciones terciarias.
- **Danger:** dos pasos. El disparador va con borde rojo y texto terminado en "…" ("Cancelar cita…", "Restablecer…"); abre el `ConfirmDialog`, y solo su botón de confirmación va en rojo lleno ("Sí, cancelar").
- **Disabled:** apagado, texto lápiz y borde punteado; si depende de una configuración, al lado va un enlace que explica cómo habilitarlo ("Sin API key. Configurar IA").
- **Icon-only:** 44px; 40px dentro de filas de tabla. Siempre con `aria-label` que nombra el objeto ("Editar a Jorge Huamán Salas"), nunca solo "Editar".
- **Enlaces repetidos en filas** ("Ver", "Ver historia", "Atender") llevan `aria-label` con el nombre de la fila.
- **Orden:** en grupos de botones la acción principal va a la derecha.

### Chips (estados de cita)
- **Programada:** contorno en tinta.
- **Confirmada:** tinta llena.
- **Atendida:** sello: contorno violeta sobre `sello-suave`, MAYÚSCULAS espaciadas y rotación de −2°.
- **Cancelada:** apagado y tachado.
- **No asistió:** contorno punteado gris.
- **Alergia:** `alerta-suave` con borde y texto en alerta y el triángulo.
- **Código a verificar:** contorno punteado de lápiz y el código en Courier Prime.

### Cards / Containers (Hoja)
- **Corner Style:** 4px.
- **Background:** papel sobre mesa.
- **Shadow Strategy:** sombra Hoja; copias apiladas según la regla.
- **Border:** ninguno; el título interno se separa con la perforación.
- **Internal Padding:** 28px 32px.
- Nunca hojas dentro de hojas.

### Inputs / Fields
- **Style:** fondo `campo`, sin bordes laterales, renglón inferior de 1.5px en `texto`; etiqueta arriba en estilo Label.
- **Focus:** el renglón pasa a 2px en tinta y la etiqueta a tinta.
- **Error:** fondo `alerta-suave`, renglón de 2px en alerta y mensaje debajo con ícono, que nombra el problema y cómo corregirlo ("El DNI debe tener 8 dígitos.").
- **Sugerido por IA (a lápiz):** fondo `lapiz-suave`, borde punteado, texto lápiz en cursiva y la etiqueta "Sugerido por IA" con un lápiz. Al editarlo o aceptarlo pasa a tinta con la etiqueta "Revisado".

### Navigation
- Ítems de 44px con ícono de trazo de 20px y texto de 15px.
- **Normal:** texto. **Hover:** fondo `campo`. **Activo:** `tinta-suave` con texto en tinta, peso 700 y `aria-current="page"`.
- El pie del menú lleva el usuario (iniciales, nombre, especialidad y colegiatura) y "Cerrar sesión".
- En recepción, los conteos por estado son pestañas-filtro de la tabla (número grande y etiqueta), no tarjetas de métricas.

### Aviso de alergia (signature)
Recuadro con `alerta-suave`, borde de 2px en alerta por los cuatro lados, triángulo y título en peso 800. Se muestra al agendar y al recetar.
- **Franja:** en atención y receta, además, franja roja de todo el ancho (ver Layout).
- **Bloqueo en línea:** cuando un medicamento cruza con una alergia, la fila se enmarca en rojo y debajo aparece el bloqueo: "Posible reacción alérgica", la explicación, la casilla "Revisé la alergia y mantengo la indicación", "Mantener" deshabilitado hasta marcarla y "Quitar medicamento" como acción principal a la derecha. No es un diálogo: el médico sigue viendo el formulario. La barra inferior muestra "1 alerta de alergia sin resolver" y "Guardar consulta" queda deshabilitado.
- **Calendario:** el evento del paciente lleva el triángulo y el panel de detalle abre con el recuadro de alergia arriba.

### Dictado (signature)
Cápsula "Grabando 00:42" con un punto rojo pulsante. El pulso se desactiva con `prefers-reduced-motion`. Pausar y Detener son secundarios; Limpiar es ghost. La carga ("Analizando consulta…") va en un recuadro a lápiz con spinner de tinta.

### Sello "Consulta guardada"
Doble borde violeta, MAYÚSCULAS espaciadas, fecha y hora en Courier Prime, rotado −4°. Aparece en la hoja al guardar y habilita "Generar receta PDF".

### Calendario (tema de FullCalendar)
- **Vistas:** recepción usa Semana (filtrable por médico); el médico usa Día y Semana. Solo días y horas del horario de atención.
- **Rejilla:** franjas de 30 min; 48px en semana, 56px en día. Línea de hora en `renglon`, media hora en `apagado`. Horas en Courier Prime 12px.
- **Pasado:** días y horas transcurridos con fondo `pasado`. Hora actual: línea de 2px en tinta con punto (no rojo).
- **Hoy:** cabecera de columna en `tinta-suave` con el número en círculo de tinta.
- **Eventos:** mismos estilos que los chips de estado (programada con borde, confirmada llena, atendida en sello, cancelada tachada, no asistió punteada). En semana, hora y apellido; en día, fila completa con motivo, estado y acción.
- **Selección:** anillo oscuro doble (`0 0 0 2px papel, 0 0 0 4px texto`), distinto del foco en tinta.
- **Hueco libre:** al pasar, "+ Nueva cita HH:mm" con borde punteado en tinta.
- **Detalle:** panel flotante con alergia, horario, médico, consultorio, motivo, estado, "Reprogramar" y "Cancelar cita…".

### Tira de horario
Seis casillas (Lu a Sá) de 26×24px: día de atención en tinta llena con texto blanco, día libre con borde punteado. Debajo, el rango en Courier Prime. El contenedor lleva `aria-label` en palabras ("Lunes a viernes").

### Línea de tiempo (historia clínica)
Riel vertical de 2px en `linea` con un punto por consulta; el más reciente en copia amarilla. Cada consulta es una hoja con franja superior amarilla de 8px: diagnóstico como título, fecha y médico, signos vitales en una línea de datos, y motivo, examen, diagnóstico y plan como lista de definiciones en lectura (16px, ~72ch). Las antiguas se pliegan con "Ver detalle".

### Receta y vista previa
El formulario de receta es una hoja con franja superior rosa de 8px. Cada medicamento confirma "Sin cruce con las alergias registradas". La vista previa del PDF (A5) muestra membrete, médico, paciente, "Rp/" con la tabla, indicaciones, firma, id y QR; nada por debajo de 10px en pantalla y 10–11pt en el PDF. Descargar e Imprimir se deshabilitan con el motivo escrito debajo.

### Gráficos (panel del día)
- Hoja simple aparte bajo la tabla ("Resumen de la semana"), nunca dentro de la hoja del listado; dos gráficos lado a lado desde 1024px y apilados por debajo.
- Colores por estado, leídos de los tokens: Programadas `tinta`, Atendidas `sello`, Canceladas `apagado` con borde `texto-secundario`, No asistió `texto-secundario`. La carga por médico va en `tinta`.
- Ningún estado solo con color: leyenda con el texto de cada serie, tooltip con serie y cantidad, y `role="img"` con `aria-label` que resume los totales.
- Sin datos: "No hay citas esta semana." en lugar del gráfico. Sin tarjetas de métrica.

### Diálogos
- **Formulario:** `Dialog` de 680px sobre velo, título en 22px, perforación bajo el título, acciones abajo a la derecha.
- **Confirmación destructiva:** `alertdialog` con franja superior roja de 6px, consecuencia concreta con números ("Se borrarán 10 pacientes…"), "Cancelar" con borde y "Sí, …" en rojo lleno.
- **Aviso sin salida:** `alertdialog` sin franja, explica el motivo ("Tiene 4 consultas registradas") y solo "Entendido".

### Interruptor
Pista de 52×30px en tinta con perilla blanca; `input type="checkbox" role="switch"` real encima, con descripción asociada.

### Estados vacíos, de carga y de error
- **Vacío:** ícono de trazo o la hoja vacía punteada, título que dice qué falta, una línea con el siguiente paso y una sola acción principal. Distinguir "sin resultados" (ofrece limpiar la búsqueda) de "primer uso" (ofrece crear o cargar la demo).
- **Carga:** esqueleto con las mismas columnas que la tabla real, brillo suave desactivado con `prefers-reduced-motion`, visible solo si tarda más de 300 ms. Operaciones cortas: spinner de tinta con el texto de lo que ocurre.
- **Error:** qué pasó, qué hacer y qué se conservó ("No se cambió ningún dato", "Su transcripción sigue intacta").

## Do's and Don'ts

### Do:
- **Do** mostrar toda sugerencia de IA a lápiz hasta que el médico la toque o la acepte.
- **Do** mantener el aviso fijo "Sugerencias generadas por IA. Verifique toda la información antes de guardar." en el panel del asistente.
- **Do** tematizar las superficies del navegador: selección en copia amarilla, caret en tinta, foco en tinta y numerales tabulares.
- **Do** usar solo íconos de trazo (1.75, redondeados) de una misma familia.
- **Do** mantener contraste AA en todo el texto, incluido el secundario.

### Don't:
- **Don't** usar rojo fuera de alergias, errores y grabación.
- **Don't** usar amarillo o rosa como acento decorativo; son documentos.
- **Don't** poner bordes de color solo a la izquierda en avisos, tarjetas o filas.
- **Don't** armar el dashboard con tarjetas de métrica (número gigante con etiqueta y acento).
- **Don't** usar fondos crema, degradados, glassmorphism ni modo oscuro por defecto.
- **Don't** usar emoji como íconos.
- **Don't** abrir un diálogo cuando el aviso puede ir en línea; los diálogos son para formularios y confirmaciones.
- **Don't** usar `role="tab"` para filtros; los filtros son botones con `aria-pressed`.
- **Don't** usar el mismo anillo para selección y foco.
