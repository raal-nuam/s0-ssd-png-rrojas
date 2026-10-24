# S0 Core

Plantilla mínima para compartir contexto, especificaciones y cambios de un sistema
por Git. Usa OpenSpec nativo `spec-driven`. El código y los tests viven en los
repositorios de producto; PO, DEV y QA consultan las mismas fuentes.

El núcleo no fija un proveedor ni un modelo. Incluye integraciones de Codex y
Claude Code para aplicar los mismos métodos y reglas; otros agentes pueden
consumir los archivos comunes cuando tengan acceso a ellos.

Versión inicial: **0.1**. CLI comprobada: **OpenSpec 1.14.3**, con Node.js
**>=20.19.0** para la distribución npm. Stores sigue siendo beta: actualiza la
versión sólo después de repetir la validación. No necesitas Node si utilizas el
binario oficial equivalente.

## Empezar en 10 minutos

Requisitos: Git, OpenSpec 1.14.3 y un agente con acceso a la carpeta para utilizar
los workflows. Puedes iniciar Codex o Claude Code desde la raíz S0. Instalación
npm fijada si la política del equipo lo permite:

```powershell
npm install -g @fission-ai/openspec@1.14.3
openspec --version
git clone https://github.com/fdelvalle01/s0-core-template.git s0-core
cd s0-core
openspec store register .
openspec store list
openspec status --store s0-core
openspec store doctor s0-core
code .
```

El registro es local a tu máquina; no sincroniza Git. Actualiza el clon con
`git pull` antes de trabajar y comprueba rama/commit. Un ID sólo puede resolver
un checkout por registro: si ya usas `s0-core`, revisa `store list` antes de
registrar otro clon. No reemplaces un registro existente sin saber qué usa.

El repositorio incluye los seis workflows oficiales para ambos clientes y dos
skills institucionales con una única fuente funcional. Las instrucciones comunes
están en [AGENTS.md](AGENTS.md); `CLAUDE.md` importa ese archivo. Las propias se
mantienen en `.agents/skills/`; sus entradas de `.claude/skills/` sólo indican
cómo leer esa misma fuente.

Abre una sesión nueva en la raíz S0 y comprueba las entradas del cliente elegido:

| Tarea | Codex | Claude Code |
| --- | --- | --- |
| Explorar | `$openspec-explore` | `/opsx:explore` |
| Proponer | `$openspec-propose` | `/opsx:propose` |
| Implementar | `$openspec-apply-change` | `/opsx:apply` |
| Actualizar el plan | `$openspec-update-change` | `/opsx:update` |
| Sincronizar specs | `$openspec-sync-specs` | `/opsx:sync` |
| Archivar | `$openspec-archive-change` | `/opsx:archive` |
| Refinar HDU | `$refinar-hdu` | `/refinar-hdu` |
| Preparar casos QA | `$preparar-pruebas` | `/preparar-pruebas` |

La tabla muestra las formas de invocación previstas por las integraciones.
Tener archivos instalados y comprobar su descubrimiento efectivo son pruebas
distintas; el resultado observado por cliente se registra en [VALIDACION.md](VALIDACION.md).

También puedes pedir el trabajo en lenguaje natural:

```text
Usa refinar-hdu con esta necesidad: el operador debe cancelar varias órdenes.
Usa preparar-pruebas sobre los criterios que acabamos de acordar.
```

`propose` prepara artefactos; `apply` implementa cuando lo solicitas. Las skills
institucionales devuelven su resultado en conversación por defecto: refinar no
crea cambios/tickets y preparar casos no ejecuta tests ni acredita aprobación QA.
Una HDU puede ser texto; Jira requiere una conexión autorizada y es opcional.

## Contenido y responsabilidades

```text
.openspec-store/store.yaml         Identidad creada por la CLI oficial
openspec/config.yaml              Schema, contexto conciso y reglas
openspec/specs/.gitkeep            Requisitos vigentes; vacío en esta plantilla
openspec/changes/archive/.gitkeep  Cambios e historial; vacío al comenzar
contexto/proyecto.md              Contexto confirmado del sistema
AGENTS.md                        Instrucciones comunes para agentes
CLAUDE.md                        Puente que importa AGENTS.md
.agents/skills/openspec-*/        Seis skills oficiales para Codex
.agents/skills/refinar-hdu/       Fuente común de refinamiento PO
.agents/skills/preparar-pruebas/  Fuente común de preparación QA
.claude/commands/opsx/            Seis comandos oficiales generados
.claude/skills/openspec-*/        Seis skills oficiales para Claude
.claude/skills/refinar-hdu/       Puente a la fuente común de PO
.claude/skills/preparar-pruebas/  Puente a la fuente común de QA
VALIDACION.md                     Evidencia y pendientes del piloto de la plantilla
```

