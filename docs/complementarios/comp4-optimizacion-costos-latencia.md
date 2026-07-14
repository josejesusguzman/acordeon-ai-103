# Tema Complementario 4: Optimización de costos y latencia en aplicaciones LLM

### Descripción
La factura y la lentitud son las dos razones más comunes por las que los proyectos de IA generativa mueren después del piloto. Optimizar ambas es un rol central del AI Engineer, y casi todas las palancas están en la arquitectura, no en el modelo.

### Aspectos Clave

1. **Entiende el costo:** pagas por **tokens de entrada + tokens de salida** (los de salida suelen costar 3-4×). Costo por conversación = (system prompt + historial + contexto RAG + pregunta) + respuesta. El historial que crece sin control es el asesino silencioso de la factura.

2. **Palancas de costo (en orden de impacto):**
   - **Modelo correcto por tarea:** GPT-4o-mini/Phi para clasificar y resumir; el modelo frontier solo donde la evaluación demuestre que hace falta. Diferencia típica: 10-30× por token.
   - **Recortar contexto:** resumir/truncar historial (ventana deslizante + resumen), top-k más pequeño en RAG, system prompts concisos.
   - **Prompt caching:** los proveedores descuentan tokens de prefijos repetidos (system prompt largo, few-shots). Estructura tus prompts con la parte estática AL INICIO para maximizar el hit.
   - **Semantic caching:** cachea respuestas por similitud de embeddings (Redis); preguntas frecuentes ni siquiera llegan al modelo.
   - **Batch API:** para cargas offline (clasificar 1M de registros), la API batch cuesta ~50% menos a cambio de latencia de horas.
   - **max_tokens y formatos concisos:** limita la salida; JSON compacto en vez de prosa florida.

3. **Palancas de latencia:**
   - **Streaming:** no reduce el tiempo total pero SÍ el percibido (primer token en <1 s). Imprescindible en chat.
   - **Paralelizar:** llamadas independientes (multi-query, tools) con `asyncio.gather`.
   - **Menos round-trips de agente:** cada iteración tool→modelo suma segundos; diseña tools que devuelvan todo lo necesario de una vez.
   - **Modelos pequeños y regiones cercanas**; para carga sostenida, **PTU (provisioned throughput)** da latencia estable vs el "ruido" del pago por uso.

4. **PTU vs pago por uso (Standard):**

   | | Standard (tokens) | PTU (aprovisionado) |
   |---|---|---|
   | Costo | Variable, por uso | Fijo mensual |
   | Latencia | Variable | Estable y predecible |
   | Cuándo | Cargas irregulares, desarrollo | Producción con volumen sostenido |

5. **Ejemplo — semantic cache casero:**
   ```python
   # Si una pregunta muy similar ya fue respondida, devuelve el cache
   emb = embed(pregunta)
   hit = vector_store.search(emb, top_k=1)
   if hit and hit.score > 0.95:
       return hit.respuesta_cacheada          # $0, ~10 ms
   respuesta = llm(pregunta)                  # camino normal
   vector_store.add(emb, respuesta)
   ```

6. **Mide antes de optimizar:** con tracing identifica el paso caro/lento (¿el retrieval?, ¿un tool?, ¿tokens de historial?). Optimizar a ciegas produce complejidad sin ahorro. KPI útiles: costo por conversación, tokens p95, latencia p95, cache hit rate.

### Fuentes para profundizar
- [Prompt caching en Azure OpenAI](https://learn.microsoft.com/es-es/azure/ai-foundry/openai/how-to/prompt-caching)
- [Provisioned throughput (PTU)](https://learn.microsoft.com/es-es/azure/ai-foundry/openai/concepts/provisioned-throughput)
- [Batch API](https://learn.microsoft.com/es-es/azure/ai-foundry/openai/how-to/batch)

### Beneficios
Con estas palancas es normal reducir 60-90% el costo por conversación y bajar la latencia percibida a sub-segundo — la diferencia entre un piloto cancelado y un producto rentable.
