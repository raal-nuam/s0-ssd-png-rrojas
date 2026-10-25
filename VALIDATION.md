# Verificar S0 Core

Comprobaciones reproducibles de la plantilla y Project Discovery. Se ejecutan en
copias aisladas, con datos sintéticos; los resultados de cada sistema se conservan
en su propio repositorio o herramienta de QA.

## Requisitos

- Git y OpenSpec **1.14.3** para comprobar adopción y workflows nativos.
- Python **>=3.10**, sólo biblioteca estándar, para el verificador de Discovery.
- Agentes compatibles con acceso a las fuentes para las pruebas de comportamiento.

## 1. Estructura y adopción del core

Desde la raíz de una copia de prueba:

```powershell
openspec --version
openspec schema which spec-driven --json
openspec status
openspec validate --all --strict --no-interactive
openspec doctor
git diff --check
```

Comprueba que el schema efectivo es el oficial del paquete, sin overrides locales
o personales. Una plantilla sin requisitos ni cambios puede devolver listas vacías.

1. Adopta una instancia con ID propio siguiendo [INSTALLATION.md](INSTALLATION.md).
   Comprueba identidad oficial, remoto propio y registro local del checkout.
2. Conecta uno o varios consumidores con `store:`. Desde cada raíz,
   `openspec context --json` debe resolver el S0; el puntero debe conservarse y no
   deben aparecer `openspec/specs/` ni `openspec/changes/` locales.
3. Comprueba fuentes propias en `.agents/skills/`, puentes Claude, nombres, enlaces
   y acceso desde un clon limpio. Al generar o actualizar integraciones oficiales,
   conserva configuración, contexto y métodos propios.
4. Verifica catálogo y ejecución por separado en cada cliente; registra versión,
   modelo, revisión y fuentes usadas. La presencia de archivos no acredita discovery.

## 2. Verificador de Project Discovery

Desde la raíz, sin generar caches Python en la plantilla:

```powershell
py -3 -B -m unittest discover -s .agents/skills/project-discovery/scripts -p test_project_discovery_scope.py -v
```

Estado técnico de referencia: **18 tests, 17 aprobados y 1 omitido** en Windows
por falta de privilegio para crear symlinks. Un caso omitido conserva ese estado.
La suite usa Git temporales: cubre cambios previos, allowlist exacta de Markdown
bajo `context/`, archivos protegidos/ignorados, código de lectura, rutas, permisos,
HEAD e índice. El [contrato de alcance](.agents/skills/project-discovery/SKILL.md#scope-checks)
explica su cobertura y límites.

## 3. Comportamiento de Discovery

Usa un S0 y dos o tres repositorios sintéticos: incluye monorepo, servicio
separado, documentación contradictoria y una dependencia desconocida.

1. Captura la referencia inicial fuera de los repositorios y pide a un agente
   independiente un borrador para un foco concreto, sin darle las conclusiones.
2. Comprueba fuentes y revisiones, hechos frente a inferencias, incertidumbres y
   conservación del contexto ajeno al foco. Verifica lectura sin allowlist.
3. Aprueba contenido y rutas concretos. Publica sólo ese diff de `context/` y
   verifica con la misma referencia y las rutas exactas aprobadas.
4. Configuración, identidad, specs, changes, workflows, código, HEAD e índice
   deben permanecer intactos. Comprueba la fuente canónica y el puente Claude.

## 4. Regresión funcional nativa en Codex y Claude

Prepara dos copias equivalentes del S0 y las mismas fuentes autorizadas. Fija
CLI, schema oficial, cliente, modelo y prompt; aísla registros y preferencias CLI.

1. Guarda hashes de configuración, identidad, schemas, plantillas y workflows.
2. Genera una propuesta con la skill oficial local `openspec-propose` y una
   necesidad funcional concreta. Conserva proposal, specs, design y tasks.
3. En la otra copia, ejecuta Discovery, revisa/publica sólo contexto aprobado y
   verifica que no alteró archivos nativos. No añadas reglas para forzar resultados.
4. En una sesión nueva repite el mismo `propose`; valida ambas copias con la CLI.
5. Revisa el significado: specs con comportamiento observable y escenarios;
   diseño con decisiones técnicas; tareas con implementación y comprobaciones.
   Un contrato técnico observable puede pertenecer a una spec.
6. Repite el par con cada cliente. Conserva fallos, omisiones y fuentes; no esperes
   identidad textual entre resultados ni sustituyas un cliente por otro.

**Pendiente:** completar esta regresión semántica dual con ejecución de modelos.
Hashes iguales, instrucciones CLI iguales y catálogos no demuestran su resultado.

El helper verifica cambios de archivos, no aprobación humana, exactitud del texto
ni escrituras transitorias revertidas. Las comprobaciones estructurales no ejecutan
el producto ni certifican permisos de conectores, onboarding o aceptación funcional.