Mantén una página de contexto con propósito, usuarios, alcance, vocabulario,
componentes y enlaces Git. `config.yaml` referencia esa página; el agente debe
leerla cuando corresponda, no se incrusta automáticamente. Las specs describen
comportamientos observables. No se exigen `jira.md`, `revision.md`, `qa.md`, schemas
personalizados, `.base/`, inventarios, actualizadores ni un visor.

## Crear un S0 para otro sistema

No cambies a mano `.openspec-store/store.yaml` ni publiques todos los sistemas con
el ID de la plantilla. Crea una instancia con identidad propia mediante la CLI.
La secuencia siguiente parte de la raíz del clon de esta plantilla y deja ambas
carpetas como hermanas. Sustituye el ID y el nombre ficticios por los del sistema;
el directorio de destino debe ser nuevo:

```powershell
$s0Template = (Get-Location).Path
$s0Id = 'mi-sistema'
$s0Destino = Join-Path (Split-Path -Parent $s0Template) 's0-mi-sistema'
Set-Location (Split-Path -Parent $s0Template)
openspec store setup $s0Id --path $s0Destino
Set-Location $s0Template
```

`store setup` crea la estructura oficial, registra el store e inicializa Git con
un commit; no vuelvas a ejecutar `git init`. Desde el clon de esta plantilla,
copia las instrucciones comunes, las dos fuentes propias y sus puentes:

```powershell
Copy-Item -LiteralPath AGENTS.md,CLAUDE.md,.gitignore,.gitattributes,README.md -Destination $s0Destino
Copy-Item -LiteralPath contexto -Destination $s0Destino -Recurse
Copy-Item -LiteralPath openspec\config.yaml -Destination (Join-Path $s0Destino 'openspec\config.yaml')
$s0Canonicas = Join-Path $s0Destino '.agents\skills'
$s0Puentes = Join-Path $s0Destino '.claude\skills'
New-Item -ItemType Directory -Force -Path $s0Canonicas,$s0Puentes | Out-Null
Copy-Item -LiteralPath .agents\skills\refinar-hdu,.agents\skills\preparar-pruebas -Destination $s0Canonicas -Recurse
Copy-Item -LiteralPath .claude\skills\refinar-hdu,.claude\skills\preparar-pruebas -Destination $s0Puentes -Recurse
Set-Location $s0Destino
openspec config get delivery
```

La generación oficial se hace en la raíz del nuevo S0. `--profile core` elige los
seis workflows; la preferencia local `delivery` decide si se generan skills,
comandos o ambos. La entrega de esta plantilla usa `both`. Si tu preferencia es
distinta y quieres reproducirla, ejecuta `openspec config set delivery both`
antes de inicializar. Ese ajuste cambia la preferencia de la máquina; no modifica
el store ni los permisos del agente.

```powershell
openspec init --tools claude,codex --profile core --no-animation
openspec status --store $s0Id
openspec store doctor $s0Id
```

Sustituye el contexto y las referencias a la plantilla en este README por datos
confirmados del sistema: ID, URLs Git, componentes y fuentes. Actualiza también
los enlaces a la validación de la plantilla para enlazar tu propia evidencia o
retirarlos si no aplican. No copies `.git/`, `.openspec-store/`, `VALIDACION.md`,
specs del piloto ni estado local. Las carpetas `specs/` y `changes/archive/`
comienzan vacías. No hace falta copiar todos los directorios de agente: las
oficiales se generan y las propias conservan su estructura relativa sin symlinks.
Conecta después el remoto **del sistema**, publica mediante ramas/PR y configura
la protección de rama según la política del host Git.

## Conectar un repositorio de código

Registra primero el S0 en la máquina. En la raíz Git del consumidor, crea
`openspec/config.yaml` con un único puntero (usa el ID real de tu sistema):

```yaml
store: mi-sistema
```

Luego, siempre desde esa raíz:

```powershell
openspec init --tools claude,codex --profile core --no-animation
openspec context --json
openspec status
```

El inicializador conserva el puntero y sólo añade integraciones al consumidor.
La misma preferencia local `delivery` se aplica aquí; compruébala si necesitas
skills y comandos. No crees `openspec/specs/` ni `openspec/changes/` allí: tienen
precedencia sobre el puntero. Confirma que `context` resuelve la raíz del S0.

El puntero no instala las skills propias del S0 ni carga sus instrucciones en
una sesión iniciada en el código. Para PO y QA inicia la sesión en la raíz S0.
Si trabajas desde el consumidor, indica al agente que lea `AGENTS.md`, el contexto
y la skill concreta de la raíz resuelta; comprueba que puede acceder a ellos.
No presupongas que aparecen en el selector del cliente por registrar el store.

Para implementar, abre S0 y el código con acceso autorizado a ambas raíces.
Un workspace puede hacerlas visibles, pero no concede permisos. Guarda los
workspaces con rutas personales fuera del repositorio compartido. Worksets es
opcional.

