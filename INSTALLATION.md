# Preparar o incorporarse a un S0

Elige el recorrido que corresponde a tu situación:

| Situación | Recorrido |
| --- | --- |
| Vas a adoptar S0 en un producto existente, con uno o varios repositorios | [1. Conectar el sistema](#existing-system) |
| Vas a comenzar un proyecto sin código | [2. Preparar un S0 nuevo](#new-system) |
| El equipo ya preparó el S0 y vas a utilizarlo | [3. Incorporarte al equipo](#join-team) |

La plantilla maestra crea instancias independientes. Cada S0 comparte contexto,
specs y cambios de un sistema; sus repositorios de código son los consumidores.
Los ejemplos usan nombres y rutas ficticios. Los prompts van en el chat del agente.

<a id="requirements"></a>
## Herramientas según tu tarea

| Tarea | Qué necesitas |
| --- | --- |
| Crear desde GitHub Template | Cuenta con acceso a la plantilla y navegador |
| Clonar y conectar | Git, agente con acceso a las carpetas y OpenSpec 1.14.3 |
| Descargar fuentes remotas | Acceso Git HTTPS o SSH a esos repositorios y una carpeta de trabajo externa al S0 |
| Leer, refinar historias o preparar casos desde el S0 | Editor o agente con acceso al S0; no requiere CLI OpenSpec |
| Ejecutar workflows oficiales | CLI OpenSpec 1.14.3 y acceso a las fuentes pertinentes |
| Guardar contexto mediante Discovery | Git y Python >=3.10 para verificar el alcance |
| Revisar PR | Acceso al PR o snapshot y código de su revisión; Node.js >=20 para verificar los informes; OpenSpec 1.14.3 si se resuelve un store desde un consumidor |
| Ejecutar producto o sus tests | Herramientas, entorno, datos y permisos del producto |

`connect-project` puede instalar OpenSpec 1.14.3 mediante npm si ya existen
Node.js >=20.19.0 y npm. El binario autónomo equivalente no requiere Node.
La skill no instala Git, Node, Python ni clientes de IA. Si falta una herramienta,
indica qué fase queda pendiente.

Instaladores oficiales: [Git](https://git-scm.com/install/windows),
[Node.js](https://nodejs.org/en/download), [OpenSpec](https://openspec.dev/docs/installation),
[Codex](https://learn.chatgpt.com/docs/quickstart) y
[Claude Code](https://code.claude.com/docs/en/quickstart).

<a id="existing-system"></a>
## 1. Conectar un sistema existente

**Crear desde plantilla → clonar → pedir connect-project → revisar contexto.**
Si ya creaste y clonaste tu S0 independiente, continúa en el paso 3.

### Paso 1. Crea el S0 en GitHub

1. Abre [s0-core-template](https://github.com/fdelvalle01/s0-core-template).
2. Elige **Use this template → Create a new repository**.
3. Indica propietario, nombre como `s0-my-system` y visibilidad.
4. Copia sólo la rama predeterminada; deja desmarcado **Include all branches**.

Obtendrás un repositorio con historial propio. La persona realiza esta acción;
las mejoras futuras de la plantilla no se incorporan automáticamente.

### Paso 2. Clona el repositorio nuevo

Usa la URL del S0 recién creado y una carpeta separada del código. En PowerShell:

```powershell
git clone https://github.com/your-team/s0-my-system.git s0-my-system
Set-Location s0-my-system
```

### Paso 3. Abre el S0 y pide la conexión

Abre con tu agente la raíz que contiene `AGENTS.md`, `context/` y `openspec/`.
En el chat, sustituye nombre, propósito y fuentes. Si ya tienes el código local:

```text
Usa la skill local .agents/skills/connect-project/SKILL.md.
Conecta este S0 como my-system. El producto es [nombre y propósito].
Puedes configurar estos repositorios Git: D:\my-system-web y D:\my-system-api.
Continúa con el borrador de contexto de Project Discovery.
```

Para un monorepo indica una sola raíz Git, aunque contenga varios servicios.
Para varios Git, enumera las raíces o una carpeta que los agrupe; el agente
identifica los repositorios dentro de ese límite. Puedes conectar más después.

Si el código está en GitHub, GitLab o Bitbucket, proporciona sus URLs Git:

```text
Usa connect-project para conectar este S0 con:
- Web: https://github.com/your-team/web.git, rama development.
- API: git@gitlab.com:your-team/api.git, rama main.
Descarga las copias en D:\workspaces y presenta el borrador de Project Discovery.
```

También puedes mezclar fuentes en una petición:

```text
Usa connect-project para conectar D:\my-system-api y
https://github.com/your-team/web.git, rama development.
Descarga lo necesario en D:\workspaces y continúa con el borrador de Discovery.
```

Elige una carpeta de descargas fuera del S0 y que no esté dentro de otro Git.
El agente recuerda ese destino en este clon del S0. Usa el acceso Git disponible;
no necesita tokens en el chat. En una fuente nueva, si omites la rama, resuelve la
predeterminada del remoto. No busca copias por todo el equipo ni descarga fuentes
no solicitadas.

La skill prepara identidad, registro, punteros e integraciones oficiales para
Codex/Claude. Conserva cambios locales y una identidad ya adoptada. Si encuentra
otro store o planificación OpenSpec previa incompatible, informa el conflicto
sin borrarla ni migrarla. Los resultados parciales deben identificar lo pendiente.

### Paso 4. Revisa el contexto y comparte la preparación

Después de configurar, el agente informa las copias, ramas, SHA y cambios locales
observados. Discovery toma entonces su referencia y presenta un borrador con fuentes
y archivos destino. Revisa ese contenido antes de aprobar su escritura. **Sin Python/Git o
una referencia válida, entrega sólo el borrador en conversación y no escribe
archivos de contexto.** Las fuentes inaccesibles se declaran como limitaciones.

Revisa los diffs del S0 y de sus consumidores. `connect-project` deja los cambios
locales, sin stage, commit, push ni PR. Publícalos mediante el proceso del equipo;
después comparte la URL del S0, su ID y el [recorrido de incorporación](#join-team).

<a id="connect-code"></a>
### Añadir repositorios o actualizar fuentes

Pide `connect-project` desde el mismo S0 para las nuevas rutas o URLs Git. Conserva
el ID existente; la reejecución reutiliza conexiones correctas y completa lo pendiente
sin buscar automáticamente nuevas revisiones.

Para trabajar con código remoto más reciente, solicítalo expresamente:

```text
Usa connect-project para actualizar la fuente Web conectada, rama development.
Conserva mi copia actual y prepara la revisión reciente en el workspace recordado.
Continúa con Project Discovery hasta presentar el borrador.
```

La actualización prepara otra copia si corresponde y conserva íntegramente la
anterior. Si la revisión no cambió y la copia sigue siendo válida, la reutiliza e
informa sus cambios locales. No hace pull, reset ni cambios de rama sobre tu trabajo.
Para actualizar desde una ruta local necesita un remoto y una rama inequívocos.
Las copias no se eliminan automáticamente; un cambio manual de rama o HEAD en una
copia registrada se informa, sin repararlo silenciosamente.

Un mapa en `context/project.md` suele bastar. `repositories.yaml` es un inventario
opcional, no interpretado por OpenSpec; puede contener responsabilidades, URLs y
ramas, sin rutas personales ni credenciales. Indica qué entradas quieres conectar:
su existencia no autoriza descargar todas.

<a id="new-system"></a>
## 2. Comenzar un proyecto nuevo, sin código

Crea y clona el S0 siguiendo los pasos 1 y 2 anteriores. Abre su raíz y escribe:

```text
Usa la skill local .agents/skills/connect-project/SKILL.md.
Prepara este S0 para my-system. El objetivo inicial es [problema y usuarios].
Todavía no hay código ni consumidores. Prepara un borrador de contexto inicial
para revisar, distinguiendo intención de implementación existente.
```

Revisa el borrador y sus destinos antes de guardarlo, con los mismos requisitos
de comprobación del paso 4. Publica la preparación por Git y continúa con el
[playbook](PLAYBOOK.md). No necesitas crear servicios o repositorios vacíos.
Cuando exista código, conecta sus repositorios al mismo S0. Discovery es opcional y
no constituye un requisito previo de OpenSpec.

<a id="join-team"></a>
## 3. Usar un S0 que el equipo ya preparó

1. Obtén la URL del **S0 del sistema** y la rama o revisión que debes consultar.
2. Clona ese repositorio, o descarga su ZIP si sólo necesitas los archivos.
3. Conserva `.agents/` y `.claude/` y abre la raíz que contiene `AGENTS.md`.
4. Continúa con las [skills y recorridos del playbook](PLAYBOOK.md#skills).

Para leer, refinar historias y preparar casos desde esa raíz no necesitas crear
otro S0, registrar el store, instalar OpenSpec ni descargar todos los servicios.
Para clonar una rama concreta puedes usar `git clone --branch branch-to-review URL`,
sustituyendo ambos valores. Con ZIP descarga la revisión acordada; no incluye Git.

Los cambios deben estar publicados para que otros los reciban. Comprueba la
rama/commit al revisar; un clon o ZIP no se actualiza solo. Recibe nuevas revisiones
conservando tu trabajo local.

Si eres DEV y necesitas resolver el store desde código en esta máquina, pide
`connect-project` desde el S0 con su ID existente y las rutas o URLs pertinentes.
Preparará lo que falte sin regenerar la identidad compartida.

<a id="local-skills"></a>
## Si el agente encuentra otra skill

Comprueba que usa la entrada del repositorio bajo `.agents/skills/` o su puente
`.claude/skills/`. Una skill personal del mismo nombre puede ser otra versión.
Si no aparece la local, pide leer su `SKILL.md` explícitamente. El descubrimiento
y la prioridad dependen del cliente; otros agentes pueden leer los archivos,
pero su integración debe comprobarse.

Desde un consumidor, `store:` no instala las skills propias ni carga por sí solo
las instrucciones del S0. Resuelve `openspec context --json` y lee `AGENTS.md`,
`context/project.md` y la skill pertinente bajo `root.path`.

<a id="project-discovery"></a>
## Actualizar el contexto después

Pide `project-discovery` con las fuentes y el foco que necesitas revisar.
Documenta propósito, mapa, arquitectura, tecnologías, interfaces e infraestructura
según la evidencia disponible. `context/project.md` es el índice; los demás
archivos son opcionales y se separan sólo cuando aportan valor.

El método de lectura, borrador y aprobación se define en
[Project Discovery](.agents/skills/project-discovery/SKILL.md), junto con la
[organización del contexto](.agents/skills/project-discovery/SKILL.md#context-content)
y sus comprobaciones. El código observado no se convierte automáticamente en specs.

<a id="daily-work"></a>
Para trabajar con historias, cambios y pruebas, continúa en [PLAYBOOK.md](PLAYBOOK.md).

<a id="cli-setup"></a>
## Referencia técnica avanzada

Estos comandos sirven para diagnóstico o configuración manual deliberada.
La primera conexión con el agente sigue el recorrido anterior. Usa el ejecutable
1.14.3 comprobado en los comandos que siguen y la raíz indicada en cada sección.

### Localizar las copias remotas

`connect-project` recuerda el destino en la configuración Git local del S0:
`git config --local --get s0.sourcesRoot`. Bajo ese workspace mantiene
`.s0-connections/<store-id>.json` con la instancia, fuentes, ramas, rutas, SHA y
estado de sus copias. Es un índice local que el agente contrasta con Git, no
configuración OpenSpec ni evidencia suficiente del contexto. No se publica con
el S0; las fuentes y revisiones pertinentes sí acompañan al contexto aprobado.

### Comprobar la CLI

```powershell
openspec --version
Get-Command openspec -All | Select-Object Source
$s0NpmPrefix = (npm prefix -g).Trim()
& (Join-Path $s0NpmPrefix 'openspec.cmd') --version
```

El prefijo npm aplica si utilizas esa distribución. Si falta la CLI y ya tienes
Node/npm compatibles: `npm install -g @fission-ai/openspec@1.14.3`.
Si otra versión aparece primero, utiliza la ruta explícita correcta. Para
priorizarla sólo en esa terminal:

```powershell
$env:Path = "$s0NpmPrefix;$env:Path"
openspec --version
```

Esto no cambia permanentemente PATH ni afecta procesos ya abiertos. Comprueba
el ejecutable efectivo al comenzar una nueva sesión, según [AGENTS.md](AGENTS.md).

### Registrar una identidad existente

Desde la raíz de un S0 ya adoptado:

```powershell
Get-Content -LiteralPath .openspec-store/store.yaml
openspec store list
openspec store register .
```

Un ID resuelve una ruta por registro. Ante un conflicto, conserva la asociación
existente. `register --id` no renombra un store que ya tiene otra identidad.

Sólo en un clon nuevo comprobado de la plantilla, con remoto propio y sin contenido
de producto, puede retirarse el marcador heredado para crear la identidad oficial.
Conserva antes una copia del marcador fuera del repositorio y comprueba que el ID
está disponible; restaura esa copia si falla el registro:

```powershell
$s0Id = 'my-system'
Remove-Item -LiteralPath .openspec-store/store.yaml
openspec store register . --id $s0Id --yes
openspec store doctor $s0Id
```

### Conectar un consumidor sin planificación previa

Desde su raíz Git, con el S0 registrado. Una configuración o planificación previa
requiere revisión; este bloque no la migra:

```powershell
if ((Test-Path openspec/config.yaml) -or (Test-Path openspec/config.yml) -or
    (Test-Path openspec/specs) -or (Test-Path openspec/changes)) {
    throw 'Existe OpenSpec previo: requiere revisión.'
}
New-Item -ItemType Directory -Path openspec -Force | Out-Null
Set-Content -LiteralPath openspec/config.yaml -Value 'store: my-system' -Encoding utf8
openspec init --tools "claude,codex" --profile core --no-animation
openspec context --json
```

Conserva las comillas en PowerShell. La preferencia `delivery` controla skills y
comandos; `openspec config get delivery` permite consultarla. `connect-project`
usa preferencias temporales para generar ambos sin cambiar las persistentes.
La raíz resuelta debe ser el S0; no deben aparecer specs/changes en el consumidor.
`members: []` es normal sin Worksets. Revisa también instrucciones raíz y anidadas:
`init` no reconcilia sus referencias antiguas por ti.

<a id="maintenance"></a>
## Mantenimiento

Adopta mejoras del maestro mediante un PR selectivo, conservando identidad,
contexto, specs e historial del sistema. `git pull` recibe cambios del repositorio;
`openspec update` regenera integraciones oficiales. Revisa primero
`openspec update --dry-run`; no edites los workflows generados a mano.

Al migrar nombres antiguos, usa `context/project.md`, `refine-user-story`,
`prepare-tests`, `repositories.yaml` y `VALIDATION.md`; ajusta puentes y enlaces
conservando los datos propios. No hace falta crear un inventario para migrar.

Consulta [VALIDATION.md](VALIDATION.md) para comprobar una actualización y conocer
lo pendiente. Referencias oficiales: [Stores](https://openspec.dev/docs/stores),
[CLI](https://openspec.dev/docs/cli), [Profiles](https://openspec.dev/docs/profiles) y
[configuración](https://openspec.dev/docs/configuration/config-yaml).
