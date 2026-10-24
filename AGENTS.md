# Instrucciones comunes de S0 Core

Este repositorio comparte contexto, especificaciones y cambios OpenSpec entre
personas y agentes. No fija un proveedor ni un modelo. Usa el mismo contenido y
las mismas reglas funcionales con cualquier cliente que pueda leer los archivos.

## Fuentes y raíz de trabajo

- Lee `contexto/proyecto.md` cuando la tarea requiera conocimiento del sistema.
  Las reglas y el schema están en `openspec/config.yaml`.
- `openspec/specs/` describe lo vigente; `openspec/changes/` contiene propuestas
  e historial. Distingue una propuesta activa de un requisito consolidado.
- En un consumidor con `store:`, resuelve la raíz con `openspec context --json`.
  Lee el contexto y los artefactos de esa raíz. Si no se resuelve, comunica la
  limitación sin crear specs locales ni elegir un sistema vecino.
- Código y tests ejecutables viven en los repositorios consumidores autorizados.

## Capacidades compartidas

- Refinamiento PO: [.agents/skills/refinar-hdu/SKILL.md](.agents/skills/refinar-hdu/SKILL.md).
- Preparación QA: [.agents/skills/preparar-pruebas/SKILL.md](.agents/skills/preparar-pruebas/SKILL.md).

Lee la skill correspondiente al realizar esa tarea, aunque el cliente no la
descubra automáticamente. Resolver un store no instala sus skills ni carga sus
instrucciones en una sesión iniciada en otro repositorio. Si no puedes leer la
fuente requerida, identifica el acceso faltante sin inventar su contenido.

Consultar o preparar casos no escribe ni ejecuta pruebas por defecto. Respeta
las acciones ya autorizadas por el usuario y sus destinos; Jira es opcional.
No atribuyas aprobaciones humanas ni ejecuciones a archivos o casos preparados.
Redacta en español y conserva trazabilidad con fuentes y revisiones identificables.

## Mantenimiento de la plantilla

Mantén las reglas comunes en este archivo y los métodos PO/QA sólo en sus skills
canónicas de `.agents/skills/`. `CLAUDE.md` y las entradas propias de `.claude/`
son puentes a esas fuentes. No necesitan enlaces simbólicos ni scripts de copia.
Los workflows `openspec-*` son generados por OpenSpec; actualízalos con la CLI
acordada, sin editarlos a mano. Valida los artefactos con
`openspec validate --all --strict --no-interactive`; eso no ejecuta tests de producto.
