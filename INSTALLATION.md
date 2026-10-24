# Instalar y usar S0 en tu equipo

Elige el recorrido que corresponde a tu situación. Los pasos de creación los hace
una persona por sistema; quienes se incorporan después reciben esa configuración
por Git. Un S0 puede reunir el contexto y las especificaciones de un producto
completo, aunque su código esté repartido entre muchos repositorios.

| Recorrido | Quién lo sigue | Resultado |
| --- | --- | --- |
| [1. Sistema existente](#existing-system) | Responsable de incorporar S0 a un producto | S0 propio, contexto inicial y repositorios conectados |
| [2. Proyecto nuevo](#new-system) | Responsable de iniciar un producto sin código | S0 listo para definir y planificar; código conectado cuando exista |
| [3. Incorporarse al equipo](#join-team) | PO, QA, lector o nuevo integrante | Copia del S0 ya preparado y acceso a sus métodos |

**Los comandos de esta guía son para PowerShell en Windows.** En los ejemplos,
`your-team`, `my-system` y las rutas `D:\...` se sustituyen por los datos reales.
Ejecuta los bloques de tu recorrido, en orden; si un paso falla, resuelve ese
error antes de continuar. Los prompts marcados como texto van en el chat del
agente, no en la terminal.

## Antes de empezar: tres cosas diferentes

| Nombre | Qué contiene | Ejemplo |
| --- | --- | --- |
| Plantilla maestra | Estructura, instrucciones y skills reutilizables | `s0-core-template` |
| S0 del sistema | Contexto, specs, propuestas y skills de **tu sistema** | `s0-my-system` |
| Repositorios de código | Aplicaciones, servicios y tests | `web-app`, `order-api`, `market-data` |

Crear desde plantilla en GitHub copia los archivos iniciales a un **nuevo
repositorio con historial independiente**. Clonar descarga un repositorio que
ya existe. Por eso sólo el responsable crea desde la plantilla; PO y QA clonan
el S0 que el equipo ya preparó. No hay un S0 distinto por persona.

El **store** es la raíz OpenSpec de ese S0. Su ID compartido, por ejemplo
`my-system`, vive en `.openspec-store/store.yaml`. El registro local asocia ese
ID a una carpeta de tu máquina para que la CLI pueda encontrarla. Registrar no
descarga contenido, no actualiza Git ni copia skills a otras carpetas.

<a id="requirements"></a>
## Herramientas: instala sólo lo que necesitas

| Lo que vas a hacer | Lo necesario |
| --- | --- |
| Leer documentos en GitHub | Navegador y acceso al repositorio |
| Leer una copia local | Editor; Git para clonar/actualizar, o ZIP sin Git |
| Refinar historias y preparar casos desde el S0 | Copia del S0 y agente instalado con acceso a sus archivos; no requiere CLI OpenSpec |
| Descubrir contexto y publicar el borrador aprobado | Skill Project Discovery y repositorios autorizados; Git y Python >=3.10 para verificar la escritura. No requiere CLI OpenSpec |
| Crear/configurar el S0 | Git y CLI OpenSpec 1.14.3 |
| Ejecutar workflows nativos como `propose`, `apply` o `archive` | CLI OpenSpec 1.14.3, agente compatible y acceso a las carpetas implicadas |
| Automatizar o ejecutar pruebas | Repositorio de la suite, entorno y herramientas del producto; OpenSpec sólo si además usas sus workflows |

La distribución npm de OpenSpec requiere Node.js **>=20.19.0**. El binario
autónomo equivalente no requiere Node. El agente y las herramientas del producto
tienen sus propios requisitos. S0 no selecciona un modelo ni impone una cuenta
Jira; las conexiones que quieras usar deben estar disponibles en tu sesión.

Si te falta alguna herramienta, usa su distribución oficial:

- [Git para Windows](https://git-scm.com/install/windows), para clonar, publicar y recibir cambios.
- [Node.js](https://nodejs.org/en/download), si eliges instalar OpenSpec con npm: una versión LTS compatible con el mínimo indicado.
- [Instalación de OpenSpec](https://openspec.dev/docs/installation), si eliges el binario autónomo: selecciona la versión acordada **1.14.3**, pues el instalador predeterminado puede instalar otra más reciente.
- [Inicio con Codex](https://learn.chatgpt.com/docs/quickstart) o [Claude Code](https://code.claude.com/docs/en/quickstart), si todavía no tienes un agente con acceso a carpetas locales.

Después de instalar Git o Node, abre otra terminal para que reciba su PATH.

<a id="cli-setup"></a>
### Preparar la CLI, cuando tu recorrido la necesita

Si ya la tienes, comprueba primero `openspec --version`. Esta guía está validada
con **1.14.3**. Para instalar esa versión mediante npm:

```powershell
git --version
node --version
npm --version
npm install -g @fission-ai/openspec@1.14.3
openspec --version
```

El último comando debe mostrar `1.14.3`. Si muestra otra versión, **la instalación
puede estar bien y Windows estar ejecutando otra copia anterior**. Nos ocurrió
con un wrapper de BrainProjects que devolvía `1.13.2`. Comprueba las rutas:

```powershell
Get-Command openspec -All | Select-Object Source
$s0NpmPrefix = (npm prefix -g).Trim()
& (Join-Path $s0NpmPrefix 'openspec.cmd') --version
```

Si la copia de npm muestra `1.14.3`, puedes ponerla primero en el PATH de **esta
terminal** y comprobar de nuevo:

```powershell
$env:Path = "$s0NpmPrefix;$env:Path"
openspec --version
```

Este cambio no es permanente y no actualiza aplicaciones ya abiertas. Usa la
terminal corregida para iniciar tu cliente CLI, o ajusta la precedencia del PATH
de usuario mediante la configuración de Windows y reabre tu editor/cliente.
Comprueba la versión también en la terminal desde la que trabajará el agente.
No necesitas borrar otras instalaciones para diagnosticarlo.

<a id="existing-system"></a>
## 1. Adoptar S0 en un sistema existente

Sigue este recorrido si tu aplicación o sistema ya tiene código. Lo hace el
responsable de la adopción; al terminar, el resto usa el [recorrido 3](#join-team).
La secuencia es: crear S0 → darle identidad y contexto → conectar código →
publicar los cambios para el equipo.

### Paso 1. Identifica el sistema y sus componentes

Acuerda el nombre del S0, su ID y qué repositorios pertenecen al sistema. No hace
falta clonar todo para escribir ese mapa; el equipo puede aportar URLs y
responsabilidades. Para verificar código necesitarás acceso a los repositorios
relevantes.

- Si hay un único Git con frontend y backend en subcarpetas, tienes **un
  repositorio con varios componentes**. Se conecta una vez desde su raíz Git.
- Si hay diez o veinte Git independientes, conectas cada repositorio participante
  al **mismo ID de S0**. El número de repositorios no crea más S0 automáticamente.
- Si un subsistema tiene S0 propio, define qué requisitos son canónicos allí y
  referencia esa fuente. Evita mantener dos copias del mismo requisito.

No conviertas automáticamente todo el código existente en specs. Empieza con
contexto verificable y consolida requisitos con el equipo; señala lo desconocido.

### Paso 2. Crea el repositorio S0 en GitHub

1. Abre [s0-core-template](https://github.com/fdelvalle01/s0-core-template) con una cuenta que tenga acceso.
2. Encima de la lista de archivos, pulsa **Use this template** y luego **Create a new repository**. Es un botón de GitHub, no un comando.
3. Elige propietario, nombre —por ejemplo `s0-my-system`— y visibilidad. Copia sólo la rama predeterminada; deja desmarcado **Include all branches**.
4. Pulsa **Create repository** y abre el nuevo repositorio.

Ahora tienes un Git independiente con una copia inicial del core. Los siguientes
comandos usan **la URL del repositorio nuevo**, no la de `s0-core-template`.
Este comportamiento está descrito en la
[guía de GitHub Template](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-repository-from-a-template).

### Paso 3. Clona y prepara tu rama

Prepara [Git y la CLI](#cli-setup). Elige una carpeta para el S0 separada de las
carpetas de código. En este ejemplo todos los repositorios son hermanos en `D:\`:

```powershell
Set-Location D:\
git clone https://github.com/your-team/s0-my-system.git s0-my-system
Set-Location D:\s0-my-system
git remote -v
git switch -c s0/setup-my-system
```

Comprueba que `origin` apunta a `s0-my-system`. El clon ya contiene su Git;
no necesita `git init` ni copiar la carpeta `.git` de otra instalación.

### Paso 4. Dale identidad propia, una sola vez

**Sólo en el S0 recién creado desde la plantilla**, revisa el marcador heredado
y los registros locales:

```powershell
Get-Content -LiteralPath .openspec-store/store.yaml
openspec store list
```

La plantilla puede traer `id: s0-core` o metadata genérica con sólo `version: 1`.
Todavía no es la identidad de tu sistema. Elige un ID propio, por ejemplo
`my-system`, que no esté asociado a otro sistema en tu máquina. Si el archivo ya
identifica un sistema configurado, corresponde al recorrido 3: no lo retires.

En ese clon nuevo, retira únicamente el marcador heredado y deja que la CLI
oficial cree la identidad:

```powershell
$s0Id = 'my-system'
Remove-Item -LiteralPath .openspec-store/store.yaml
openspec store register . --id $s0Id --yes
openspec store doctor $s0Id
openspec status --store $s0Id
```

Resultado esperado: identidad `my-system`, registro apuntando a tu carpeta y
`store doctor` sin incidencias. `No active changes` es correcto al empezar.
En 1.14.3, `register --id` rechaza una identidad existente distinta incluso con
`--yes`; por eso se regenera sólo el marcador inicial. No se edita su YAML a mano.
No repitas este paso para cada integrante o repositorio de código.

Si ya registraste el clon con el ID de la plantilla o hay un conflicto de ID,
revisa qué ruta muestra `store list` antes de cambiar asociaciones. No retires
un registro que pertenece a otro checkout o sistema por suponer que sobra.

### Paso 5. Completa el contexto que recibirá todo el equipo

Abre `context/project.md` y reemplaza el contenido de ejemplo por información
confirmada. Es el lugar donde alguien nuevo debe poder entender el sistema:

| Información | Qué escribir |
| --- | --- |
| Propósito y usuarios | Qué problema resuelve y para quién |
| Alcance y límites | Qué pertenece al sistema y qué depende de otros |
| Identidad | Nombre, ID del store y URL de este S0 |
| Componentes | Nombre, responsabilidad, URL Git y subcarpeta cuando corresponda |
| Fuentes | Specs, contratos/documentos existentes y S0 de otros subsistemas |
| Pendientes | Lo que aún falta confirmar; sin inventar reglas ni tecnología |

Puedes completar esta página manualmente o usar [Project Discovery](#project-discovery)
para inspeccionar los repositorios autorizados y revisar un borrador antes de
guardarlo. Es opcional y también permite actualizar sólo algunos componentes.

Para un monorepo, describe sus componentes con la misma URL y sus respectivas
subcarpetas. Para muchos repositorios puedes usar una tabla en esa misma página.
Si el mapa crece, `repositories.yaml` es una alternativa **opcional**:

```yaml
repositories:
  - name: web-app
    type: frontend
    url: https://github.com/your-team/web-app.git
    responsibility: Interfaz de usuarios
  - name: order-api
    type: backend
    url: https://github.com/your-team/order-api.git
    responsibility: Gestión de órdenes
  - name: market-data
    type: backend
    url: https://github.com/your-team/market-data.git
    responsibility: Publicación de datos de mercado
```

Es un ejemplo ficticio, no un inventario incluido. Puedes añadir `technology`
si está confirmada. No guardes rutas personales ni credenciales. OpenSpec no
interpreta este archivo: no clona repositorios, no concede acceso y no conecta
por sí mismo los consumidores.

En `openspec/config.yaml`, reemplaza la primera frase de `context: |` que
describe S0 Core por una descripción de tu sistema. **Conserva** `schema:
spec-driven`, el resto de instrucciones pertinentes y las reglas nativas.
El resto del bloque remite a `context/project.md`; no dupliques allí todo el mapa.
Esta personalización es parte de la adopción manual. Project Discovery nunca
modifica ese archivo ni añade instrucciones para generar artefactos.

Adapta también el inicio del README de la instancia: nombre, propósito, ID, URL
y enlace directo al [recorrido de incorporación](#join-team). Así PO y QA no
confundirán el S0 del sistema con una plantilla que aún deben configurar.
Conserva esta guía como referencia. `VALIDATION.md` registra pruebas del core;
no demuestra que tu producto se haya probado. Identifica la versión/commit de
la plantilla adoptada y registra la validación de tu sistema por separado.

### Paso 6. Conecta los repositorios de código

Sigue [Conectar cada repositorio](#connect-code) para cada Git participante.
Hazlo una vez por repositorio y publica sus cambios. En un sistema con veinte
repositorios puedes incorporarlos progresivamente, dejando claro cuáles ya
están conectados. No hace falta que PO o QA descarguen esos veinte repositorios
para leer el contexto o preparar casos.

### Paso 7. Publica el S0 configurado

Desde la raíz S0, revisa el resultado y prepara sólo los archivos de esta adopción:

```powershell
Set-Location D:\s0-my-system
openspec store doctor my-system
openspec status --store my-system
git diff --check
git diff
git add .openspec-store/store.yaml context/project.md openspec/config.yaml README.md
git commit -m "chore: initialize system S0"
git push -u origin s0/setup-my-system
```

Si creaste `repositories.yaml`, añádelo explícitamente antes del commit. En
GitHub abre el PR de esa rama hacia `main`, revisa identidad, contexto y enlaces
y fusiónalo según el proceso del equipo. Después:

```powershell
git switch main
git pull --ff-only
```

Entrega al equipo la URL del **S0 del sistema**, su ID y el recorrido 3. La
configuración está compartida cuando se publica y se integra en la rama que
el equipo va a clonar; un cambio sólo local no llega a los demás.

**Al terminar:** el S0 tiene identidad y contexto propios publicados; cada
consumidor incorporado resuelve ese S0; los archivos de integración y sus
instrucciones se revisaron en los PR correspondientes. Esto prepara el trabajo;
no acredita todavía un cambio funcional completo.

<a id="connect-code"></a>
## Conectar cada repositorio de código

Este procedimiento sirve tanto para código existente como para los repositorios
que crees más adelante. Presupone el S0 clonado y registrado en tu máquina.
Lo ejecuta una vez quien prepara **cada repositorio**, desde su raíz Git.

### A. Revisa lo que ya existe

Abre la raíz del repositorio elegido, comprueba su remoto, rama y cambios con
`git remote -v` y `git status`. Trabaja en una rama para esa integración. Conserva
el trabajo previo del equipo; no hagas un commit de todo lo pendiente.

Si ya hay `openspec/config.yaml`, `config.yml`, `specs/` o `changes/` dentro de
`openspec/`, léelos antes de continuar. Si ya apunta al ID correcto, no necesitas
reescribir el puntero. Si hay specs o cambios locales, planifica su migración
al S0: conserva contenido e historial, resuelve capacidades duplicadas y valida
las referencias. Este bloque de instalación inicial se detiene ante ese caso;
no sirve como migrador de un OpenSpec anterior.

Revisa además `AGENTS.md`, `CLAUDE.md` y sus variantes dentro de componentes.
Pueden contener rutas y reglas de un proceso anterior. Conserva las convenciones
de código válidas y actualiza las referencias al contexto del nuevo S0.

### B. Añade el puntero, si no hay OpenSpec previo

Ejemplo: ya estás situado en la raíz del repositorio de código elegido. Cambia
`my-system` por el ID compartido. El control evita sobrescribir una configuración
o planificación anterior:

```powershell
$s0Id = 'my-system'
if ((Test-Path -LiteralPath openspec/config.yaml) -or
    (Test-Path -LiteralPath openspec/config.yml) -or
    (Test-Path -LiteralPath openspec/specs) -or
    (Test-Path -LiteralPath openspec/changes)) {
    throw 'Hay OpenSpec previo: revisa su contenido antes de conectar este repositorio.'
}
New-Item -ItemType Directory -Path openspec -Force | Out-Null
Set-Content -LiteralPath openspec/config.yaml -Value "store: $s0Id" -Encoding utf8
```

El archivo del consumidor queda así:

```yaml
store: my-system
```

El schema, contexto y reglas siguen en el S0. No crees `openspec/specs/` ni
`openspec/changes/` en el consumidor: una raíz local tiene precedencia sobre
el puntero. Esta separación está documentada en
[Stores de OpenSpec](https://openspec.dev/docs/stores).

### C. Genera las integraciones oficiales y comprueba la conexión

La plantilla ya trae integraciones en el S0. Este paso las añade **al repositorio
de código**, para trabajar desde allí con los workflows del agente.
Comprueba primero `openspec config get delivery`. Si quieres skills y comandos
como en el core, usa `openspec config set delivery both`; es una preferencia local
de esa instalación de CLI, no un cambio en la configuración compartida del S0.

```powershell
openspec init --tools "claude,codex" --profile core --no-animation
openspec context --json
openspec status
```

**En PowerShell conserva las comillas en `"claude,codex"`.** Sin ellas un wrapper
`.ps1` puede recibir `claude codex` como un único nombre inválido. Puedes elegir
sólo `"codex"` o `"claude"` si ese repositorio sólo necesita una integración.

Resultado esperado: `context` muestra `root.path` de tu S0, `source: declared`
y el ID correcto. `members: []` es normal sin Worksets; no significa que falten
repositorios en el sistema. El puntero permanece y los artefactos OpenSpec se
guardan en el S0. Que los archivos estén generados no prueba que el cliente ya
los haya seleccionado: comprueba [la skill local](#local-skills) al abrir sesión.

### D. Alinea instrucciones y comparte la integración

El puntero no carga automáticamente `AGENTS.md` ni las skills propias del S0
en una sesión iniciada en el código. En las instrucciones existentes del
consumidor incorpora el criterio siguiente, adaptando el ID y preservando las
convenciones de desarrollo que ya sean válidas:

```text
El contexto y las especificaciones compartidas están en el S0 my-system.
Desde esta raíz, ejecuta openspec context --json y lee root.path.
Lee AGENTS.md y context/project.md en esa raíz antes de planificar cambios.
Las specs y los artefactos del cambio se mantienen allí; el código y sus tests,
en los repositorios afectados. Si usas una skill propia del S0, lee su fuente
.agents/skills/<skill-name>/SKILL.md en esa raíz.
```

Es una instrucción para el agente, no un script. Revisa también instrucciones
anidadas que remitan al proceso anterior. `openspec init` genera integraciones;
no reconcilia esas referencias personalizadas por ti.

Revisa `git diff` y los archivos nuevos. Añade expresamente el puntero, las
integraciones oficiales generadas y los archivos de instrucciones que realmente
adaptaste. Publica un commit y PR en **ese repositorio de código**; enlázalo desde
la adopción del S0. No incluyas cambios previos ajenos ni configuraciones personales.

Repite A–D en los demás Git del sistema, usando el mismo ID. En un monorepo
conectar la raíz basta; sus carpetas de servicios no requieren otro puntero.

### Implementar un cambio que afecta a varios Git

Las tareas del cambio deben identificar el repositorio de cada modificación.
Clona los repositorios afectados y da al agente acceso al S0 y al código necesario.
Desde cada consumidor usa el cambio compartido y delimita qué tareas pertenecen
a ese repositorio. Los PR de código y resultados de tests se enlazan al cambio S0.

OpenSpec no distribuye automáticamente las tareas entre repositorios. En 1.14.3,
el `actionContext` de `status --change <change-id> --json` declara el alcance
de edición esperado para el agente; no es un control de permisos del filesystem.
Desde un consumidor declarante incluye ese consumidor y su S0;
seleccionar otro store desde fuera no incorpora por sí solo todo el código del
sistema. Si faltan raíces, configura el acceso y el contexto de trabajo antes de
implementar allí. Un workspace o Workset ayuda a abrir carpetas; no concede
permisos ni hace que todas las tareas pertenezcan al repo actual.

<a id="new-system"></a>
## 2. Empezar un proyecto nuevo, todavía sin código

Puedes comenzar desde S0 sin crear primero un frontend, una API o veinte
repositorios vacíos. El S0 reúne la necesidad y las decisiones iniciales; el
código se incorpora cuando exista.

### Paso 1. Crea el S0 y su identidad

Sigue los **pasos 2, 3 y 4 del recorrido 1**: botón de GitHub Template, clon del
nuevo repositorio, rama de preparación y registro con un ID propio. Usa un nombre
de sistema que no se confunda con la plantilla. Los requisitos de CLI son los
mismos para la persona que hace esta preparación.

### Paso 2. Escribe un contexto inicial honesto

Completa `context/project.md` con el problema, usuarios previstos, alcance inicial
y preguntas abiertas. Declara que el producto está por construirse. Las decisiones
propuestas son propuestas: no describas componentes, contratos ni funcionalidades
como implementados cuando aún no existen.

Si todavía no sabes cuántos repositorios habrá, deja esa decisión pendiente. No
necesitas `repositories.yaml`, URLs inventadas ni una arquitectura completa para
empezar. Adapta la frase de `context` en `openspec/config.yaml` y el inicio del
README como explica el paso 5. Publica la preparación con el **paso 7**; omite la
conexión a código hasta disponer de esos repositorios.
Project Discovery no es un requisito previo para iniciar este proyecto o proponer
su primer cambio; está orientado especialmente a comprender sistemas existentes.

### Paso 3. Define y comparte el primer cambio desde el S0

Después de publicar la preparación, crea una rama en el S0 para la propuesta:

```powershell
git switch -c s0/first-change
```

Abre la carpeta S0 con tu agente y comprueba las [skills locales](#local-skills).
Puedes empezar con `refine-user-story`, aportando una necesidad real. Para crear
los artefactos OpenSpec, usa `openspec-propose` de este repositorio y describe el
objetivo y el alcance. Por ejemplo, en el chat:

```text
Lee AGENTS.md y context/project.md. Usa la skill local
.agents/skills/openspec-propose/SKILL.md para proponer este cambio:
[necesidad, usuarios y alcance que queremos construir].
Identifica las decisiones pendientes y los repositorios previstos por tarea.
Prepara la propuesta; la implementación vendrá después.
```

Sustituye el texto entre corchetes por tu necesidad. La CLI debe estar disponible
para este workflow. El resultado se crea en `openspec/changes/` del S0; no es
una aplicación ejecutable ni acredita requisitos ya implementados.

Revisa los artefactos generados. Sustituye `first-change` en la ruta siguiente
por el ID que haya creado el workflow:

```powershell
git diff --check
git status --short
git add openspec/changes/first-change
git commit -m "docs: propose first system change"
git push -u origin s0/first-change
```

Abre el PR de la propuesta en el S0 y comparte su enlace o rama con PO/QA para
revisión. Hasta integrarlo en `main`, deben consultar esa rama para recibirlo.
Los cambios en archivos no publicados todavía sólo existen en tu copia local.

### Paso 4. Crea y conecta el código cuando corresponda

Crea los repositorios de producto mediante el proceso habitual del equipo.
Añade sus URLs y responsabilidades al contexto. Aplica [Conectar cada
repositorio](#connect-code) a cada Git; usa el mismo ID del S0 ya creado.

Con repositorios accesibles y tareas delimitadas, solicita la implementación.
El código y los tests van allí; contexto, specs y planes continúan en el S0.
PO y QA pueden incorporarse desde el paso 2 publicado, sin esperar a que exista
todo el código.

**Al terminar:** tienes un S0 independiente para planificar el nuevo sistema.
Cuando comience el desarrollo, sus repositorios consumen esa misma fuente.

<a id="join-team"></a>
## 3. Usar un S0 que el equipo ya configuró: PO, QA y nuevos integrantes

Para entender el sistema, refinar historias o preparar casos, obtén el **S0 del
sistema** y ábrelo con tu agente. No crees otro repositorio desde la plantilla,
no regeneres su identidad y no repitas `openspec init`.

### Paso 1. Descarga la copia que vas a revisar

El equipo te comparte la URL del S0 y la rama, PR o revisión de interés. Puedes
leer directamente en GitHub. Para trabajar con un agente local elige:

- **Clon Git**, recomendado si vas a recibir actualizaciones o contribuir.
- **ZIP**, si sólo quieres una copia de esa revisión sin instalar Git.

Con Git, sustituye la URL y ejecuta:

```powershell
git clone https://github.com/your-team/s0-my-system.git s0-my-system
Set-Location s0-my-system
```

Para revisar una propuesta aún en PR, el equipo debe indicar su rama; puedes
clonarla con `git clone --branch branch-to-review <URL-del-S0> s0-my-system`,
sustituyendo ambos valores. `main` no contiene cambios que sólo estén en otra rama.

Con ZIP, abre la rama o revisión acordada en GitHub y usa **Code → Download ZIP**;
extrae su contenido. Conserva `.agents/` y `.claude/` junto al resto de archivos.
Son parte del S0, aunque el explorador oculte carpetas que empiezan con punto.

### Paso 2. Abre la raíz del S0 con tu agente

Abre la carpeta que contiene `AGENTS.md`, `context/` y `openspec/`. Selecciónala
como proyecto/carpeta de trabajo en tu cliente e inicia una sesión con acceso a
esos archivos. No abras sólo `context/` ni la carpeta de código para este recorrido.

Ya recibes instrucciones y métodos versionados: `AGENTS.md` es común;
`CLAUDE.md` lo referencia; las dos skills PO/QA viven en `.agents/skills/` y
sus entradas Claude apuntan a esas mismas fuentes. No se instalan globalmente.
Si no aparecen o seleccionas otra copia, usa la [comprobación de rutas](#local-skills).

No necesitas instalar OpenSpec ni registrar el store para **leer los documentos,
refinar una historia o preparar casos desde esta carpeta**. Necesitas el agente
compatible y su acceso a los archivos. Tampoco necesitas clonar el código de los
diez o veinte servicios para esas tareas.

### Paso 3. Usa el método según tu trabajo

Para PO, escribe en el chat:

```text
Lee AGENTS.md, context/project.md y la skill local
.agents/skills/refine-user-story/SKILL.md de este S0.
Refina esta necesidad usando las specs relacionadas y devuelve el resultado
en conversación: [texto de la historia o necesidad].
```

Para QA:

```text
Lee AGENTS.md, context/project.md y la skill local
.agents/skills/prepare-tests/SKILL.md de este S0.
Prepara casos para [historia, criterio o cambio identificado].
Indica las fuentes, las dudas y qué casos quedan preparados sin ejecutar.
```

Puedes aportar la historia como texto; Jira no es obligatorio. Si faltan specs,
el agente debe señalarlo y usar sólo las fuentes disponibles. Estos métodos
devuelven resultados en conversación por defecto. Si quieres guardarlos en un
archivo o herramienta, indícalo expresamente; no hay un entregable obligatorio
por rol.

Para implementar o ejecutar automatizaciones QA sí necesitarás acceso al código
o a la suite, entorno, datos y versión que comprobar. Preparar una matriz de
casos no equivale a haber implementado o ejecutado esos tests.

### Paso 4. Mantén tu copia actualizada

El clon refleja lo publicado en la revisión descargada; no se actualiza solo.
Comprueba rama, commit y cambios locales:

```powershell
git status --short --branch
git rev-parse --short HEAD
```

En la rama acordada y después de conservar tu trabajo local, recibe cambios con:

```powershell
git pull --ff-only
```

Si revisas un PR, usa su rama o diff. Con ZIP, descarga una nueva copia cuando
cambie la revisión; no sobrescribas resultados locales que quieras conservar.
Para contribuir archivos al S0, usa una rama y PR en ese mismo repositorio.

### Si después necesitas workflows OpenSpec o trabajar desde código

Esto aplica también a un DEV que se incorpora. Instala la [CLI acordada](#cli-setup),
clona el S0 del sistema y, desde su raíz, registra **la identidad que ya trae**:

```powershell
openspec store list
openspec store register .
openspec store doctor my-system
openspec status --store my-system
```

Sustituye `my-system` por el ID que figura en `.openspec-store/store.yaml`. Se
registra ese checkout una vez por máquina/entorno para poder resolver el ID;
no vuelvas a generar su identidad. Un ID resuelve una sola ruta por registro:
si ya aparece, comprueba que corresponde a este clon antes de cambiarlo.
Desde la propia raíz S0 la CLI puede resolver el proyecto directamente; el
registro es lo que permite encontrarlo por ID desde un consumidor o `--store`.

Clona los repositorios de código necesarios. Si el equipo publicó sus punteros
e integraciones, ya los recibes: comprueba desde cada raíz `openspec context
--json`, sin repetir la preparación del sistema. Una ruta local distinta de la
de tus compañeros es válida; el ID compartido sigue siendo el mismo.

<a id="local-skills"></a>
## Comprobar que usas la skill del repositorio

La CLI instalada, las skills personales y las skills de proyecto son cosas
distintas. `openspec --version` comprueba la CLI; no dice qué `SKILL.md` seleccionó
el agente. Una ruta como `C:\Users\usuario\.agents\skills\openspec-propose\SKILL.md`
es personal. Una ruta dentro de `D:\s0-my-system\.agents\skills\...` pertenece
a ese S0; dentro de un consumidor es la integración de ese consumidor.

Codex busca `.agents/skills/` en el proyecto y también en ubicaciones personales.
Si dos skills tienen el mismo nombre, no las fusiona y ambas pueden aparecer.
Comprueba la ruta seleccionada; si una actualización no aparece, reinicia el
cliente. Véase [documentación oficial de skills de OpenAI](https://learn.chatgpt.com/docs/build-skills).

Claude Code usa `.claude/skills/` en el proyecto y también puede tener skills
personales. Una coincidencia de nombre puede dar prioridad a la personal; no
supongas que abrir el proyecto siempre selecciona la local. Véase
[skills y resolución de nombres en Claude Code](https://code.claude.com/docs/en/skills).

En ambos casos puedes pedir explícitamente:

```text
Usa la skill de este repositorio en
.agents/skills/refine-user-story/SKILL.md.
Indica la ruta que leíste y utiliza AGENTS.md y context/project.md de este S0.
```

Para una skill oficial cambia el nombre por el correspondiente. En Claude puedes
leer su entrada `.claude/skills/`; los puentes propios remiten a `.agents/skills/`.
No borres ni dupliques skills globales para resolver una selección puntual.

Si trabajas desde código, resuelve el S0 con `openspec context --json` y pide
leer la fuente necesaria bajo `root.path`. Registrar un store no instala sus
skills propias en el consumidor ni garantiza que aparezcan en su selector.
Otros agentes pueden leer esos archivos; su descubrimiento automático necesita
comprobarse en cada cliente. Cowork/Chat y sesiones sin acceso al filesystem
requieren su propia integración; esta guía no presume esa carga.

<a id="project-discovery"></a>
## Project Discovery: preparar contexto antes de proponer

Usa esta skill cuando necesites entender un sistema existente o actualizar el
conocimiento de algunos componentes. Funciona con uno o múltiples repositorios,
incluidos monorepos. Es un paso opcional, separado de los workflows de OpenSpec.

1. Abre el S0 del sistema con tu agente y asegúrate de usar la skill local.
2. Indica qué repositorios puede leer y qué quieres comprender. No hace falta
   revisar todo el sistema para actualizar un solo componente.
3. El agente analiza, cita fuentes y presenta un borrador con archivos destino,
   hechos, inferencias y preguntas. Durante esa fase los repositorios quedan
   sin modificaciones.
4. Revisa el borrador. Cuando apruebes su contenido y rutas, el agente lo guarda
   únicamente en `context/` y comprueba que no cambió archivos fuera del alcance.

En Codex usa `$project-discovery`; en Claude Code, `/project-discovery`. También
puedes escribir en el chat, sustituyendo las rutas de ejemplo:

```text
Usa Project Discovery de este S0. Puedes leer D:\my-system-web y
D:\my-system-api para comprender la arquitectura y los componentes existentes.
Prepara un borrador de contexto con sus fuentes y las dudas pendientes.
```

Para actualizar selectivamente:

```text
Usa Project Discovery para revisar sólo el componente de notificaciones en
D:\my-system-notifications. Presenta el diff propuesto para
context/components/notifications.md y los enlaces del índice que deban cambiar.
```

La fuente común está en `skills/project-discovery/SKILL.md`; `.agents/skills/`
y `.claude/skills/` contienen puentes. Si no aparece en tu selector, pide leer
explícitamente esa fuente. Mantén los tres directorios al copiar la skill a una
instancia existente; los recursos del método viven junto a su fuente común.

El contexto puede organizarse en `context/project.md`, `context/architecture.md`
y `context/components/`, sin exigir todos esos documentos en cada sistema.
Discovery describe lo observado; no convierte código en requisitos aprobados.
No toca `openspec/config.yaml`, schemas, specs, changes, identidad ni workflows
oficiales. Después puedes pedir una propuesta con OpenSpec y consultar el
contexto relevante; no se inicia una propuesta automáticamente.

La publicación requiere **Git y Python >=3.10** para el verificador incluido,
sin paquetes adicionales. El [procedimiento de comprobación](skills/project-discovery/references/scope-checks.md)
captura una referencia antes del análisis y compara la escritura posterior
con las rutas aprobadas. Si falta una herramienta o una fuente sólo está en ZIP,
la skill puede entregar un borrador con sus límites, sin acreditar una publicación
verificada. La aprobación del contenido sigue siendo humana: el helper sólo
comprueba archivos y estado Git.

Al adoptar la skill en un S0 creado antes de esta versión, incorpora sus archivos
y el enlace de instrucciones mediante un PR selectivo. No vuelvas a crear el S0
ni reemplaces su contexto o configuración. La incorporación y las comprobaciones
del core se registran en [VALIDATION.md](VALIDATION.md).

<a id="daily-work"></a>
## Después de instalar: un cambio de PO a QA

| Paso | Trabajo | Dónde queda |
| --- | --- | --- |
| PO refina | Necesidad, alcance, criterios y dudas | Conversación, HDU o fuente acordada |
| DEV propone | `proposal.md`, `design.md`, `tasks.md` y deltas de specs | `openspec/changes/<change>/` en el S0 |
| Equipo revisa | Alcance y decisiones del cambio | Rama y PR del S0 |
| DEV implementa | Código y tests, con tareas por repositorio | Repositorios de código y sus PR |
| QA prepara/verifica | Casos; automatizaciones y ejecuciones cuando se soliciten | Conversación/herramientas y suite correspondiente |
| Equipo consolida/cierra | Specs vigentes e historial | Workflows `sync` y `archive` del cambio S0 |

Usa las [invocaciones del README](README.md#trabajar-con-las-skills) en el chat
del agente. Los seis workflows oficiales requieren CLI. `sync` es un workflow
del agente: no existe un comando de terminal `openspec sync`.

Enlaza la solicitud, el cambio S0, los PR de código y los resultados de pruebas.
Identifica commit/build, entorno y datos cuando se hayan ejecutado tests.
Las aprobaciones y el merge siguen el proceso Git del equipo; preparar
documentos o pasar la validación estructural no acredita aceptación del producto.

<a id="maintenance"></a>
## Mantenimiento y problemas habituales

| Situación | Qué comprobar |
| --- | --- |
| Instalé 1.14.3 pero sale 1.13.2 | Ejecutable efectivo y PATH en [Preparar la CLI](#cli-setup) |
| `Invalid tool(s): claude codex` | Usa `--tools "claude,codex"` con comillas en PowerShell |
| El consumidor no encuentra el store | Clona el S0 del sistema y registra su identidad existente en esa máquina |
| `context` resuelve el código en vez del S0 | Revisa raíces locales `specs/changes` y configuración anterior; no borres historia para forzar el puntero |
| El agente cita una skill global | Comprueba la ruta y solicita la fuente de proyecto en [Skills locales](#local-skills) |
| PO/QA no ve lo que hizo DEV | Comprueba que se publicó el cambio, rama/PR correctos y actualización del clon |
| El agente sigue el proceso antiguo | Revisa instrucciones raíz y anidadas del consumidor; `init` no las reconcilia |
| Mejoró el core pero mi sistema sigue igual | Las instancias son independientes; incorpora el diff necesario mediante PR |

Desde una instancia registrada, sustituye el ID en estas comprobaciones:

```powershell
openspec store doctor my-system
openspec list --store my-system
openspec list --specs --store my-system
openspec validate --all --strict --no-interactive --store my-system
openspec doctor --store my-system
```

Listados vacíos son correctos si aún no hay specs ni cambios. Para actualizar
integraciones oficiales con la CLI acordada revisa primero `openspec update
--dry-run`. Al cambiar de versión, prueba registro, resolución desde consumidores,
generación de integraciones y un ciclo de cambio. No edites manualmente workflows
generados. Conserva los métodos propios y verifica sus puentes.

Las mejoras de `s0-core-template` no se propagan solas. Compara el diff del maestro
con la versión adoptada y lleva los cambios pertinentes a un PR de la instancia.
Conserva su identidad, contexto, inventario, specs, historial y personalizaciones.
`git pull` actualiza **tu repositorio**; `openspec update` regenera integraciones
oficiales. Ninguno distribuye las mejoras propias de S0 Core a sus instancias.

### Si adoptaste una versión con nombres anteriores

Aplica estos renombres mediante un PR de la instancia y actualiza los enlaces:

| Nombre anterior | Nombre actual |
| --- | --- |
| `contexto/proyecto.md` | `context/project.md` |
| `refinar-hdu` | `refine-user-story`, incluidas carpetas, campo `name` y puentes |
| `preparar-pruebas` | `prepare-tests`, incluidas carpetas, campo `name` y puentes |
| `repositorios.yaml`, si existe | `repositories.yaml` |
| `VALIDACION.md` | `VALIDATION.md` |

En el inventario opcional usa `repositories`, `name`, `type`, `technology` y
`responsibility` en lugar de `repositorios`, `nombre`, `tipo`, `tecnologia` y
`responsabilidad`; `url` se conserva. Mantén los valores propios del sistema.
No necesitas crear un inventario para adoptar los nombres nuevos.

Consulta el [registro de validación](VALIDATION.md) para distinguir lo comprobado
de las pruebas de clientes y onboarding todavía pendientes. La referencia de
comandos es la CLI fijada y las fuentes oficiales de
[Stores](https://openspec.dev/docs/stores), [CLI](https://openspec.dev/docs/cli),
[Profiles](https://openspec.dev/docs/profiles) y
[configuración](https://openspec.dev/docs/configuration/config-yaml).
