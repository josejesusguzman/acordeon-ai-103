# Explicación técnica de Azure AI Search

### Descripción
**Azure AI Search** es el motor de búsqueda administrado de Azure y la columna vertebral de casi todo RAG serio (Foundry IQ corre sobre él). Técnicamente combina un índice invertido (keyword), búsqueda vectorial (embeddings) y re-ranking semántico, más un pipeline de ingesta con enriquecimiento por IA.

> [!NOTE]
> Módulo de referencia: [AI knowledge mining con Azure AI Search](https://learn.microsoft.com/es-es/training/modules/ai-knowledge-mining/)

### Arquitectura de ingesta (cómo entran los datos)

```text
Data source (Blob, SQL, Cosmos...) 
   └─> Indexer (extrae y programa actualizaciones)
         └─> Skillset (enriquecimiento IA: OCR, entidades, embeddings, chunking)
               └─> Index (documentos buscables con campos tipados)
```

1. **Data source:** conexión a datos (Blob Storage, SQL, Cosmos DB…).
2. **Indexer:** el "crawler" que extrae contenido y refresca el índice según programación.
3. **Skillset:** cadena de **skills cognitivas** aplicadas a cada documento: OCR, detección de idioma, NER, key phrases, y skills críticas para RAG: **Text Split (chunking)** y **embedding** (vectorización con un modelo de Azure OpenAI). Se pueden crear **custom skills** (Azure Functions).
4. **Index:** colección de documentos JSON con campos tipados y atributos: `searchable`, `filterable`, `sortable`, `facetable`, `retrievable`, y campos vectoriales (`Collection(Edm.Single)` + perfil vectorial).
5. **Knowledge store (opcional):** proyecciones de los datos enriquecidos hacia Blob/tablas para analytics fuera de la búsqueda.

### Los modos de búsqueda (el corazón técnico)

| Modo | Cómo funciona | Fortaleza |
|---|---|---|
| **Full-text (BM25)** | Índice invertido + scoring por frecuencia de términos | Exactitud léxica (códigos, nombres) |
| **Vector (kNN/HNSW)** | Embeddings + vecinos más cercanos por similitud coseno | Significado semántico, sinónimos, multilingüe |
| **Híbrida** | BM25 + vectorial en paralelo, fusión con **RRF** (Reciprocal Rank Fusion) | Lo mejor de ambos: default recomendado para RAG |
| **Semantic ranker** | Re-ranking con modelos de lenguaje sobre el top-50 + *captions/answers* extractivos | Mejora precisión del top final |
| **Agentic retrieval** | Descompone la conversación en subconsultas paralelas y fusiona resultados | Preguntas complejas de agentes |

### Query features que debes conocer
- **Filtros OData:** `$filter=categoria eq 'legal' and año gt 2023` (pre-filtra antes de rankear).
- **Facetas:** agregaciones para navegación (contar por categoría).
- **Scoring profiles:** boosting por campo o frescura.
- **Analyzers:** tokenización por idioma (es.microsoft) que afecta cómo se indexa el texto.

### Ejemplo: búsqueda híbrida en Python

```python
from azure.search.documents import SearchClient
from azure.search.documents.models import VectorizedQuery

results = search_client.search(
    search_text="política de vacaciones",              # keyword (BM25)
    vector_queries=[VectorizedQuery(
        vector=embedding_de_la_pregunta,               # vector (semántico)
        k_nearest_neighbors=50,
        fields="content_vector")],
    query_type="semantic",                             # re-ranking semántico
    semantic_configuration_name="default",
    select="title,chunk,url",
    top=5
)
for r in results:
    print(r["title"], r["@search.reranker_score"])
```

### Decisiones de diseño que importan en RAG
1. **Chunking:** tamaño ~500-1000 tokens con solapamiento 10-20%; chunks demasiado grandes diluyen la relevancia, demasiado pequeños pierden contexto.
2. **Híbrido + semantic ranker** como default: keyword atrapa términos exactos, vector atrapa significado, el ranker pule el top.
3. **Metadatos filtrables** (fecha, departamento, permisos) para seguridad y precisión.
4. **Relevancia = 80% del éxito del RAG:** si el retrieval falla, ningún prompt lo arregla.

### Beneficios
Entender AI Search "por dentro" (ingesta→skills→índice; BM25 vs vector vs híbrido) te permite diagnosticar el 90% de los problemas de RAG y responder las preguntas técnicas de arquitectura del examen.
