# Cuándo y cómo elegir data sources con Foundry IQ

### Descripción
Tu knowledge base vale lo que valen sus datos. El curso enseña **seis tipos principales de data source**, pero la lista oficial de *knowledge sources* de Azure AI Search ya llega a **doce** (varios en preview). Elegir bien depende de dos preguntas: **dónde vive tu información** y **cómo necesitas accederla** — indexada (ingerida y procesada de antemano) o remota (consultada en tiempo real en la plataforma origen).

> [!NOTE]
> Referencias: [Configure data sources for knowledge bases](https://learn.microsoft.com/es-mx/training/modules/introduction-foundry-iq/4-data-requirements) y [¿Qué es un knowledge source?](https://learn.microsoft.com/es-mx/azure/search/agentic-knowledge-source-overview)

### Concepto base: indexado vs remoto (la distinción que lo explica todo)

Un **knowledge source** es un objeto de nivel superior en tu servicio de AI Search que define contenido para el retrieval agéntico. Cada uno es **indexado** o **remoto**:

- **Indexado:** el contenido se ingiere ANTES de la consulta, por una de tres rutas: (1) *trae tu propio índice* (envuelve un índice existente), (2) *carga directa de archivos* (File, sin storage externo), o (3) *pipeline de indexador autogenerado* (data source + skillset + indexer + índice, creados automáticamente desde Blob, SQL, OneLake, SharePoint…). Las consultas corren localmente en AI Search: keyword, vectorial o híbrida.
- **Remoto:** el contenido **nunca entra a AI Search**; se recupera en tiempo de consulta vía las APIs nativas de la plataforma (Bing, SharePoint, Fabric, MCP, Work IQ). Siempre fresco, sin índice que mantener.
- **Clasificación unificada:** venga de donde venga, todo lo recuperado pasa por el mismo pipeline de ranking — se puntúa por relevancia, se fusiona entre consultas y se re-rankea antes de responder. Por eso puedes mezclar fuentes sin que el agente lo note.

### Los 6 data sources principales (los del curso)

| Data source | Tipo de acceso | Ideal para |
|---|---|---|
| **Azure AI Search Index** | Indexado | Búsqueda empresarial con pipelines personalizados ya existentes |
| **Azure Blob Storage** | Indexado (pipeline autogenerado) | Archivos/documentos en Azure Storage (PDF, DOCX, TXT, MD, HTML) |
| **Web** | Remoto (tiempo real) | Información pública y actual vía Bing |
| **SharePoint (Remote)** | Remoto (tiempo real) | Contenido vivo de SharePoint con gobernanza Microsoft 365 |
| **SharePoint (Indexed)** | Indexado | Búsqueda avanzada sobre SharePoint con pipelines custom |
| **OneLake** | Indexado (pipeline autogenerado) | Datos no estructurados en Microsoft Fabric |

**Detalles por fuente:**

1. **Azure AI Search Index:** úsalo si ya invertiste en AI Search. Ventajas: **ranking semántico** (relevancia contextual, no solo keywords), **scoring profiles** (priorizar por lógica de negocio), **facetas** (filtrar por categorías) y **multi-idioma**. Es la opción con capacidades de búsqueda más sofisticadas.
2. **Blob Storage:** el camino más directo de archivo→conocimiento; Foundry IQ genera el pipeline completo. Organiza contenedores por tema/nivel de acceso — esa organización ES tu gobernanza.
3. **Web:** grounding con Bing para eventos recientes, precios, disponibilidad. **Advertencia oficial:** menos control sobre las fuentes; si la exactitud verificable es crítica, prefiere fuentes indexadas propias. Combínala como fuente **suplementaria** cuando el conocimiento interno no alcance. Ojo: una KB con fuente web **requiere un LLM** configurado.
4. **SharePoint Remote vs Indexed** (favorito de examen):

   | Característica | Remote | Indexed |
   |---|---|---|
   | Acceso | Consulta en tiempo real | Índice preprocesado |
   | Velocidad | Depende de SharePoint | Más rápida |
   | Mantenimiento | Sin índice que mantener | Requiere actualizar índice |
   | Búsqueda avanzada | Limitada | Todo el poder de AI Search (analizadores custom, enriquecimiento con IA, combinar fuentes) |
   | Frescura | Siempre actual | Según programación de indexado |
   | Permisos | Respeta permisos de SharePoint automáticamente | Se configuran al indexar |

5. **OneLake:** si tu organización usa Fabric — reportes BI, documentación de datasets, hallazgos analíticos accesibles conversacionalmente.

### La lista completa oficial (12 knowledge sources)

Además de los 6 anteriores, la documentación de AI Search agrega estos (todos en versión preliminar):

| Knowledge source | Tipo | Para qué |
|---|---|---|
| **File** | Indexado | Subir archivos **directamente** a AI Search, sin Blob ni pipeline — la ruta más rápida para prototipos |
| **Azure SQL** | Indexado | Indexar una tabla o vista de Azure SQL |
| **Fabric data agent** | Remoto | Respuestas de un **agente de datos** de Microsoft Fabric (analítica conversacional) |
| **Fabric ontology** | Remoto | Respuestas basadas en **entidades y relaciones** de una ontología de Fabric (la conexión con Fabric IQ) |
| **Servidor MCP** | Remoto | Resultados en vivo de un **servidor MCP externo** como fuente de conocimiento |
| **Work IQ** | Remoto | **Inteligencia organizativa de Microsoft 365** (correos, reuniones, colaboración) como fuente — el puente entre Foundry IQ y Work IQ del tema 6 |

Lectura de examen: la familia "indexado" crece hacia datos estructurados (SQL) y carga directa (File); la familia "remoto" crece hacia **otros sistemas inteligentes** (Fabric, MCP, Work IQ) — Foundry IQ se vuelve el punto de unión de las tres capas IQ.

### Ciclo de vida, permisos y extras (reglas operativas)

- **Orden de creación:** el knowledge source se crea ANTES que la knowledge base (la KB lo referencia por nombre); ambos deben vivir en el **mismo servicio de búsqueda**; para borrar una fuente, primero actualiza/elimina las KBs que la referencian.
- **Permisos:** crear fuentes requiere **Search Service Contributor**; si la fuente genera pipeline de indexación, también **Search Index Data Contributor** (o admin API key).
- **Etiquetas de confidencialidad (preview):** en Blob, OneLake y SharePoint indexados puedes incorporar **etiquetas de Microsoft Purview** (`ingestionPermissionOptions: sensitivityLabel`); se sincronizan al índice y aplican acceso a nivel de documento en el momento de la consulta.
- **Imágenes embebidas (preview):** con un `assetStore` configurado, las imágenes de tus documentos (diagramas, gráficos) se conservan y se inyectan en la síntesis de respuesta para que el LLM razone sobre ellas.

### Controlar QUÉ fuente se consulta en cada pregunta

Cuando una KB tiene varias fuentes, el motor decide cuáles consultar:

- **`alwaysQuery: true`** en una fuente = se consulta siempre, sin importar el nivel de razonamiento.
- **Retrieval reasoning effort:** `minimal` (sin LLM — máxima velocidad y mínimo costo), `low` (el LLM planifica y **selecciona fuentes**), `medium` (agrega una pasada iterativa para resultados más profundos).
- **Qué usa el LLM para elegir fuente** (con low/medium): el **`name`** del knowledge source, la **`description`** del índice y las **`retrievalInstructions`** de la KB ("usa la fuente de hoteles para hospedaje; si no, la de políticas").

> [!TIP]
> Consecuencia práctica: el **nombre y la descripción de cada fuente son parte del prompt**. "docs-blob-01" no le dice nada al planificador; "documentacion-tecnica-productos" sí. Nómbralas como le explicarías a un colega qué contienen.

### Guía de decisión rápida

| Si tus datos están… | Y necesitas… | Elige |
|---|---|---|
| En SharePoint | Setup simple, siempre actual, permisos M365 | SharePoint Remote |
| En SharePoint | Búsqueda avanzada, pipelines custom | SharePoint Indexed |
| Archivos en Azure Storage | Pipeline automático desde contenedores | Blob Storage |
| Archivos sueltos (prototipo) | Subirlos ya, sin infraestructura | File (carga directa) |
| En una tabla/vista SQL | Indexar datos estructurados | Azure SQL |
| En Microsoft Fabric | Contenido del lakehouse | OneLake |
| En Fabric con semántica de negocio | Razonar sobre entidades/analítica | Fabric data agent / ontología |
| Ya indexados | Aprovechar inversión en AI Search | AI Search Index |
| Públicos y cambiantes | Contenido web en tiempo real | Web (+ LLM obligatorio) |
| En Microsoft 365 | Contexto organizacional (correo, reuniones) | Work IQ |
| En un sistema con servidor MCP | Resultados en vivo de sus tools | Servidor MCP |

> [!TIP]
> Puedes **combinar múltiples fuentes** en una sola knowledge base: el motor genera subconsultas por fuente, las ejecuta en paralelo y unifica el ranking. Patrón típico: SharePoint interno como fuente principal + Web como complemento para información actual.

### Beneficios
Elegir el data source correcto equilibra frescura, velocidad, capacidades de búsqueda y gobernanza — y como Foundry IQ unifica todo detrás de la knowledge base (con ranking unificado y selección inteligente de fuentes), puedes cambiar o sumar fuentes sin tocar a los agentes.
