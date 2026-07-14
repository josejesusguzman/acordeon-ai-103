# Tema Complementario 2: Arquitecturas RAG avanzadas

### Descripción
El RAG "ingenuo" (embed → buscar top-k → pegar en el prompt) se queda corto con preguntas complejas, corpus grandes o conversaciones largas. Estas técnicas son el siguiente nivel que un ingeniero intermedio debe manejar.

### Aspectos Clave

1. **Query rewriting / expansión:** la pregunta cruda del usuario suele ser mala query. Antes de buscar:
   - Reescribe con un LLM incorporando el contexto de la conversación ("¿y el precio?" → "¿cuál es el precio del plan Enterprise de X?").
   - **Multi-query:** genera 3-5 variantes de la pregunta, busca con todas y fusiona resultados (RRF).
   - **HyDE:** genera una respuesta hipotética y busca con SU embedding (documentos se parecen más a respuestas que a preguntas).

2. **Re-ranking en dos etapas:** recupera 50 candidatos rápido (híbrido) y re-rankea el top con un modelo más caro (semantic ranker de AI Search o un cross-encoder). Barato en la primera etapa, preciso en la segunda.

3. **Chunking inteligente:**
   - Respetar estructura (títulos, secciones, tablas) en lugar de cortar por caracteres.
   - **Parent-child:** buscas con chunks pequeños (precisión) pero entregas al LLM el chunk padre (contexto).
   - Enriquecer chunks con metadatos/resúmenes ("contextual retrieval": anteponer 1-2 frases de contexto del documento a cada chunk).

4. **GraphRAG:** además de vectores, construye un **grafo de conocimiento** (entidades y relaciones extraídas con LLM). Responde preguntas "globales" que el RAG clásico no puede ("¿cuáles son los temas dominantes en todo el corpus?"). Proyecto open source de Microsoft Research: [microsoft/graphrag](https://github.com/microsoft/graphrag).

5. **Agentic RAG:** el agente decide *si* buscar, *dónde* (múltiples knowledge bases/tools), *cuántas veces* (búsquedas iterativas hasta tener suficiente evidencia) y auto-critica su respuesta. Es el enfoque de Foundry IQ (agentic retrieval) y de patrones como Self-RAG/CRAG.

6. **Routing y filtros:** clasifica la pregunta primero (¿RRHH, legal, producto?) y busca solo en la fuente correcta con filtros de metadatos (fecha, departamento, permisos del usuario). Menos ruido = mejores respuestas y seguridad.

7. **Evaluación específica de RAG (la tríada):**
   - **Context relevance:** ¿lo recuperado es pertinente?
   - **Groundedness:** ¿la respuesta se apoya en lo recuperado?
   - **Answer relevance:** ¿la respuesta atiende la pregunta?
   Diagnóstico: si falla (1) es problema de búsqueda/chunking; si falla (2) es prompt/modelo; si falla (3) es entendimiento de la pregunta.

### Pipeline avanzado de referencia

```text
Pregunta ─> reescritura (LLM) ─> multi-query
             └─> búsqueda híbrida + filtros (50 candidatos)
                  └─> re-ranker (top 5-8)
                       └─> ensamblado de contexto (parent chunks + citas)
                            └─> LLM genera con instrucciones de grounding
                                 └─> verificación de groundedness → responder o re-buscar
```

### Fuentes para profundizar
- [Agentic retrieval en Azure AI Search](https://learn.microsoft.com/es-es/azure/search/search-agentic-retrieval-concept)
- [GraphRAG — Microsoft Research](https://microsoft.github.io/graphrag/)
- Paper: "Retrieval-Augmented Generation for Large Language Models: A Survey" (Gao et al., 2023)

### Beneficios
Estas técnicas convierten un RAG demo (70% de respuestas útiles) en un RAG de producción (90%+), y te dan el vocabulario para decidir qué palanca mover cuando la calidad se estanca.
