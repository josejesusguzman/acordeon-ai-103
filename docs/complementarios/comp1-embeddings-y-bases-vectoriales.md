# Tema Complementario 1: Embeddings y bases de datos vectoriales a fondo

### Descripción
Los embeddings son la tecnología que hace posible RAG, la búsqueda semántica y la memoria de agentes. Entenderlos "por dentro" —y conocer las bases vectoriales más allá de Azure AI Search— te permite diseñar y depurar cualquier sistema de recuperación.

### Aspectos Clave

1. **Qué es un embedding:** un vector de números (p. ej. 1536 o 3072 dimensiones con `text-embedding-3`) que representa el *significado* de un texto. Textos similares producen vectores cercanos; la cercanía se mide con **similitud coseno** (o producto punto / distancia euclidiana).

2. **Propiedades que debes conocer:**
   - Son específicos del modelo: NUNCA compares vectores de modelos distintos (ni siquiera versiones distintas). Si cambias de modelo de embeddings, re-indexa todo.
   - `text-embedding-3-large` permite **recortar dimensiones** (Matryoshka): menos memoria a cambio de algo de precisión.
   - El embedding captura el significado global; textos largos diluyen detalles → por eso el **chunking** importa tanto.

3. **Índices vectoriales — cómo escalan:** comparar contra millones de vectores uno a uno (kNN exacto) es lento; se usan índices aproximados (ANN):
   - **HNSW (Hierarchical Navigable Small World):** grafo multinivel; el estándar (lo usa Azure AI Search). Rápido y preciso, consume RAM.
   - **IVF/PQ:** particionado + compresión; menos RAM, algo menos de recall.
   - Trade-off universal: **velocidad vs recall vs memoria**.

4. **Opciones de base vectorial en el ecosistema Azure:**

   | Opción | Cuándo usarla |
   |---|---|
   | Azure AI Search | Default empresarial: híbrido + semantic ranker + skillsets |
   | Cosmos DB (vector search) | Ya usas Cosmos como BD operacional; vectores junto a los datos |
   | PostgreSQL + pgvector (Azure Database) | Equipo SQL-céntrico, costos bajos, filtros relacionales ricos |
   | Redis (Azure Managed Redis) | Latencia mínima, semantic caching |
   | Especializadas (Qdrant, Milvus, Pinecone) | Multi-cloud o features muy específicas |

5. **Ejemplo en Python — similitud coseno desde cero:**
   ```python
   import numpy as np

   def cos_sim(a, b):
       return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

   e1 = openai_client.embeddings.create(model="text-embedding-3-small",
                                        input="¿Cómo reinicio mi contraseña?").data[0].embedding
   e2 = openai_client.embeddings.create(model="text-embedding-3-small",
                                        input="Olvidé mi password, ¿qué hago?").data[0].embedding
   print(cos_sim(np.array(e1), np.array(e2)))   # ~0.8+ → mismo significado
   ```

6. **Errores comunes:**
   - Indexar documentos completos sin chunking (recall pésimo).
   - Olvidar normalizar/limpiar texto (boilerplate HTML ensucia el vector).
   - Confiar solo en vectores: los términos exactos (SKUs, nombres) se buscan mejor con keyword → usa **búsqueda híbrida**.
   - No versionar el modelo de embeddings usado por índice.

### Fuentes para profundizar
- [Embeddings — documentación de Azure OpenAI](https://learn.microsoft.com/es-es/azure/ai-foundry/openai/concepts/understand-embeddings)
- [Vector search en Azure AI Search](https://learn.microsoft.com/es-es/azure/search/vector-search-overview)
- Paper HNSW: Malkov & Yashunin, "Efficient and robust approximate nearest neighbor search" (2016)

### Beneficios
Dominar embeddings te convierte en el ingeniero que sabe POR QUÉ el RAG no encuentra un documento (¿chunking?, ¿modelo?, ¿índice?, ¿híbrido?) en lugar de solo reintentar prompts.
