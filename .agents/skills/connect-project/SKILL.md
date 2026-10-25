---
name: connect-project
description: Conectar y configurar una instancia S0 ya clonada con un proyecto, monorepo o varios repositorios. Usar al pedir enlazar el S0 con el código o preparar su adopción; comprueba OpenSpec, identidad e integraciones y continúa con un borrador de Project Discovery.
---

# Connect Project

Recibe el S0 actual y las rutas del código. Resuelve la preparación técnica con
las herramientas del agente y OpenSpec oficial; el usuario no tiene que ejecutar
la secuencia manual. Esta es la fuente común para Codex, Claude y otros agentes.

## Alcance de esta petición

La creación desde GitHub Template y el clon los hace el usuario. Trabaja sobre
esa instancia independiente: nunca conviertas `s0-core-template` en el S0 de un
producto. Comprueba raíz Git, remoto e instrucciones antes de modificar archivos.
Si el destino es el maestro o no puedes identificar el S0, comunica el dato que
falta antes de preparar su identidad.

Pedir la conexión autoriza configurar el S0 y los consumidores indicados, registrar
el store e instalar OpenSpec 1.14.3 si están disponibles sus requisitos. No pide
commits, cambios de rama, stage, push, PR, clones adicionales ni migraciones.
Deja las diferencias locales para revisión y conserva el trabajo previo.
No escribas código de producto ni crees specs, cambios o schemas. No añadas
scripts, inventarios obligatorios, archivos de estado ni carpetas auxiliares al S0.

## 1. Comprueba el alcance y las herramientas

- Usa sólo las rutas aportadas. Identifica sus raíces con `git rev-parse
  --show-toplevel`, comprueba su estado y elimina duplicados: frontend y backend
  dentro de un único Git requieren una sola conexión.
- Si la ruta es una carpeta agrupadora, busca Git dentro de ella, sin atravesar
  enlaces, `.git`, dependencias o caches ni salir a carpetas vecinas. Al encontrar
  una raíz Git, úsala como unidad; no conviertas automáticamente sus submódulos
  o repositorios referenciados. Si la selección es ambigua, pregunta por las raíces
  que pertenecen al sistema. Excluye el propio S0 de la lista de consumidores;
  otro S0 encontrado no se convierte en repositorio de código. Una ruta
  inaccesible se informa como bloqueada.
- Lee instrucciones y configuración existentes antes de escribir. Un árbol sucio
  no impide trabajar si puedes preservar sus cambios. No hagas reset, limpieza
  ni sustitución completa de archivos personalizados.
- Comprueba Git y las instalaciones de OpenSpec. Ejecuta `--version` sobre los
  candidatos reales; una instalación correcta puede estar detrás de otra en PATH.
  En Windows usa `Get-Command openspec -All` y, si hay npm, su prefijo global y
  `openspec.cmd`. Reutiliza una copia que devuelva exactamente **1.14.3**.
- Si falta esa versión, comprueba Node **>=20.19.0** y npm; cuando estén disponibles,
  ejecuta `npm install -g @fission-ai/openspec@1.14.3` y verifica su ejecutable.
  Usa la ruta comprobada o el PATH de esta sesión durante todos los pasos.
  Los ajustes de PATH se limitan a la sesión; no retires otros wrappers para
  resolver su precedencia.
  Si falta Git o Node/npm, informa el requisito concreto; no instales otras
  herramientas del sistema por inferencia ni finjas una configuración completa.

## 2. Resuelve la identidad sin sustituir otro sistema

Lee `.openspec-store/store.yaml`, el remoto del S0, su configuración y
`store list --json` usando la CLI comprobada. Conserva una identidad ya adoptada:
registra ese checkout con `store register .` si hace falta, sin regenerar metadata.
Si el ID solicitado contradice esa identidad, informa el conflicto antes de
conectar consumidores; esta skill no renombra sistemas existentes.

Sólo en una adopción nueva confirmada desde plantilla, con remoto propio,
contenido inicial genérico y sin specs ni cambios de producto, sustituye el
marcador heredado. `id: s0-core` por sí solo no demuestra que sea descartable.
Deriva el ID del nombre del repositorio remoto quitando `s0-`, salvo que el usuario
lo haya indicado. Si no es válido o resulta ambiguo, pide ese dato.

Comprueba primero que el ID no esté registrado en otra ruta ni la ruta bajo otro
ID. No desregistres asociaciones ajenas para resolver conflictos. Retira sólo
el marker heredado, conservando su contenido temporalmente fuera del repositorio,
y deja crear la identidad oficial con `store register . --id <id> --yes`.
Si falla, restaura únicamente ese marker que acabas de retirar y explica el error;
no continúes suponiendo que quedó registrado. No uses `git init` ni edites el ID
a mano: 1.14.3 rechaza `--id` distinto cuando ya existe identidad.

En una instancia recién adoptada, cambia el título/descripción inicial del README
y la frase genérica que identifica la plantilla en `openspec/config.yaml` por
el nombre confirmado del sistema. Preserva schema, reglas y demás propiedades;
no añadas instrucciones técnicas ni dupliques el contexto funcional. Conserva
descripciones propias existentes. Project Discovery prepara después `context/`.

Comprueba también las integraciones oficiales del propio S0 con el procedimiento
de generación/verificación siguiente. Si faltan, ejecútalo desde su raíz,
conservando su configuración, specs, cambios y skills propias. El S0 mantiene
su raíz OpenSpec: no se sustituye su configuración por un puntero a sí mismo.

## 3. Conecta cada consumidor compatible

En cada raíz Git seleccionada, aplica esta decisión antes de ejecutar `init`:

