# Recorrido de instalación: SummitApp

Actualizado el **24 de octubre de 2026**.

Este registro histórico muestra cómo se adoptó S0 Core para un sistema existente
en Windows. **Los bloques describen la preparación ya realizada; no deben
reejecutarse sobre esta instalación adoptada.** Para una instalación nueva o
incorporarse a un equipo, sigue [INSTALLATION.md](../INSTALLATION.md). El alcance
del núcleo está en [README.md](../README.md).

El S0 está en `D:\s0-summitapp`; el código existente, en `D:\SummitApp`. Son
repositorios Git independientes. La aplicación es un monorepo con Flutter en
`mobilApp/` y Spring Boot en `summit-backend/`; ambos componentes comparten el S0.

## 1. Crear y clonar el repositorio del sistema

En el navegador se abrió `fdelvalle01/s0-core-template` y se eligió
**Use this template → Create a new repository**. El destino fue
<https://github.com/fdelvalle01/s0-summitapp>, usando la rama predeterminada.
Se clonó el repositorio nuevo:

```powershell
git clone https://github.com/fdelvalle01/s0-summitapp.git D:\s0-summitapp
Set-Location D:\s0-summitapp
```

Resultado observado: `origin` apunta a `fdelvalle01/s0-summitapp`. La plantilla
aportó instrucciones, skills y estructura; SummitApp tiene su propio remoto,
contexto, identidad e historial.

## 2. Instalar la CLI y corregir su precedencia en PowerShell

Un wrapper de Brain aparecía primero en `PATH` y devolvía OpenSpec 1.13.2. Se
instaló la versión acordada por npm y se dio precedencia a su directorio en la
sesión de PowerShell:

```powershell
npm install -g @fission-ai/openspec@1.14.3
$npmBin = (npm prefix -g).Trim()
$env:Path = "$npmBin;$env:Path"
openspec --version
```

Resultado ejecutado: `openspec --version` devolvió `1.14.3`. El ajuste mostrado
afecta esa sesión; no modifica permanentemente el `PATH` de Windows. La CLI
global anterior y sus skills 1.13.2 permanecieron separadas de las integraciones
de proyecto 1.14.3. No fue necesario desinstalar Brain ni borrar skills globales.

Para diagnosticar una futura discrepancia se sugiere `Get-Command openspec -All`:
permite ver qué ejecutable resuelve PowerShell. Esta sugerencia no es una nueva
comprobación ejecutada sobre la instalación.

## 3. Crear la rama y asignar la identidad del sistema

Antes de personalizar el clon nuevo se creó la rama de preparación:

```powershell
Set-Location D:\s0-summitapp
git switch -c s0/setup-summitapp
```

El clon heredaba la identidad genérica de la plantilla. Se regeneró una sola vez
mediante el registro oficial, para obtener `id: summitapp`:

```powershell
Get-Content -LiteralPath .openspec-store/store.yaml
Remove-Item -LiteralPath .openspec-store/store.yaml
openspec store register . --id summitapp --yes
openspec store doctor summitapp
openspec status --store summitapp
```

Resultado observado: metadata `version: 1` e `id: summitapp`, doctor sin
incidencias y `No active changes`. **Esta retirada puntual de metadata correspondió al clon nuevo;
no se repite al usar este S0 ni al incorporarse otro integrante.**

En otro equipo sólo hace falta registrar el clon si se utilizará la CLI para
resolver ese ID. Leer contexto/specs como PO o QA no exige instalar la CLI ni
registrar el store. El registro es local: no publica ni sincroniza Git.

## 4. Personalizar el contexto y publicar el S0

El usuario adaptó `context/project.md` y `openspec/config.yaml` con el nombre
SummitApp, sus repositorios y sus componentes Flutter/Spring Boot. El contexto
inicial reconoce que usuarios, funcionalidades, contratos y reglas de negocio
todavía deben completarse desde fuentes verificadas.

