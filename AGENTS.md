# Instrucciones comunes de S0 Core

Este repositorio comparte contexto, especificaciones y cambios OpenSpec entre
personas y agentes. No fija un proveedor ni un modelo. Usa el mismo contenido y
las mismas reglas funcionales con cualquier cliente que pueda leer los archivos.

## Fuentes y raíz de trabajo

- Lee `context/project.md` cuando la tarea requiera conocimiento del sistema.
  Las reglas y el schema están en `openspec/config.yaml`.
- Cada instancia S0 corresponde a un sistema con uno o varios repositorios de
  código. Usa el mapa de componentes del contexto; si existe `repositories.yaml`,
  consulta sólo las entradas pertinentes. Es una extensión opcional de S0, no
  configuración de OpenSpec ni autorización para clonar o modificar repositorios.
- `openspec/specs/` describe lo vigente; `openspec/changes/` contiene propuestas
  e historial. Distingue una propuesta activa de un requisito consolidado.
- Respeta el S0 canónico de cada subsistema. Si tiene otro S0, referencia sus
  requisitos o contratos y limita aquí el cambio al alcance de este sistema;
  no dupliques sus especificaciones internas. Un cambio en ambos S0 debe
  identificar sus fuentes, límites y acciones autorizadas.
- En un consumidor con `store:`, resuelve la raíz con `openspec context --json`.
  Lee el contexto y los artefactos de esa raíz. Si no se resuelve, comunica la
  limitación sin crear specs locales ni elegir un sistema vecino.
- Código y tests ejecutables viven en los repositorios consumidores autorizados.

## Capacidades compartidas

- Contextualización opcional: [.agents/skills/project-discovery/SKILL.md](.agents/skills/project-discovery/SKILL.md).
- Refinamiento PO: [.agents/skills/refine-user-story/SKILL.md](.agents/skills/refine-user-story/SKILL.md).
- Preparación QA: [.agents/skills/prepare-tests/SKILL.md](.agents/skills/prepare-tests/SKILL.md).

Lee la skill correspondiente al realizar esa tarea, aunque el cliente no la
descubra automáticamente. Resolver un store no instala sus skills ni carga sus
instrucciones en una sesión iniciada en otro repositorio. Si no puedes leer la
fuente requerida, identifica el acceso faltante sin inventar su contenido.

Consultar o preparar casos no escribe ni ejecuta pruebas por defecto. Respeta
las acciones ya autorizadas por el usuario y sus destinos; Jira es opcional.
No atribuyas aprobaciones humanas ni ejecuciones a archivos o casos preparados.
Redacta en español y conserva trazabilidad con fuentes y revisiones identificables.

## Mantenimiento de la plantilla

Usa inglés para nombres de carpetas, archivos propios, skills y claves de
inventario. Conserva los nombres oficiales de OpenSpec. El idioma de los nombres
técnicos no cambia la redacción en español de instrucciones y resultados.

Mantén las reglas comunes en este archivo y los métodos propios sólo en sus skills
canónicas de `.agents/skills/`. `CLAUDE.md` y las entradas propias de `.claude/`
son puentes a esas fuentes. No necesitan enlaces simbólicos ni scripts de copia.
Project Discovery reúne su método completo y el verificador de `scripts/` en
`.agents/skills/project-discovery/`; Claude utiliza un puente a esa misma fuente.
Es independiente y no es un paso obligatorio de OpenSpec;
documentar contexto no autoriza cambiar configuración ni artefactos OpenSpec.
Los workflows `openspec-*` son generados por OpenSpec; actualízalos con la CLI
acordada, sin editarlos a mano. Valida los artefactos con
`openspec validate --all --strict --no-interactive`; eso no ejecuta tests de producto.

Las instancias Git son independientes del maestro. Incorpora sus actualizaciones
de forma manual, revisando el diff y conservando identidad del store, contexto,
inventario propio, specs e historial. No propagues cambios automáticamente entre
la plantilla, sus instancias u otros S0.