| Estado encontrado | Acción |
| --- | --- |
| Sin configuración ni specs/changes locales | Crea `openspec/config.yaml` con `store: <id>` |
| Configuración sólo de puntero al mismo S0 | Consérvala byte a byte y comprueba las integraciones |
| Otro store, configuración propia, YAML inválido o specs/changes locales | Informa el conflicto; no conviertas ni borres esa planificación |

Considera también `config.yml`; no crees un segundo archivo para ocultar el
existente. Detecta los conflictos antes de escribir en ese consumidor. Puedes
continuar con consumidores compatibles; el resultado debe distinguir los
conectados de los pendientes y nunca presentarse como completo si falta alguno.

Genera las integraciones que falten con la CLI oficial, desde esa raíz:

```powershell
openspec init --tools "claude,codex" --profile core --no-animation
```

En PowerShell conserva las comillas. En la ejecución real sustituye `openspec`
por el ejecutable 1.14.3 comprobado. No uses `--force` ni `--language` sobre un
consumidor. No crees `openspec/specs/` o `openspec/changes/` allí.

Para obtener skills y comandos sin cambiar preferencias persistentes, ejecuta
`init` en un proceso con `XDG_CONFIG_HOME` temporal, inicialmente vacío y fuera
de los repositorios. En ese mismo entorno aislado ejecuta primero
`config set delivery both` con la CLI comprobada: no dependas del valor por
defecto, pues la migración puede inferirlo de integraciones existentes. Mantén
`XDG_DATA_HOME` y el registro del usuario que acabas de utilizar. Si modificas
variables en la sesión, conserva los valores originales y restáuralos con
`finally`, incluso ante un error. No cambies `delivery` o `defaultStore` en la
configuración persistente del usuario. El directorio temporal no se versiona.

En una reejecución, omite `init` si ya están las integraciones oficiales 1.14.3
esperadas: seis skills por cliente y seis comandos Claude del perfil core.
Comprueba versión, contenido y errores, no sólo la existencia de directorios.
Si faltan archivos, inicializa los pendientes después de comprobar que no se
sobrescribirán adaptaciones. Si hay integraciones personalizadas incompatibles,
informa el bloqueo y conserva sus contenidos; no las regeneres a ciegas.

## 4. Alinea las instrucciones y verifica

Aplica las escrituras de instrucciones sólo a consumidores compatibles cuya
conexión al S0 elegido esté comprobada. Los consumidores bloqueados conservan
también sus instrucciones, sin añadirles el bloque de otra conexión.
En el `AGENTS.md` de cada consumidor conectado incorpora este bloque,
reemplazando sólo el ID:

```text
<!-- s0-connection:start -->
El contexto y las especificaciones compartidas de este sistema están en el S0 <id>.
Usa OpenSpec 1.14.3; comprueba el ejecutable efectivo si hay varias instalaciones.
Desde esta raíz, ejecuta openspec context --json y lee root.path.
Antes de planificar, lee AGENTS.md y context/project.md en esa raíz resuelta.
Mantén las specs y los artefactos OpenSpec allí; el código y sus tests van en
los repositorios afectados. Lee las skills propias desde la raíz S0 cuando
corresponda; registrar el store no las instala en este consumidor.
<!-- s0-connection:end -->
```

Créalo si falta, añade el bloque si no existe y actualiza sólo ese bloque en
reejecuciones. Conserva las convenciones, contenido y modificaciones ajenas.
Para Claude crea `CLAUDE.md` con `@AGENTS.md` si falta; si ya existe, preserva su
contenido y añade esa importación sólo si aún no carga las instrucciones locales.
Revisa las instrucciones anidadas pertinentes y corrige referencias de conexión
obsoletas dentro del alcance, sin reescribir reglas de producto por suposición.
Si un archivo inaccesible o un conflicto impide esa alineación, informa el límite.

Desde el S0 ejecuta `store doctor <id>` y `status --store <id>`. Desde cada
consumidor ejecuta `context --json` y `status`: la raíz debe ser exactamente el
S0 elegido. Comprueba los punteros conservados, las integraciones esperadas y
la ausencia de specs/changes locales. Revisa diferencias y archivos nuevos con
Git y `git diff --check`; no hagas stage para obtener el diff de archivos nuevos.

Una segunda ejecución reutiliza identidad, registro, punteros e instrucciones
correctos y completa únicamente lo pendiente. Ante un fallo conserva las partes
correctas y el trabajo previo; explica qué está conectado, qué falta y la causa.
No hagas rollback general ni elimines fuentes para obtener una comprobación verde.

## 5. Continúa con el borrador de contexto

Entrega un resumen breve de S0/ID, raíces únicas, conexiones y bloqueos, versión
de CLI usada y archivos modificados. Luego lee y utiliza la skill local
[project-discovery](../project-discovery/SKILL.md), pasando el S0 y los repositorios
autorizados accesibles. No presupongas que su entrada aparece en otro cliente.

El snapshot de Discovery se captura **después** de terminar la configuración:
los cambios técnicos locales pasan a ser su referencia y Discovery sólo puede
publicar el contexto aprobado. No reduzcas sus protecciones ni escribas contexto
en nombre de la conexión para saltarte la revisión. Presenta el borrador y pide
su aprobación conforme a esa skill; no inicies `propose` automáticamente.

Sin código todavía, deja la identidad y los datos iniciales preparados y utiliza
sólo el propósito que haya aportado el usuario para el borrador. No inventes
componentes, repositorios o arquitectura. Si falta Python para el verificador
de Discovery, informa el límite y entrega el borrador de lectura; la conexión
técnica no depende de Python y no instala ese runtime automáticamente.
