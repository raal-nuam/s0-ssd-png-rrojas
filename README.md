# S0 Core

Plantilla mínima para compartir contexto, especificaciones y cambios de un sistema
por Git. Usa OpenSpec nativo `spec-driven`. El código y los tests viven en los
repositorios de producto; PO, DEV y QA consultan las mismas fuentes.

Versión inicial: **0.1**. CLI comprobada: **OpenSpec 1.14.3**, con Node.js
**>=20.19.0** para la distribución npm. Stores sigue siendo beta: actualiza la
versión sólo después de repetir la validación. No necesitas Node si utilizas el
binario oficial equivalente.

## Empezar en 10 minutos

Requisitos: Git, OpenSpec 1.14.3 y, para usar las skills, Claude Code con acceso a
la carpeta. Instalación npm fijada si la política del equipo lo permite:

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

El repositorio incluye seis workflows oficiales generados por OpenSpec y dos
skills propias. Abre una sesión nueva de Claude Code y confirma que aparecen
`/opsx:explore`, `/opsx:propose`, `/opsx:apply`, `/opsx:update`, `/opsx:sync` y
`/opsx:archive`; prueba también `/refinar-hdu` y `/preparar-pruebas`. La instalación
de archivos y el descubrimiento efectivo en el cliente son comprobaciones distintas.

Ejemplos para el chat:

```text
Usa refinar-hdu con esta necesidad: el operador debe cancelar varias órdenes.
Usa preparar-pruebas sobre los criterios que acabamos de acordar.
/opsx:propose <descripción del cambio>
/opsx:apply <nombre-del-cambio>
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
contexto/proyecto.md               Contexto confirmado del sistema
.claude/commands/opsx/             Seis comandos oficiales generados
.claude/skills/openspec-*/         Seis skills oficiales generadas
.claude/skills/refinar-hdu/        Refinamiento funcional para PO
.claude/skills/preparar-pruebas/   Casos trazables para QA
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
La secuencia siguiente parte de la raíz del
clon de esta plantilla y deja ambas carpetas como hermanas; el nombre es ficticio:

```powershell
$s0Template = (Get-Location).Path
$s0Destino = Join-Path (Split-Path -Parent $s0Template) 's0-mi-sistema'
Set-Location (Split-Path -Parent $s0Template)
openspec store setup mi-sistema --path $s0Destino
Set-Location $s0Template
```

`store setup` crea la estructura oficial, registra el store e inicializa Git con
un commit; no vuelvas a ejecutar `git init`. Desde el clon de esta plantilla,
copia solamente las piezas del núcleo a la nueva instancia:

```powershell
Copy-Item -LiteralPath .gitignore,.gitattributes,README.md -Destination ..\s0-mi-sistema
Copy-Item -LiteralPath contexto -Destination ..\s0-mi-sistema -Recurse
Copy-Item -LiteralPath openspec\config.yaml -Destination ..\s0-mi-sistema\openspec\config.yaml
New-Item -ItemType Directory -Force ..\s0-mi-sistema\.claude\skills | Out-Null
Copy-Item -LiteralPath .claude\skills\refinar-hdu,.claude\skills\preparar-pruebas -Destination ..\s0-mi-sistema\.claude\skills -Recurse
cd ..\s0-mi-sistema
openspec init --tools claude --profile core --no-animation
```

`--profile core` elige los seis workflows; la preferencia local `delivery` decide
si se generan skills, comandos o ambos. Para generar ambas integraciones comprueba
`openspec config get delivery` y, si corresponde, usa
`openspec config set delivery both` antes de `init`. Ese ajuste cambia la
preferencia de la máquina; no modifica el store ni los permisos del agente.

Sustituye el contexto y las referencias a la plantilla en este README por datos
confirmados del sistema. No copies `.git/`, `.openspec-store/`, `VALIDACION.md`,
specs del piloto ni estado local. Las carpetas `specs/` y `changes/archive/`
comienzan vacías. Conecta después el remoto **del sistema**, publica mediante
ramas/PR y configura la protección de rama según la política del host Git.

## Conectar un repositorio de código

Registra primero el S0 en la máquina. En la raíz Git del consumidor, crea
`openspec/config.yaml` con un único puntero (usa el ID real de tu sistema):

```yaml
store: s0-core
```

Luego, siempre desde esa raíz:

```powershell
openspec init --tools claude --profile core --no-animation
openspec context --json
openspec status
```

El inicializador conserva el puntero y sólo añade integraciones al consumidor.
La misma preferencia local `delivery` se aplica aquí; compruébala si necesitas
ambas formas de invocación.
No crees `openspec/specs/` ni `openspec/changes/` allí: tienen precedencia sobre el
puntero. Confirma que `context` resuelve la raíz del S0. Para implementar abre S0
y el código en un workspace con acceso a ambas carpetas. Guarda los workspaces
con rutas personales fuera del repositorio compartido. Worksets es opcional.

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
6. **Equipo:** revisa resultados y alcance. `/opsx:sync` consolida specs;
   `/opsx:archive` cierra el cambio. La CLI `archive` también puede consolidar specs.

Usa el PR para enlazar HDU o solicitud, ID OpenSpec, PR de código y resultados de
pruebas sobre una revisión identificable. Mientras una propuesta está en una rama,
los revisores deben usar esa rama o su diff; `main` conserva lo consolidado antes.
Protección de rama, aprobaciones y merge se configuran en Git, no por estas skills.

En QA distingue **caso preparado**, **test implementado** y **test ejecutado sobre
una versión**. Mantén fallos y pendientes visibles; no marques como ejecutado lo
que sólo es una matriz o un archivo de test.

## Comprobaciones y mantenimiento

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
registro, resolución store-only, validación y ciclo de cambio. No edites a mano
los workflows generados. Las dos skills institucionales se revisan por PR.

## Clientes y alcance de la certificación

El primer cliente objetivo es Claude Code en VS Code. Claude Desktop Code usa
las skills del proyecto cuando la carpeta y sus permisos están habilitados, pero
necesita su propia prueba de descubrimiento y ejecución. En Cowork/Chat no se
supone carga automática de `.claude/skills`: empaquetado y habilitación son un
piloto separado. Codex, Jira y Worksets son extensiones opcionales no instaladas
por esta plantilla.

El [registro de validación](VALIDACION.md) distingue pruebas locales ejecutadas de
revisión humana, permisos reales y clientes pendientes. Un piloto local no implica
certificación institucional completa.

## Referencias oficiales

- [Stores y resolución store-only](https://openspec.dev/docs/stores).
- [CLI y flags soportados](https://openspec.dev/docs/cli).
- [Perfiles y seis workflows core](https://openspec.dev/docs/profiles).
- [Configuración de proyecto](https://openspec.dev/docs/configuration/config-yaml).
