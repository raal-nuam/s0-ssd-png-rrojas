# Conectar y usar S0 en tu equipo

Elige tu recorrido. El responsable crea y clona el S0 manualmente; después pide
`connect-project` al agente. La skill prepara la conexión local y continúa con
un borrador de contexto. El equipo revisa ese borrador antes de guardarlo.

| Tu situación | Recorrido |
| --- | --- |
| Sistema existente con uno o varios repositorios | [1. Conectar un sistema existente](#existing-system) |
| Proyecto nuevo, todavía sin código | [2. Comenzar sin código](#new-system) |
| PO, QA o lector de un S0 ya preparado | [3. Incorporarse al equipo](#join-team) |

Los ejemplos usan `your-team`, `my-system` y carpetas ficticias. Sustitúyelos
por tus datos. Los prompts van en el chat del agente; los bloques PowerShell
de la [referencia avanzada](#cli-setup) no forman parte del recorrido inicial.

## Qué vas a conectar

- **Plantilla maestra:** `s0-core-template`, con estructura y métodos reutilizables.
- **S0 del sistema:** repositorio independiente con contexto, specs y cambios propios.
- **Consumidores:** uno o varios repositorios de código que apuntan al mismo S0.

Un monorepo con varios componentes se conecta una vez desde su raíz Git.
Diez o veinte repositorios pueden compartir el mismo S0. El número de servicios
no obliga a crear más stores. Si un subsistema tiene otro S0, sus requisitos
internos mantienen allí su fuente canónica; referencia sus contratos e integración.

<a id="requirements"></a>
## Herramientas según tu tarea

| Tarea | Qué necesitas |
| --- | --- |
| Crear el repositorio desde GitHub Template | Navegador y una cuenta con acceso a la plantilla |
| Clonar y conectar el S0 | Git, agente con acceso a las carpetas y OpenSpec 1.14.3; `connect-project` puede instalar la CLI con Node.js >=20.19.0 y npm disponibles |
| Leer, refinar o preparar casos desde el S0 | Editor o agente con acceso al S0; no necesitas OpenSpec |
| Ejecutar workflows nativos como `propose` o `apply` | CLI OpenSpec 1.14.3 y agente con acceso a S0 y repositorios pertinentes |
| Guardar el contexto aprobado por Discovery | Git y Python >=3.10 para el verificador incluido |
| Automatizar o ejecutar pruebas de producto | Repositorio de la suite, entorno, datos y herramientas del producto |

Node.js es requisito de la distribución npm de OpenSpec, no de la lectura del S0.
El binario autónomo equivalente no lo requiere. La instalación del agente puede
tener otros requisitos. Python se usa para verificar la publicación de contexto;
su ausencia no impide preparar la conexión ni entregar un borrador de lectura.

Si falta Git, Node o el agente, utiliza sus instaladores oficiales:

- [Git para Windows](https://git-scm.com/install/windows).
- [Node.js](https://nodejs.org/en/download), si usarás npm para OpenSpec.
- [OpenSpec](https://openspec.dev/docs/installation), si prefieres el binario autónomo 1.14.3.
- [Codex](https://learn.chatgpt.com/docs/quickstart) o [Claude Code](https://code.claude.com/docs/en/quickstart).

`connect-project` no instala Node, Git, Python ni clientes de IA. Si falta una
herramienta indispensable para una fase, debe indicar qué quedó pendiente.

<a id="existing-system"></a>
## 1. Conectar un sistema existente

### Paso 1. Crea el S0 manualmente en GitHub

1. Abre [s0-core-template](https://github.com/fdelvalle01/s0-core-template).
2. Elige **Use this template → Create a new repository**.
3. Indica propietario, nombre como `s0-my-system` y visibilidad.
4. Copia sólo la rama predeterminada; deja desmarcado **Include all branches**.
5. Crea el repositorio y comprueba que su URL corresponde al nuevo sistema.

GitHub inicia un historial independiente. No es un fork ni una sincronización
continua con el maestro. Esta acción la realiza la persona, no la skill.

### Paso 2. Clona el repositorio nuevo

Usa la URL del S0 recién creado, no la de la plantilla. Con tu cliente Git,
clónalo en una carpeta separada del código. Por ejemplo, en PowerShell:

```powershell
git clone https://github.com/your-team/s0-my-system.git s0-my-system
Set-Location s0-my-system
```

No necesitas crear otra carpeta `.git` ni configurar OpenSpec a mano.
Si tu código aún no está disponible localmente, identifica las fuentes que sí
puedes compartir; una URL en un inventario no concede acceso a su contenido.

### Paso 3. Abre el S0 y pide `connect-project`

Abre la carpeta que contiene `AGENTS.md`, `context/` y `openspec/` con tu agente.
Comprueba que la skill pertenece a este S0, según [Skills locales](#local-skills).
En Codex usa `$connect-project`; en Claude Code, `/connect-project`, o escribe:

```text
Usa la skill local .agents/skills/connect-project/SKILL.md.
Conecta este S0 como my-system. El producto es [nombre y propósito].
Puedes configurar estos repositorios Git: D:\my-system-web y D:\my-system-api.
Prepara la conexión local y continúa con el borrador de contexto de Discovery.
```

Sustituye nombre, propósito y rutas. Para un monorepo indica sólo la raíz Git
y menciona sus componentes; no repitas sus subcarpetas como repositorios.
Para varios Git, enumera las raíces que quieres conectar ahora. Puedes hacerlo
progresivamente: no hace falta descargar veinte repositorios para describir
el mapa o empezar por un componente.
También puedes indicar una carpeta que agrupe los repositorios del sistema;
el agente identifica sus raíces Git dentro de ese límite y aclara selecciones
ambiguas. No clona repositorios adicionales por recibir esa carpeta o un enlace.

Con esa petición autorizas la preparación local de los repositorios indicados.
La skill comprueba la raíz y la identidad del S0, resuelve la CLI 1.14.3 y puede
instalarla por npm si dispone del runtime requerido. Si el PATH selecciona otra
versión, utiliza una ruta de CLI comprobada y comunica esa diferencia.

Conserva una identidad ya adoptada y un registro correcto. Un conflicto con otro
checkout o sistema se presenta como bloqueo, sin reemplazarlo por inferencia.
En cada consumidor añade o conserva el puntero y genera las integraciones
oficiales. Si encuentra planificación OpenSpec local, no la borra ni migra
automáticamente: señala el contenido que requiere una decisión aparte.

### Paso 4. Revisa el borrador de Discovery

Terminada la preparación posible, el agente presenta contexto basado en las
fuentes disponibles: propósito, componentes, relaciones, evidencias y dudas.
El código no se convierte automáticamente en specs ni en reglas acordadas.

Revisa el contenido y los archivos destino. Cuando apruebes ese borrador concreto,
Discovery guarda únicamente el contexto aprobado y verifica su alcance.
Si falta Python, puede entregar el borrador, pero no publicar contexto como
verificado. Si una fuente es inaccesible, debe indicarlo y limitar su cobertura.

Los cambios de configuración pertenecen a `connect-project`; la referencia de
lectura de Discovery se toma después de esa preparación. Discovery no modifica
identidad, configuración, integraciones ni artefactos OpenSpec.

### Paso 5. Comparte el resultado con tu equipo

Revisa los diffs del S0 y de cada consumidor. `connect-project` no hace commit,
push ni PR: la publicación sigue el proceso Git del equipo. Hasta publicar
esos cambios, otros integrantes no reciben la configuración local.

Entrega al equipo la URL del S0, su ID y el [recorrido 3](#join-team). El resultado
esperado es un S0 propio con consumidores conectados y contexto revisable;
la conexión no acredita todavía implementación ni aceptación del producto.

<a id="connect-code"></a>
### Añadir más repositorios después

Desde el mismo S0 pide `connect-project` para las nuevas raíces Git. Conserva
el ID existente. Si el puntero y el registro ya son correctos, no necesita
reescribirlos. No conectes las carpetas de componentes de un monorepo por separado.

El inventario `repositories.yaml` es opcional. Para pocos componentes basta
`context/project.md`; para muchos puedes mantener allí una tabla o añadir el
inventario con nombres, responsabilidades y URLs Git. No guardes rutas personales
ni credenciales. OpenSpec no interpreta ese inventario ni clona código por él.

<a id="new-system"></a>
## 2. Comenzar un proyecto nuevo, sin código

1. Crea el repositorio con GitHub Template y clónalo manualmente, como en los
   pasos 1 y 2 del [recorrido anterior](#existing-system).
2. Ábrelo con tu agente y pide la conexión sin inventar repositorios de producto:

```text
Usa la skill local .agents/skills/connect-project/SKILL.md.
Prepara este S0 para my-system. El objetivo inicial es [problema y usuarios].
Todavía no hay código ni repositorios consumidores. Deja esa decisión pendiente
y prepara un borrador de contexto inicial para revisar.
```

El contexto debe separar intención, decisiones propuestas e información conocida.
No presenta componentes o funcionalidades como implementados. No necesitas
crear frontend, API, inventarios ni repositorios vacíos para empezar.

3. Revisa y aprueba el borrador de contexto. Comparte la preparación mediante
   el proceso Git del equipo. PO y QA pueden incorporarse desde ese momento.
4. Para planificar el primer cambio pide `refine-user-story` y, cuando corresponda,
   la skill oficial local `openspec-propose`. Los artefactos quedan en el S0.
5. Cuando existan repositorios de producto, vuelve a pedir `connect-project`
   para esas raíces. Luego solicita la implementación con tareas y acceso claros.

La planificación y las specs permanecen en el mismo S0 cuando comience el código.
Discovery es opcional para proyectos nuevos; no es un requisito previo de OpenSpec.

<a id="join-team"></a>
## 3. PO/QA: usar un S0 que el equipo ya preparó

### Obtén la revisión del sistema

Pide la URL del **S0 del sistema**, la rama o PR que debes revisar y el ID si
después usarás workflows nativos. No crees otro S0 desde la plantilla.

- **Git:** clona ese repositorio para recibir cambios o contribuir.
- **ZIP:** descarga la revisión acordada si no necesitas Git.

```powershell
git clone https://github.com/your-team/s0-my-system.git s0-my-system
Set-Location s0-my-system
```

Para una rama de PR añade `--branch branch-to-review` a `git clone`, sustituyendo
el nombre. Con ZIP, descarga la rama o revisión acordada y conserva también
`.agents/` y `.claude/`. Abre la carpeta que contiene `AGENTS.md`, `context/`
y `openspec/`, no sólo la carpeta de contexto o la del código.

### Trabaja con las fuentes locales

No necesitas CLI OpenSpec, registrar el store ni volver a pedir `connect-project`
para leer, refinar o preparar casos desde la raíz S0. Las instrucciones y skills
ya vienen versionadas. Tampoco necesitas clonar diez o veinte servicios.

Para PO, en el chat:

```text
Lee AGENTS.md, context/project.md y .agents/skills/refine-user-story/SKILL.md.
Refina esta necesidad con las specs pertinentes y devuelve el resultado
en conversación: [historia o necesidad].
```

Para QA:

```text
Lee AGENTS.md, context/project.md y .agents/skills/prepare-tests/SKILL.md.
Prepara casos para [historia, criterio o cambio]. Indica fuentes, dudas y
qué casos están preparados sin ejecutar.
```

Una historia puede ser texto; Jira es opcional. Si faltan specs, el agente debe
declararlo. Guardar resultados o automatizar pruebas requiere ese alcance y
acceso a la suite, entorno, datos y versión. Los tests de producto viven fuera
del S0. Distingue caso preparado, test implementado y test ejecutado.

Un clon no se actualiza solo: comprueba rama/commit y recibe los cambios de la
rama acordada sin perder tu trabajo local. Para un PR usa su rama o diff;
`main` conserva lo que ya se integró. Con ZIP descarga una nueva copia cuando
cambie la revisión. Los cambios sólo locales de DEV no llegan a otros usuarios.

Si te incorporas como DEV y necesitas resolver el store desde código, pide
`connect-project` desde el S0 con su ID existente y las raíces pertinentes.
Debe conservar identidad y configuración compartidas; sólo prepara lo que falte
en tu entorno o repositorios autorizados.

<a id="local-skills"></a>
## Comprobar la skill que usa tu agente

La CLI instalada y el catálogo de skills son cosas distintas. Comprueba que
la ruta seleccionada está bajo `.agents/skills/` del S0, o en su puente
`.claude/skills/`. Una entrada personal del mismo nombre puede coexistir.
Si no aparece o hay una coincidencia, pide leer la ruta local explícitamente.

Codex usa `.agents/skills/`; Claude Code usa sus entradas `.claude/skills/`.
Las propias de Claude apuntan a las mismas fuentes completas. La prioridad y
el selector dependen del cliente; otros agentes pueden leer los archivos,
pero su descubrimiento automático necesita comprobarse por separado.

Un `store:` en el código no instala las skills propias ni carga automáticamente
las instrucciones del S0. Desde el consumidor resuelve `openspec context --json`
y lee `AGENTS.md`, `context/project.md` y la skill requerida bajo `root.path`.
Da acceso a las raíces necesarias; abrir un workspace no concede permisos.

<a id="project-discovery"></a>
## Discovery después de la conexión

`connect-project` continúa con el borrador de Discovery. Para actualizar sólo
un componente de un sistema ya conectado, puedes pedir directamente
`$project-discovery`, `/project-discovery` o su fuente local
[.agents/skills/project-discovery/SKILL.md](.agents/skills/project-discovery/SKILL.md).

Indica las fuentes y el foco. Revisa el diff propuesto antes de guardar contexto.
La publicación exige Git/Python y el [verificador de alcance](.agents/skills/project-discovery/SKILL.md#scope-checks),
con la misma referencia antes y después del borrador aprobado. Sus archivos
de comprobación quedan fuera de los repositorios analizados.

Discovery documenta hechos, inferencias y dudas; no instala producto, ejecuta
tests, genera specs ni altera la configuración o las integraciones OpenSpec.
Sin Python o una fuente verificable, entrega un borrador limitado sin publicar.

<a id="daily-work"></a>
## Del contexto a un cambio

| Paso | Resultado |
| --- | --- |
| PO refina | Necesidad, alcance, criterios y dudas en conversación o fuente acordada |
| DEV propone | `proposal.md`, specs delta, `design.md` y `tasks.md` en el S0 |
| Equipo revisa | Alcance y decisiones mediante su proceso Git |
| DEV implementa | Código y tests en los repositorios afectados |
| QA comprueba | Casos y ejecuciones sobre revisión, entorno y datos identificados |
| Equipo cierra | Specs consolidadas e historial mediante `sync` y `archive` |

Usa las [invocaciones del README](README.md#trabajar-con-las-skills) en el chat.
Los seis workflows oficiales requieren CLI. `sync` es un workflow del agente;
no existe un comando de terminal `openspec sync`. Enlaza cambio S0, PR de código
y resultados de pruebas. La validación estructural no acredita aceptación funcional.

<a id="cli-setup"></a>
## Referencia técnica avanzada

**No sigas estos bloques para una primera conexión con el agente:**
`connect-project` realiza esas comprobaciones y operaciones. Esta referencia
sirve para diagnóstico o configuración manual deliberada, desde la raíz indicada.
Los comandos son PowerShell y usan datos ficticios.

### CLI correcta

```powershell
openspec --version
Get-Command openspec -All | Select-Object Source
$s0NpmPrefix = (npm prefix -g).Trim()
& (Join-Path $s0NpmPrefix 'openspec.cmd') --version
```

Si falta la CLI y ya tienes Node.js >=20.19.0 y npm, la instalación manual es
`npm install -g @fission-ai/openspec@1.14.3`. Si el PATH conserva otra versión,
usa la ruta comprobada de 1.14.3. Para priorizarla sólo en esa terminal:

```powershell
$env:Path = "$s0NpmPrefix;$env:Path"
openspec --version
```

No actualiza procesos abiertos ni cambia permanentemente el PATH. Comprueba
también el ejecutable que verá el agente antes de usar workflows nativos.

### Identidad y registro

En un S0 ya adoptado conserva su identidad. Para registrarlo en esa máquina:

```powershell
Get-Content -LiteralPath .openspec-store/store.yaml
openspec store list
openspec store register .
```

Un ID resuelve una sola ruta por registro; no reemplaces un checkout ajeno.
En un clon nuevo de la plantilla puede existir un marcador genérico sin ID
o con `id: s0-core`. Sólo para ese caso, antes de adoptarlo, la secuencia manual
es retirar ese marcador y dejar que la CLI cree la identidad propia:

```powershell
$s0Id = 'my-system'
Remove-Item -LiteralPath .openspec-store/store.yaml
openspec store register . --id $s0Id --yes
openspec store doctor $s0Id
```

No lo repitas sobre un sistema preparado. En 1.14.3, `register --id` rechaza
una identidad existente distinta; no es un mecanismo para renombrar sistemas.

### Consumidor sin planificación OpenSpec previa

Desde la raíz Git del consumidor, tras registrar el S0. Si hay config, specs o
changes previos, detén este procedimiento y conserva su contenido e historial.

```powershell
if ((Test-Path openspec/config.yaml) -or (Test-Path openspec/config.yml) -or
    (Test-Path openspec/specs) -or (Test-Path openspec/changes)) {
    throw 'Existe OpenSpec previo: requiere revisión, no instalación inicial.'
}
New-Item -ItemType Directory -Path openspec -Force | Out-Null
Set-Content -LiteralPath openspec/config.yaml -Value 'store: my-system' -Encoding utf8
openspec init --tools "claude,codex" --profile core --no-animation
openspec context --json
```

Conserva las comillas de `"claude,codex"` en PowerShell. La preferencia local
`delivery` controla skills/comandos; consulta `openspec config get delivery`.
Cambiarla manualmente afecta futuras inicializaciones de esa instalación;
`connect-project` usa preferencias temporales y conserva la configuración persistente.
El contexto debe resolver el S0, sin specs/changes locales.
`members: []` es normal sin Worksets.
Las instrucciones raíz y anidadas pueden conservar referencias antiguas:
reconcílialas preservando convenciones válidas. `init` no las migra por ti.

<a id="maintenance"></a>
## Mantenimiento y límites

`connect-project` prepara diffs locales. La publicación Git y la migración de
planificación anterior son trabajos separados. Para implementar entre varios
Git, identifica las tareas por repositorio y el acceso requerido; OpenSpec no
distribuye automáticamente tareas o permisos.

Las mejoras del maestro se adoptan mediante un PR selectivo. Conserva identidad,
contexto, inventario, specs e historial de cada sistema. `git pull` actualiza el
repositorio del clon; `openspec update` regenera integraciones oficiales.
Revisa primero `openspec update --dry-run` y no edites workflows generados a mano.

Al actualizar desde nombres anteriores, renombra `contexto/proyecto.md` a
`context/project.md`, `refinar-hdu` a `refine-user-story`, `preparar-pruebas` a
`prepare-tests`, `repositorios.yaml` a `repositories.yaml` si existe y
`VALIDACION.md` a `VALIDATION.md`. Actualiza frontmatter, puentes y enlaces.
Conserva valores propios del sistema y usa claves de inventario en inglés.
No hace falta crear un inventario para actualizar.

Consulta [VALIDATION.md](VALIDATION.md) para repetir comprobaciones y distinguir
estructura, comportamiento y regresión semántica pendiente. El catálogo no
acredita ejecución de métodos; Cowork/Chat y otros clientes necesitan su prueba
de integración. Las referencias oficiales son [Stores](https://openspec.dev/docs/stores),
[CLI](https://openspec.dev/docs/cli), [Profiles](https://openspec.dev/docs/profiles)
y [configuración](https://openspec.dev/docs/configuration/config-yaml).
