# Validar Project Discovery sin alterar OpenSpec

Este procedimiento distingue tres pruebas: el verificador de archivos, el
comportamiento de la skill y la regresión de propuestas nativas. Instalar la
skill añade archivos de método, adaptadores y documentación; **ejecutarla** sólo
puede cambiar los documentos de contexto aprobados. No confundas ambos diffs.
Los resultados efectivamente obtenidos están en [VALIDATION.md](../VALIDATION.md).

## 1. Prueba automatizada del verificador

Desde la raíz del core, con Git y Python >=3.10:

```powershell
py -3 -m unittest discover -s tests -p test_project_discovery_scope.py -v
```

La suite trabaja en Git temporales. Comprueba lectura sin cambios incluso con
trabajo previo, allowlist exacta de contexto, archivos nuevos/ignorados del S0,
modificación y eliminación de archivos protegidos, código de sólo lectura,
estado Git y rutas que intentan salir del alcance. No se ejecuta producto ni
se utilizan repositorios reales.

El [contrato de comprobaciones](../skills/project-discovery/references/scope-checks.md)
explica la cobertura y sus límites. Un resultado verde no acredita aprobación
humana, exactitud del contexto ni aislamiento frente a escrituras transitorias.

## 2. Prueba de comportamiento de la skill

Usa copias aisladas de un S0 y dos o tres repositorios sintéticos, sin datos
personales. Incluye un monorepo con varios componentes, un servicio separado,
documentación que discrepe del código y alguna dependencia no accesible.

Pide a un agente independiente que use la skill y prepare contexto para un foco
concreto. No le des las conclusiones esperadas. Comprueba el resultado observable:

- Lee sólo fuentes autorizadas, identifica repositorios y componentes, y cita
  archivos y revisiones realmente inspeccionados.
- Separa hechos, inferencias y desconocidos; conserva las discrepancias entre
  fuentes y no convierte código o tests en reglas de negocio aprobadas.
- Presenta el borrador antes de escribir. La comprobación sin allowlist sigue
  pasando después del análisis.
- Después de aprobar contenido y rutas concretas, publica únicamente ese diff
  de contexto y verifica contra la misma referencia inicial.
- Una actualización selectiva conserva documentos ajenos al foco y trabajo
  previo. Sin herramientas de comprobación o con una fuente inaccesible, declara
  el límite y no inventa una publicación verificada.

Esta prueba con un agente evaluador no equivale a una sesión real de cada cliente.
Valida por separado la entrada Codex y el puente Claude, incluida la resolución
de recursos relativos a la fuente común.

## 3. Regresión funcional con OpenSpec nativo en ambos clientes

Ejecuta este recorrido en entornos de prueba, nunca sobre la plantilla maestra
ni un S0 de producción. Usa **OpenSpec 1.14.3**, las mismas fuentes de código y
sesiones nuevas por caso. Registra cliente, versión, modelo, commit y prompt.
Dentro de cada cliente mantén el mismo modelo y los mismos datos antes/después;
no se espera identidad textual entre agentes o ejecuciones.

1. Prepara dos copias equivalentes del S0: referencia y posterior. Aísla los
   registros y preferencias de CLI con directorios XDG de prueba. Comprueba
   `openspec --version` y `openspec schema which spec-driven --json`: usar el
   mismo nombre de schema no basta si hay una copia local o de usuario que lo
   sobrescribe. Debe resolver el schema oficial del paquete.
2. Guarda hashes y lista de rutas de `openspec/`, `.openspec-store/` y workflows
   oficiales, incluidos config, reglas y plantillas resueltas. La copia posterior
   puede tener Discovery instalado; registra esa diferencia de instalación.
3. En la referencia, invoca **la skill oficial local** `openspec-propose` con una
   necesidad funcional concreta. Por ejemplo: rechazar la publicación de una
   ruta sin título e informar el motivo. Conserva proposal, specs, design y tasks.
   No basta con `openspec new change`: ese comando sólo crea el scaffold.
4. En la copia posterior, ejecuta Project Discovery sobre las mismas fuentes,
   revisa su borrador y aprueba sólo los documentos de `context/` necesarios.
   Aplica snapshot/verificación según la skill. Comprueba que no cambió config,
   schemas, identidad, specs, changes ni workflows oficiales durante Discovery.
5. En una sesión nueva, repite el mismo `propose` con el mismo prompt, cliente y
   modelo. El agente puede consultar el contexto; no agregues nuevas reglas a
   `openspec/config.yaml` para forzar el resultado.
6. En ambas copias, ejecuta `openspec validate --all --strict --no-interactive`
   y revisa semánticamente los artefactos con la tabla siguiente. Compara también
   las instrucciones/plantillas de la CLI; su igualdad no demuestra por sí sola
   la calidad de los textos generados por un modelo.
7. Repite el par completo con **Codex y Claude Code**. Si un cliente no está
   disponible, registra esa fila como pendiente; no sustituyas su resultado por
   el de otro agente, un catálogo de skills o una validación estructural.

| Aspecto | Evidencia esperada |
| --- | --- |
| Specs | Comportamiento observable, requisitos y escenarios verificables |
| Diseño | Decisiones de arquitectura e implementación pertinentes |
| Tareas | Acciones de implementación/verificación con repositorio identificado |
| Contexto usado | Hechos con fuentes, sin convertir inferencias en requisitos |
| Integridad nativa | Config, schema efectivo, plantillas y workflows sin alteración por Discovery |
| Caso técnico observable | Un contrato entre sistemas puede estar en una spec; no se rechaza por ser técnico |
| Fallo semántico | Clases, frameworks o mecanismos internos impuestos como requisito sin necesidad observable que lo justifique |

No añadas un filtro automático basado en palabras como Java o Kafka: un contrato
observable puede mencionarlas legítimamente. La validación CLI comprueba la
estructura; la separación funcional/técnica requiere revisión del significado.

La separación coincide con la configuración nativa documentada por OpenSpec:
`context`, `rules` y `operations` pueden influir en las instrucciones. Discovery
no añade ni modifica esos campos. Véase
[configuración de OpenSpec](https://openspec.dev/docs/configuration/config-yaml)
y [schemas nativos](https://openspec.dev/docs/schemas).
