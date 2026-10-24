# Comprobación de alcance

Usa `scripts/check_discovery_scope.py` desde la carpeta de la fuente común.
Requiere **Git y Python >=3.10**, sólo biblioteca estándar. No requiere npm,
OpenSpec ni acceso de red. En Windows puedes usar `py -3`; en otros entornos,
`python3` con una versión compatible.

El script registra huellas, rutas y estado Git; no imprime contenido de archivos.
Es una comprobación de cambios, no un sandbox, un detector de secretos ni una
prueba de aprobación. No reviertas trabajo ajeno ante una diferencia.

## Antes de leer código

Sitúa la terminal en la raíz S0. Sustituye las rutas por los repositorios ya
autorizados. El JSON debe quedar fuera del S0 y de todos los repositorios fuente;
no se versiona. El nombre único evita reemplazar otra referencia.

```powershell
$discoverySnapshot = Join-Path ([System.IO.Path]::GetTempPath()) ("s0-discovery-" + [guid]::NewGuid().ToString() + ".json")
py -3 skills/project-discovery/scripts/check_discovery_scope.py snapshot --s0 . --repo D:\my-system-web --repo D:\my-system-api --output $discoverySnapshot
```

Repite `--repo` por cada raíz Git que se vaya a inspeccionar, no por cada módulo
del monorepo. Para sólo el S0 omite esos argumentos. No incluyas rutas sin acceso
ni amplíes el alcance para conseguir una comprobación verde. Si los repositorios
disponibles no bastan, registra la cobertura parcial en el borrador.

El snapshot conserva cambios previos como referencia; no exige un árbol limpio.
Comprueba el código de salida antes de continuar. No vuelvas a capturarlo mientras
el trabajo está en curso para ocultar diferencias.

## Después del análisis y antes de publicar

Sin `--allow-context` se exige que los repositorios sigan como estaban:

```powershell
py -3 skills/project-discovery/scripts/check_discovery_scope.py verify --snapshot $discoverySnapshot
```

Haz esta comprobación al entregar el borrador y de nuevo antes de escribir el
contexto aprobado. Un cambio concurrente puede explicar una diferencia, pero no
autoriza a ignorarla. Resuelve su procedencia y vuelve a acordar el borrador
afectado antes de empezar un nuevo recorrido con otra referencia.

## Después de escribir los archivos aprobados

Pasa exactamente las rutas aprobadas, relativas al S0 y con `/`. Por ejemplo,
si sólo se aprobaron el índice y la arquitectura:

```powershell
py -3 skills/project-discovery/scripts/check_discovery_scope.py verify --snapshot $discoverySnapshot --allow-context context/project.md --allow-context context/architecture.md
git diff --check
git diff -- context/
git status --short
```

Revisa también los archivos nuevos, pues `git diff` no muestra su contenido
hasta que están en el índice; Discovery no los añade al índice. La allowlist no
aprueba por sí sola el contenido: contrasta el diff con el borrador revisado.

| Salida | Interpretación |
| --- | --- |
| `0` | Sin cambios fuera del alcance indicado |
| `1` | Hay diferencias fuera del alcance; no declarar publicación correcta |
| `2` | Entrada, referencia o ruta inválida; no existe verificación válida |

## Qué cubre y qué queda fuera

- En el S0 se comparan archivos del árbol, incluidos ignorados, salvo metadata
  `.git`; se detectan altas, cambios y eliminaciones fuera de las rutas exactas.
- En código se comparan archivos seguidos por Git y nuevos no ignorados. Los
  archivos ignorados de código, como dependencias o caches, quedan fuera de la
  comprobación de contenido. Eso no autoriza modificarlos durante Discovery.
- Para inspeccionar el contenido de un submódulo de código, inclúyelo como raíz
  Git explícita en un recorrido separado si se solapa con otra raíz capturada.
  No se atraviesan automáticamente repositorios o enlaces anidados.
- Se comprueban HEAD, rama e índice Git por separado. El script no hace stage,
  commit, checkout ni reset. No audita cada archivo interno de `.git`.
- No se admiten rutas permitidas absolutas, escapes, aliases de Windows ni
  destinos de contexto mediante enlaces simbólicos, junctions o hardlinks.
  Un `.git` anidado en el S0 impide verificarlo; sólo su metadata Git raíz está
  excluida. Las huellas de archivos incluyen sus permisos de modo, además de bytes.
- Una referencia sólo acredita las raíces capturadas y los estados comparados.
  No detecta una escritura transitoria revertida entre ambas comprobaciones ni
  certifica el comportamiento de un agente o el contenido semántico del texto.

Si Python/Git no están disponibles, un ZIP no tiene metadata Git o una raíz no
se puede verificar, limita la entrega a lectura y borrador. Comunica el requisito
faltante para publicar con este contrato; no instales herramientas por inferencia.
