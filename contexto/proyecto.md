# S0 Core — contexto de la plantilla

## Propósito y alcance

Esta plantilla permite compartir por Git el contexto, las especificaciones y los
cambios OpenSpec de un sistema. PO refina necesidades; DEV propone e implementa
en repositorios de código; QA deriva casos y verifica versiones identificadas.
El núcleo usa `spec-driven`, sin dependencia de Jira ni infraestructura adicional.
Las instrucciones comunes viven en `AGENTS.md` y los métodos PO/QA en
`.agents/skills/`. Codex y Claude consumen esas mismas fuentes; el modelo se
selecciona en cada cliente, sin cambiar el contexto ni las specs del sistema.

Al adoptar la plantilla se sustituye esta página por el contexto confirmado del
sistema: propósito, usuarios, alcance, componentes, vocabulario y fuentes. No hay
componentes de producto ni reglas de negocio institucionales predefinidos.

## Componentes y fuentes

- Repositorio de la plantilla: <https://github.com/fdelvalle01/s0-core-template>.
- Requisitos vigentes: [openspec/specs/](../openspec/specs/), vacíos en la plantilla.
- Propuestas e historial: [openspec/changes/](../openspec/changes/).
- Necesidades de origen: texto aportado por el usuario, issue o HDU; Jira opcional.
- Código, contratos y tests: los repositorios consumidores que identifique cada
  sistema. Sus URLs Git se añaden aquí al adoptar la plantilla, sin rutas personales.

## Vocabulario y límites

- **HDU:** historia o necesidad de usuario; puede recibirse como texto.
- **Store:** repositorio independiente con contexto, specs y cambios compartidos.
- **Consumidor:** repositorio de código con `store:` y sin specs/changes locales.
- **Caso preparado / test implementado / test ejecutado:** estados diferentes de QA.

Consultar no autoriza escribir. Las conexiones y el acceso a código dependen de
los permisos de la sesión. Los secretos y las configuraciones personales no se
versionan. Esta página referencia las specs; no las duplica.
