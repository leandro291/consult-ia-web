# 000 — Base documental y metodología SDD

Estado: implementado
Fase: 0 (previa a la fase 1)
Depende de: ninguno

## Contexto
El proyecto arranca solo con `CLAUDE.md` (dominio de la clínica). Hace falta:
- Una metodología Spec-Driven Development con subagentes.
- Un `SETUP.md` con la arquitectura.
- Un repositorio público.

No se genera el proyecto Vue ni se instalan dependencias en este spec.

Este archivo también sirve de **ejemplo de la plantilla** que usa `spec-writer`.

## Requisitos
### Funcionales
- RF1. `CLAUDE.md` incluye:
  - reglas de orquestación SDD;
  - la referencia a los 4 agentes;
  - la referencia obligatoria a `SETUP.md`;
  - el uso de skills globales.
- RF2. Hay 4 agentes en `.claude/agents/` (orquestador, spec-writer, developer y reviewer). Cada uno tiene el frontmatter `name`, `description`, `tools` y `model`.
- RF3. `SETUP.md` documenta:
  - el árbol de carpetas con su propósito;
  - las capas;
  - los comandos de instalación;
  - las dependencias con su propósito;
  - las convenciones de nombres.
- RF4. Repositorio público `consult-ia` en GitHub con el commit inicial.

### No funcionales
- RNF1. Persistencia con localStorage vía `src/services/`, sin backend.
- RNF2. Lint con ESLint y tests con Vitest (con jsdom).
- RNF3. Todo en español.

## Criterios de aceptación
- [x] CA1. El orquestador solo puede invocar `spec-writer`, `developer` y `reviewer`. Se detiene tras el spec para esperar la aprobación y permite como máximo 3 ciclos developer → reviewer.
- [x] CA2. `spec-writer` y `developer` no tienen la herramienta Agent.
- [x] CA3. `reviewer` no tiene Edit, Write ni Agent.
- [x] CA4. Ningún subagente hace commits: los hace el hilo principal con el OK del usuario.
- [x] CA5. El árbol de carpetas, el stack y los comandos no se repiten en `CLAUDE.md`, que apunta a `SETUP.md`.
- [x] CA6. Los modelos son: orquestador `sonnet`, spec-writer `opus`, developer `sonnet` y reviewer `sonnet`.
- [x] CA7. El repositorio es público.

## Archivos afectados
| Acción | Ruta | Propósito |
|---|---|---|
| modificar | CLAUDE.md | Metodología SDD, agentes, skills, punteros a SETUP.md |
| crear | SETUP.md | Arquitectura, instalación, dependencias, nombres |
| crear | .claude/agents/orquestador.md | Clasifica y delega |
| crear | .claude/agents/spec-writer.md | Escribe specs |
| crear | .claude/agents/developer.md | Implementa |
| crear | .claude/agents/reviewer.md | Revisa en solo lectura |
| crear | specs/000-base-documental-sdd.md | Este spec / plantilla de ejemplo |
| crear | .gitignore | Ignorar node_modules, dist, .env |

## Tareas
- [x] T1. Fusionar la metodología SDD en `CLAUDE.md` y mover el stack, el árbol y los comandos a `SETUP.md`.
- [x] T2. Crear `SETUP.md`.
- [x] T3. Crear los 4 agentes.
- [x] T4. Crear `.gitignore`.
- [x] T5. Ejecutar `git init`, hacer el commit inicial y crear el repositorio público con `gh`.

## Fuera de alcance
- Generar el proyecto Vite e instalar dependencias (fase 1).
- README (fase 6).

## Decisiones abiertas
- (ninguna)