Los tres archivos de preparación se publicaron en la rama del S0:

```powershell
git add .openspec-store/store.yaml context/project.md openspec/config.yaml
git commit -m "chore: initialize SummitApp S0"
git push -u origin s0/setup-summitapp
```

El commit resultante fue `7ea5fc8`. En GitHub se abrió y fusionó el
[PR #1](https://github.com/fdelvalle01/s0-summitapp/pull/1).
La inspección posterior de Git mostró `main` y `origin/main` en el merge
`bf68068`, con el árbol del S0 limpio.

Las specs y los cambios activos están vacíos: se adoptó un S0 nuevo, sin migrar
los cambios o la documentación del store anterior. La preparación técnica no
equivale a conocer funcionalmente todo SummitApp.

## 5. Conectar el repositorio de código existente

Desde la raíz Git de la aplicación se creó el puntero:

```powershell
Set-Location D:\SummitApp
New-Item -ItemType Directory -Path openspec -Force | Out-Null
Set-Content -LiteralPath openspec/config.yaml -Value 'store: summitapp' -Encoding utf8
```

Después se instalaron las integraciones oficiales de ambos clientes:

```powershell
openspec init --tools "claude,codex" --profile core --no-animation
openspec context --json
```

El primer intento de `init` falló al transmitir la lista de herramientas sin
comillas. La invocación corregida conserva `"claude,codex"` como un único
argumento de PowerShell y finalizó con éxito usando OpenSpec 1.14.3.

Resultados observados: `context` resolvió `D:\s0-summitapp`; el archivo local
`openspec/config.yaml` contiene sólo `store: summitapp`. No se crearon
`openspec/specs/` ni `openspec/changes/` en la aplicación. Los encabezados de las
skills oficiales locales inspeccionadas contienen `generatedBy: "1.14.3"`.

## 6. Elegir las fuentes correctas al iniciar una sesión

El siguiente uso es recomendado; no acredita una prueba de descubrimiento del
cliente. PO/QA pueden abrir `D:\s0-summitapp` y leer `AGENTS.md`,
`context/project.md` y la fuente pertinente de `.agents/skills/`:
`refine-user-story` o `prepare-tests`. Los puentes Claude apuntan a las mismas
fuentes. DEV necesita acceso autorizado al S0 y al código para implementar.

Una sesión iniciada en la aplicación no hereda automáticamente las skills propias
del store por declarar `store: summitapp`. Debe poder leer las instrucciones y
la skill requerida en la raíz resuelta. La presencia de archivos generados no
demuestra que un cliente los haya descubierto o utilizado.

Para quien vaya a usar la CLI, estas comprobaciones se proponen como seguimiento;
no se presentan aquí como resultados nuevos ya ejecutados:

```powershell
openspec store list
openspec store doctor summitapp
openspec status --store summitapp
```

## Estado y pendientes

- El S0 está publicado y su PR de preparación está fusionado.
- El consumidor resuelve el S0 y tiene integraciones oficiales 1.14.3.
- No se observó todavía un commit o push de los nuevos `openspec/` e integraciones
  de la aplicación. Su rama es `development` y contiene numerosos cambios previos;
  deben revisarse sin incluirlos automáticamente en la adopción.
- Las instrucciones existentes de la aplicación aún mencionan `docs/INDICE.md`,
  `docs/trabajar-hu.md` y su convención previa de `revision.md`. Esas referencias
  requieren reconciliación con `AGENTS.md` y `context/project.md` del nuevo S0.
- No se verificó descubrimiento en una sesión nueva de Codex o Claude Code,
  Claude Desktop Code, ni equivalencia de resultados entre clientes/modelos.
- No se ejecutó una primera HDU completa, automatización de producto, prueba de
  dispositivo ni ciclo de sync/archive sobre SummitApp.

Estos resultados acreditan preparación técnica y publicación del S0. La
aceptación funcional y las pruebas del producto requieren su propia evidencia.
