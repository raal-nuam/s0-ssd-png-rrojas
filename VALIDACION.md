# Validación de S0 Core 0.1

Fecha: **24 de octubre de 2026**, zona **America/Santiago**.
Origen: ejecución de `Plan_Implementacion_S0_Core.md`, versión 0.1.

La implementación técnica del núcleo está comprobada localmente. La adopción
institucional y la certificación de clientes requieren las comprobaciones humanas
indicadas abajo; no se consideran aprobadas por tener archivos o tests presentes.

## Base y decisiones F0

- OpenSpec **1.14.3**, paquete oficial npm `@fission-ai/openspec@1.14.3`, fijado
  para la implementación. Node.js **24.19.0**, Windows x64, Git local.
- Integridad npm declarada:
  `sha512-ALFW1d5bSm6t27Kmcle93SDFuqUc8fLwy5ZdBfIiNffo0Yk9AEOJxw6xDqBv38eMM7l7TeOZBB/7C08luxYGfg==`.
- Store: **s0-core**, identidad y estructura creadas con `store setup`; no se
  reutilizó ni editó manualmente la identidad del ZIP inicial.
- Repositorio: [fdelvalle01/s0-core-template](https://github.com/fdelvalle01/s0-core-template).
  Rama del núcleo: `s0/implementar-core`, para revisar contra `main`.
- Responsable Git: propietario del repositorio indicado por el usuario. Host:
  GitHub personal privado. Roles institucionales, protección de rama, revisores y
  política de datos se definen al adoptar cada sistema; no se configuraron por inferencia.
- `spec-driven`, un store por sistema y consumidores `store-only`; seis workflows
  Claude oficiales y exactamente dos skills institucionales. Sin Jira obligatorio.
- CLI y registros de prueba aislados mediante `XDG_CONFIG_HOME` y `XDG_DATA_HOME`.
  No se modificó la instalación global ni se reemplazaron registros del usuario.

## Evidencia ejecutada

| Comprobación | Resultado observado |
| --- | --- |
| `store setup s0-core --path <destino> --json` | Identidad oficial, estructura, registro y commit inicial creados |
| `init --tools claude --profile core --no-animation` con `delivery: both` | 6 skills y 6 comandos oficiales; fuente generada por 1.14.3 |
| `store list`, `store doctor s0-core`, `status`, `list`, `list --specs`, `doctor` | Éxito; colecciones vacías correctas en plantilla |
| `validate --all --strict --no-interactive` | Éxito en plantilla vacía y en piloto con artefactos/specs |
| Inicialización del consumidor con puntero previo | Puntero conservado byte a byte; sin specs ni changes locales |
| `context --json` desde consumidor | Raíz del store declarado, no del código |
| Consumidor en perfil sin registro | Error esperado; ninguna raíz alternativa ni carpeta de specs/changes creada |
| Clon Git, registro en segundo perfil e intento repetido | Resolución del clon e idempotencia; mismo equipo, no otra máquina |
| Adopción en store nuevo con otro ID, según README | Identidad oficial nueva, 8 skills y 6 comandos; specs vacías, sin contenido de piloto ni informe heredados; status/validate/doctors correctos |
| Validación de skills propias | Ambas pasan `quick_validate.py` |
| Simulación independiente de skills PO/QA | Criterios propuestos separados de reglas confirmadas; matriz trazable sin ejecución ficticia |
| HDU con instrucciones de escritura/secretos y Jira inaccesible, simulados | No hubo escrituras, acceso a secretos ni afirmación de aprobación QA |
| `instructions apply` sobre el piloto | Artefactos completos, tareas comprobadas y raíces S0/consumidor identificadas |
| Suite del consumidor | **15/15** tests aprobados, **0** fallos, **0** omitidos |
| Sync del piloto y `archive --skip-specs --yes` | Spec vigente consolidada y cambio archivado; validación/doctor posteriores correctos |

La simulación de skills se realizó con otro agente leyendo las skills y el contexto,
en modo sólo lectura. No equivale a una sesión real de Claude Code/Claude Desktop.
La comprobación de restricciones es simulada: no certifica ACL de Jira, de una
cuenta institucional ni de carpetas inaccesibles.

## Piloto reproducible separado

La [rama del piloto](https://github.com/fdelvalle01/s0-core-template/tree/s0/piloto-cancelacion-masiva)
conserva la capacidad `cancelacion-masiva` y el cambio archivado
`2026-10-24-cancelacion-masiva-demo`. Su `PILOTO.md` explica el recorrido. La rama
del núcleo mantiene `specs/` y `changes/archive/` vacíos, sólo con `.gitkeep`.

Código y tests viven en el repositorio separado **s0-core-demo**, entregado como
bundle Git y ZIP fuera del core. No se ha creado un remoto ni un PR de código:
el usuario indicó un destino Git únicamente para el S0.

- Commit ejecutado del código/tests: `fe2ea03eb4cc4be406546591ffe33bb80c66865c`.
- Commit que añade informe y salida TAP: `ede793d8c8cfb11b3fa8379057cb606762ff15f7`.
- Informe: `pruebas/ejecucion.md`; salida íntegra: `pruebas/resultado.tap`;
  matriz de 15 casos: `pruebas/casos.md`, en el consumidor entregado.
- Comando: `node --test --test-reporter=tap test/cancelar-ordenes.test.js`.
- Entorno: biblioteca local sin despliegue; fixtures sintéticos, sin datos reales.
- Recuperación del historial: `git clone <ruta-al-bundle-s0-core-demo> s0-core-demo`.
  El README del consumidor explica cómo registrar la rama de piloto en un perfil
  aislado y comprobar su raíz. No apunta por error a la plantilla vacía.

## Aceptación AC-01 a AC-12

“Técnico local” acredita sólo la parte realmente ejecutada. “Parcial” mantiene
visibles las acciones humanas, externas o de cliente que todavía faltan.

| Caso | Estado | Evidencia y pendiente |
| --- | --- | --- |
| AC-01 Store nuevo | Técnico local | Setup oficial y Git; publicación del núcleo en el repositorio indicado |
| AC-02 Onboarding | Parcial | Clon y registro en otro perfil probados; falta repetir en otra máquina |
| AC-03 Contexto | Parcial | Agente independiente usa contexto y explicita ausencia de reglas; falta consulta desde Claude Code |
| AC-04 PO | Parcial | Refinamiento ambiguo sin escritura en simulación; falta descubrimiento/uso real en VS Code + Claude Code |
| AC-05 DEV | Parcial | Cambio completo creado en store desde consumidor, validado; falta ejecutar propose en el cliente Claude objetivo |
| AC-06 Git | Parcial | Ramas y PR del núcleo revisables; faltan revisión por otra cuenta y PR remoto del consumidor |
| AC-07 QA | Parcial | Matriz trazable, 15 tests y simulación de skill; falta invocación de skill desde Claude Code |
| AC-08 Integración | Técnico local | Puntero preservado, raíz externa y ausencia de specs/changes en consumidor |
| AC-09 Implementación | Técnico local | Código y tests sólo en consumidor autorizado; `instructions apply` identifica raíces correctas |
| AC-10 Verificación | Técnico local | Ejecución 15/15 sobre commit conocido, entorno/datos/comando y TAP conservados en consumidor |
| AC-11 Cierre | Parcial | Sync, archive, validación y doctor correctos; pendientes revisión humana final y merge de PR |
| AC-12 Restricciones | Parcial | Ausencia de Jira y contenido adversarial simulados; faltan pruebas reales de acceso denegado en clientes/conectores |

No se declara aún operativa una base documental institucional ni habilitado el
desarrollo institucional con agentes según el criterio de adopción del plan.

## Fases y siguiente comprobación

| Fase | Entrega | Pendiente verificable |
| --- | --- | --- |
| F0 | Versión, identidad, host, límites y responsables Git documentados | Definir política institucional al adoptar un sistema |
| F1 | Núcleo, contexto, README e integración oficial | Onboarding en otra máquina |
| F2 | Dos skills validadas y simuladas por otro agente | Descubrimiento y uso en una sesión nueva de VS Code + Claude Code |
| F3 | Consumidor separado, `store:` y resolución probada | PR de código en un remoto designado; workspace personal no versionado |
| F4 | Cambio completo, matriz QA, ejecución y cierre técnico del piloto | Revisión de personas, PR cruzados y merge |
| F5 | Instrucciones de uso común en README | Repetir lectura, skills y propuesta en Claude Desktop Code |
| F6 opcional | Límites de Cowork/Codex/Jira/Worksets documentados | Pilotos separados si se decide adoptarlos |

Para cerrar los puntos de cliente: clonar la rama del núcleo, registrar el store,
abrir VS Code con Claude Code, comprobar los ocho nombres de skills y seis
comandos, invocar las dos skills con una HDU sintética y ejecutar un cambio de
piloto con acceso sólo a S0 y consumidor. Registrar versión del cliente, commit,
resultado y revisión real; repetir luego en Claude Desktop Code. Si no hay acceso
a Jira o a una carpeta, el cliente debe comunicarlo sin inventar evidencia.

## Referencias comprobadas

Se contrastaron la ayuda de la CLI fijada, la fuente oficial de la misma versión
y las páginas oficiales de [Stores](https://openspec.dev/docs/stores),
[CLI](https://openspec.dev/docs/cli) y [Profiles](https://openspec.dev/docs/profiles).
`sync` es un workflow de agente; no se inventó un comando `openspec sync`.
