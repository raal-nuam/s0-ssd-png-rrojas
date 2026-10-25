# Validación del core

Referencia para mantener la plantilla: qué se comprobó, qué sigue pendiente y
cómo repetir las verificaciones. La instalación y el uso están en
[INSTALLATION.md](INSTALLATION.md) y [PLAYBOOK.md](PLAYBOOK.md).

## Estado y fuentes de evidencia

Última ejecución: **2026-10-25**, descrita abajo. Los siguientes informes son
antecedentes de las entregas correspondientes, no resultados de la revisión actual:

| Informe | Publicación¹ | Revisión de entrega | Entorno declarado |
| --- | --- | --- | --- |
| [PR #6: organización de Discovery](https://github.com/fdelvalle01/s0-core-template/pull/6) | 2026-10-24 | [2b552cc](https://github.com/fdelvalle01/s0-core-template/commit/2b552cc819d031ca67f8710c85adb3560ae18105) | Windows; verificador Git/Python y comprobaciones estructurales |
| [PR #7: connect-project](https://github.com/fdelvalle01/s0-core-template/pull/7) | 2026-10-24 | [6d58fbd](https://github.com/fdelvalle01/s0-core-template/commit/6d58fbd88d79b6557e7b95b6212ed377dfcbd771) | OpenSpec 1.14.3; repositorios sintéticos, registros y preferencias aislados; catálogo Codex |

¹ Fechas de publicación en America/Santiago, no fechas exactas de ejecución.
Los PR contienen resúmenes de resultados, no logs completos. Los registros brutos
y fixtures se conservaron fuera de la plantilla y no están enlazados desde esos
informes. Tampoco se identifica allí el modelo ni todas las versiones de cliente.
Son antecedentes de esas entregas, no una certificación de cualquier revisión futura.

| Área | Resultado documentado | Fuente |
| --- | --- | --- |
| Verificador Discovery | 18 tests: 17 aprobados y 1 omitido por privilegios Windows para crear symlinks | PR #6 |
| Conexión local | S0 nuevo, monorepo y dos Git adicionales; resolución e integraciones correctas; borrador Discovery sin escritura | PR #7 |
| Reejecución y conflictos | Sin cambios innecesarios en archivos/estado Git/preferencias; planificación previa y asociaciones de registro conservadas | PR #7 |
| CLI y preferencias | Instalación real en prefijo npm desechable; selección de 1.14.3 frente a wrapper anterior simulado; herramientas ausentes simuladas; generación temporal con `delivery=both` | PR #7 |
| Catálogo e integraciones | Codex: 10 skills habilitadas, 0 errores, sin iniciar un turno de modelo; archivos oficiales del core conservados | PR #7 |
| Recorridos completos sin código y sin Python | **Pendientes**; los controles parciales no acreditan estos recorridos | PR #7 |
| Sesión real Claude Code y regresión semántica dual | **Pendientes**; catálogo, hashes y estructura no acreditan ejecución de los métodos | PR #6 / PR #7 |

En futuras validaciones registra fecha de ejecución, commit probado, sistema,
CLI, cliente/modelo cuando intervengan, resultado y enlace al informe o logs
accesibles al equipo. Conserva fallos y omisiones; no conviertas un caso omitido
en aprobado ni sobrescribas antecedentes con resultados de otro entorno.

### Fuentes locales, remotas y mixtas — 2026-10-25

Base: `2e9fd3dbeeca8e047ceedc04e84be0a4d22f2eb6`, más esta ampliación.
Fuente final: blob Git `d91dd0c7fe30d0ece7a0ee2a57a57a55b0f9ecb0`.
Las primeras conexiones y el caso sin código usaron candidatos intermedios;
actualización, recuperación y reejecución mixta se probaron sobre esta fuente final.
Entorno: Windows, Git **2.46.2.windows.1**, OpenSpec **1.14.3**, Python **3.13.3**.
Los agentes de esta sesión ejecutaron los procedimientos en repositorios sintéticos,
con registro y preferencias aislados. Un agente independiente recibió una petición
mixta y las skills locales sin adelantarle los resultados esperados. No se midieron
versión del cliente ni modelo; no fue una sesión real de Claude Code.

| Escenario ejecutado | Resultado |
| --- | --- |
| Conexión mixta: dos remotos, monorepo y otros Git | Cuatro consumidores conectados al S0 correcto; una raíz con planificación propia bloqueada e intacta; nombres de repositorio repetidos sin colisión |
| Sólo remoto; ramas explícita y predeterminada | Copias distintas para dos ramas del mismo URL; rama `trunk` resuelta sin asumir `main`; integraciones oficiales comprobadas |
| Actualización remota y desde una ruta local con upstream | Nuevas copias conectadas; anteriores preservadas en archivos, HEAD, rama, índice y configuración, incluido trabajo staged/unstaged |
| Actualización sin avance y reconexión | Sin otro clon, regeneración o entrada duplicada; reconexión sin consultar nuevas revisiones. Reejecución mixta: seis raíces, índice externo, registro y preferencias sin diferencias |
| Dos S0 en el mismo workspace | Mismo URL/rama/SHA con copias e índices independientes; consumidores resuelven su propio S0 |
| Conflicto de registro, rama ausente y remoto inaccesible | Bloqueos explícitos sin sustituir asociaciones ni descargas existentes |
| Recuperación e índice: nueve secuencias Git instrumentadas | Descargas pendientes recuperables; copias alteradas bloqueadas al reconectar; históricas conservadas; índices ajenos/inválidos y destinos dentro del S0 o con junction externa rechazados |
| Herramientas y preferencias | CLI real 1.14.3 seleccionada detrás de 1.13.2; ausencias simuladas con PATH aislado; preferencias persistentes conservadas |
| Sin código y entrega Discovery | Identidad preparada sin workspace/índice; borrador limitado al propósito. Snapshots posteriores a preparación, verificaciones de lectura sin violaciones y contexto sin escribir |
| Regresión del core | 37 archivos protegidos sin cambios; schema oficial sin overrides; 56 enlaces locales válidos; fuente común y puente válidos; sin artefactos de producto que validar |
| Suite existente de Discovery | 18 tests: 17 aprobados y 1 omitido por privilegios Windows para symlinks |

Los remotos HTTPS se redirigieron a Git bare locales. Esto prueba preparación Git,
no autenticación real de GitHub, GitLab, Bitbucket o SSH. No se instalaron runtimes
ni se ejecutaron productos. Los commits/push de preparación de fixtures afectaron
sólo remotos sintéticos; las operaciones de conexión no publicaron cambios.

Se conservaron los fallos de ensayo: escrituras CRLF del laboratorio requirieron
corregir únicamente las adiciones propias. Un `diff --check` de consumidor sigue
señalando whitespace preexistente, cuyos bytes e índice se preservaron; no se
presenta como verde. En las pruebas de recuperación se detectaron claves de editor
`vscode-merge-base` añadidas a Git; la repetición con una referencia estable pasó
sin excluir metadata del control. Los informes y registros brutos permanecen fuera del core,
en el laboratorio de esta entrega, no como nuevos archivos de la plantilla.
La ejecución real en Claude, la regresión semántica dual y el recorrido completo
sin Python siguen pendientes. Las ausencias simuladas no equivalen a probar
una máquina sin esas herramientas instaladas.

## Comprobaciones reproducibles

Trabaja en copias aisladas con datos sintéticos. Usa Git y OpenSpec **1.14.3**;
para Discovery, Python **>=3.10** y su biblioteca estándar. Comprueba primero el
ejecutable efectivo según [la referencia de CLI](INSTALLATION.md#cli-setup).

### Estructura y resolución

Desde la raíz de la copia de prueba, usando la CLI comprobada:

```powershell
openspec --version
openspec schema which spec-driven --json
openspec status
openspec validate --all --strict --no-interactive
openspec doctor
git diff --check
```

El schema efectivo debe ser el oficial del paquete, sin overrides. La plantilla
sin requisitos o cambios puede devolver inventarios vacíos. Comprueba enlaces,
fuentes propias y puentes Claude; verifica catálogo y ejecución por separado.

Para repetir la conexión, sigue estos escenarios con `connect-project`:

| Escenario | Comprobar |
| --- | --- |
| Instancia nueva, monorepo y varios Git | ID propio; `context --json` desde cada consumidor resuelve el mismo S0; sin specs/changes locales paralelos |
| Segunda ejecución | Identidad, punteros, instrucciones, HEAD, índice y preferencias conservados; sin generación redundante |
| Registro en conflicto o planificación previa | Bloqueo explícito, contenido y asociaciones preservados, resultados parciales identificados |
| CLI ausente, PATH antiguo o requisito ausente | Instalación/selección de versión según herramientas disponibles; requisito pendiente explícito; preferencias persistentes conservadas |
| Sin código o sin Python | Preparación posible y borrador limitado; sin arquitectura inventada ni escritura de contexto sin comprobaciones |
| Remoto o mixto, rama omitida y varias ramas | SHA realmente preparado; selección predeterminada correcta; copias sin colisiones ni conexiones por inferencia |
| Reejecución o actualización explícita | Reutilización sin avance implícito; nueva copia cuando corresponda y anterior intacta, también al actualizar desde una ruta local |
| Índice, descargas pendientes y varios S0 | Propietario y rutas comprobados; recuperación sin borrar copias; discrepancias de una copia verificada bloquean reconexión |

El cierre deja diffs locales sin stage, commit, push ni PR. Captura la referencia
de Discovery después de la preparación técnica y verifica lectura sin cambios
antes de presentar su borrador.

### Verificador y comportamiento de Discovery

Desde la raíz, sin generar caches Python en la plantilla:

```powershell
py -3 -B -m unittest discover -s .agents/skills/project-discovery/scripts -p test_project_discovery_scope.py -v
```

La suite cubre cambios previos, rutas permitidas exactas, archivos protegidos,
HEAD/índice, permisos y enlaces. Aunque usa carpetas temporales, hereda la
configuración y variables Git del entorno: hooks, firma o plantillas personales
pueden afectar su reproducibilidad. Ese aislamiento sigue pendiente de mejorar.

Para comprobar el método completo, usa dos o tres Git sintéticos con documentación
contradictoria y una dependencia desconocida. Pide un borrador a un agente sin
adelantar conclusiones; revisa fuentes, hechos/inferencias y cobertura. Comprueba
lectura sin cambios y, tras aprobar contenido y rutas, publicación limitada a ese
contexto contra la misma referencia. El [contrato de Discovery](.agents/skills/project-discovery/SKILL.md#scope-checks)
define las operaciones y límites exactos.

### Regresión funcional en Codex y Claude

En copias equivalentes, fija revisión, CLI, schema, cliente, modelo y necesidad;
aísla registros y preferencias. Conserva hashes de configuración, identidad y
archivos oficiales para comprobar que Discovery no los modifica.
Compara una propuesta nativa antes y después de Discovery aprobado: conserva los
artefactos, valida con la CLI y revisa requisitos observables, decisiones de diseño
y tareas comprobables. Repite en cada cliente sin sustituirlo por el otro.
No se espera identidad textual; registra diferencias de significado, fallos y
fuentes. Este recorrido sigue pendiente según la tabla de estado.

## Límites de lo comprobado

El verificador detecta diferencias de archivos y estado Git; no acredita aprobación
humana, veracidad del texto ni escrituras transitorias revertidas. El análisis puede
ser selectivo, pero las huellas cubren los archivos incluidos de cada raíz capturada:
la arquitectura multirrepositorio no equivale a un benchmark de rendimiento.
Validación estructural, archivo de cambios y catálogo de skills no prueban aceptación
funcional, ejecución del producto ni permisos de otros clientes o conectores.
