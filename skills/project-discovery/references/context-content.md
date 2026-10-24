# Organizar contexto verificable

Elige el mínimo de documentos que permita encontrar y consultar el conocimiento.
Estas orientaciones sirven para el contexto; no son plantillas ni instrucciones
para proposal, specs, design o tasks de OpenSpec.

| Documento | Contenido útil |
| --- | --- |
| `context/project.md` | Propósito confirmado, usuarios conocidos, límites, mapa de componentes y enlaces a fuentes/documentos detallados; preguntas abiertas |
| `context/architecture.md` | Relaciones, flujos y contratos observados, tecnologías e infraestructura confirmadas; límites de lo inspeccionado |
| `context/components/<component-name>.md` | Responsabilidad, repositorio/subcarpeta, entradas/salidas y dependencias; archivos clave para investigar ese componente |

`project.md` funciona como índice legible. En un proyecto pequeño puede bastar;
en un sistema grande enlaza detalles y evita repetirlos. Usa un nombre de
componente estable en inglés; no confundas servicio, módulo y repositorio.
Un diagrama pequeño es útil si aclara relaciones sustentadas por evidencia.

Para los hallazgos, utiliza una tabla u otra forma breve de trazabilidad:

| Hallazgo | Estado | Fuente y revisión | Limitación |
| --- | --- | --- | --- |
| La API expone una operación de cancelación | Observado | `order-api`, `src/routes/orders.ts`, commit identificado, símbolo `cancelOrder` | No prueba comportamiento desplegado ni aceptación funcional |
| La cancelación podría emitir un evento | Inferencia | Referencias encontradas en el adaptador, revisión identificada | Confirmar el recorrido completo y el contrato |

Son ejemplos de redacción, no hechos del sistema que estás analizando. No copies
sus datos a un proyecto real. Identifica las fuentes con URL Git sin credenciales,
ruta relativa y revisión; si no conoces la revisión, dilo. Para una copia con
cambios locales, marca qué evidencia proviene del árbol de trabajo.

No infieras reglas de negocio de nombres de clases, README desactualizados o un
test aislado. No conviertas una tecnología observada en una obligación futura.
Los desacuerdos entre documentación, specs y código se conservan como preguntas
con las fuentes de ambos lados. Un producto nuevo puede tener sólo intención y
decisiones propuestas; eso no equivale a arquitectura implementada.

Al actualizar selectivamente, cambia únicamente los documentos afectados y los
enlaces del índice necesarios. Preserva información confirmada ajena al foco.
No elimines documentos porque un repositorio sea temporalmente inaccesible.