## Un cambio de PO a QA

1. **PO:** aporta la HDU en texto o mediante una fuente accesible. `refinar-hdu`
   consulta contexto/specs y devuelve objetivo, alcance, criterios y dudas.
2. **DEV:** explora y propone. OpenSpec crea `proposal.md`, deltas en `specs/`,
   `design.md` y `tasks.md` en `openspec/changes/<cambio>/` del S0.
3. **Equipo:** revisa el plan en una rama `s0/<hdu-o-slug>` y PR. La revisión de
   documentos no acredita aceptación del software; la CLI no impone aprobadores.
4. **DEV:** solicita `apply` con código accesible y autorizado. Implementación y
   tests permanecen en el repositorio consumidor, con su PR enlazado al S0.
5. **QA:** `preparar-pruebas` deriva casos trazables. Automatizar y ejecutar es
   trabajo explícito en la suite, identificando commit/build, entorno y datos.
6. **Equipo:** revisa resultados y alcance. El workflow `sync` consolida specs
   y `archive` cierra el cambio; usa la invocación del cliente indicada en la
   tabla. La CLI `archive` también puede consolidar specs.

Usa el PR para enlazar HDU o solicitud, ID OpenSpec, PR de código y resultados de
pruebas sobre una revisión identificable. Mientras una propuesta está en una rama,
los revisores deben usar esa rama o su diff; `main` conserva lo consolidado antes.
Protección de rama, aprobaciones y merge se configuran en Git, no por estas skills.

En QA distingue **caso preparado**, **test implementado** y **test ejecutado sobre
una versión**. Mantén fallos y pendientes visibles; no marques como ejecutado lo
que sólo es una matriz o un archivo de test.

## Comprobaciones y mantenimiento

Estos comandos usan el ID de la plantilla; en una instancia utiliza su ID propio:

```powershell
openspec store list
openspec store doctor s0-core
openspec list --store s0-core
openspec list --specs --store s0-core
openspec status --store s0-core
openspec validate --all --strict --no-interactive --store s0-core
openspec doctor --store s0-core
```

Sin cambios/specs, los listados vacíos son correctos. La validación estructural no
sustituye tests de producto. No existe un comando de terminal `openspec sync`:
sync es un workflow del agente.

Las skills oficiales se mantienen con la CLI de la versión acordada. Revisa
primero `openspec update --dry-run`; al adoptar otra versión, vuelve a probar
registro, resolución store-only, generación para ambos clientes y ciclo de
cambio. No edites a mano los workflows generados.

Las instrucciones compartidas se editan en `AGENTS.md`; los métodos PO/QA se
editan sólo en sus dos fuentes de `.agents/skills/`. Mantén `CLAUDE.md` y los
wrappers propios como puentes, sin duplicar reglas. Revisa estos cambios por PR
y comprueba que los enlaces relativos resuelven después de clonar o adoptar.
Al actualizar OpenSpec, verifica también que conserva las fuentes y los puentes.

## Clientes y alcance de la certificación

Codex y Claude Code cuentan con integraciones de proyecto equivalentes y
consumen las mismas fuentes institucionales. Cada cliente necesita comprobar
descubrimiento y uso en una sesión nueva, con su versión y el commit del S0.
Claude Desktop Code requiere su propia prueba. La estructura no garantiza
resultados idénticos entre modelos ni concede acceso a carpetas o conectores.

Con otro agente, pide que lea `AGENTS.md`, `contexto/proyecto.md` y la skill
concreta de `.agents/skills/`. Si OpenSpec ofrece una integración oficial para
ese cliente, genérala con la CLI acordada y compruébala. La lectura explícita de
archivos no acredita descubrimiento automático; evita prometerlo universalmente.

En Cowork/Chat no se supone carga automática de las carpetas del repositorio:
empaquetado y habilitación son un piloto separado. Jira y Worksets siguen siendo
opcionales.

El [registro de validación](VALIDACION.md) distingue pruebas locales ejecutadas de
revisión humana, permisos reales y clientes pendientes. Un piloto local no implica
certificación institucional completa.

## Referencias oficiales

- [Stores y resolución store-only](https://openspec.dev/docs/stores).
- [CLI y flags soportados](https://openspec.dev/docs/cli).
- [Perfiles y seis workflows core](https://openspec.dev/docs/profiles).
- [Configuración de proyecto](https://openspec.dev/docs/configuration/config-yaml).
- [OpenAI Docs: skills de Codex y rutas de proyecto](https://learn.chatgpt.com/docs/build-skills).
- [Claude Code: instrucciones e importación de archivos](https://code.claude.com/docs/en/memory).
- [Claude Code: skills de proyecto e invocación](https://code.claude.com/docs/en/skills).
