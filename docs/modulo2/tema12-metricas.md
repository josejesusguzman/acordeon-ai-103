# Métricas de agentes y cómo usarlas

### Descripción
"Lo que no se mide, no se mejora" aplica doble a los agentes: su salida es no determinista y su costo variable. Necesitas tres familias de métricas: **calidad**, **seguridad** y **operación** — y usarlas en momentos distintos del ciclo de vida.

> [!NOTE]
> Módulos de referencia: [Evaluación de modelos](https://learn.microsoft.com/es-es/training/modules/model-catalog-evaluate/) y [Observe and troubleshoot apps](https://learn.microsoft.com/es-es/training/modules/observe-troubleshoot-apps/)

### 1. Métricas de calidad (¿responde bien?)

| Métrica | Qué mide | Cuándo usarla |
|---|---|---|
| **Groundedness** | Si la respuesta se apoya en el contexto/fuentes recuperadas (no alucina) | Soluciones RAG, siempre |
| **Relevance** | Si la respuesta atiende la pregunta | General |
| **Coherence** | Fluidez lógica del texto | General |
| **Fluency** | Calidad gramatical | General |
| **Similarity** | Parecido con una respuesta esperada (ground truth) | Cuando tienes dataset etiquetado |
| **F1 / BLEU / ROUGE / METEOR** | Superposición textual con la referencia | Traducción, resumen, extracción |
| **Retrieval score** | Calidad de los documentos recuperados | Diagnóstico de RAG (¿falla la búsqueda o el modelo?) |

**Específicas de agentes:**
- **Intent resolution:** ¿entendió la intención del usuario?
- **Task adherence:** ¿cumplió la tarea según sus instrucciones?
- **Tool call accuracy:** ¿llamó las tools correctas con los parámetros correctos?

### 2. Métricas de seguridad (¿responde sin daño?)
- Tasas de contenido dañino (odio, sexual, violencia, autolesión) en salidas.
- Éxito de **jailbreak/prompt injection** en pruebas adversarias.
- Fugas de material protegido o PII.
- Se obtienen con evaluaciones de seguridad y red teaming automatizado antes de cada release.

### 3. Métricas operativas (¿funciona como software?)
- **Latencia** (p50/p95, tiempo al primer token en streaming).
- **Tokens y costo** por conversación (prompt + completion).
- **Tasa de errores** (content_filter, rate limit 429, timeouts de tools).
- **Tasa de fallback/escalamiento a humano** y **frecuencia de citación** (en RAG).
- **Satisfacción** (thumbs up/down, CSAT).
- Se capturan con **tracing** (OpenTelemetry → Application Insights) por cada run del agente.

### Cómo usarlas en el ciclo de vida

```text
Desarrollo:  evaluación batch (calidad) por cada cambio de prompt/modelo → comparar versiones
Pre-release: evaluación de seguridad + red teaming → gate del pipeline CI/CD
Producción:  tracing + métricas operativas + evaluación continua sobre muestras de tráfico real
Mejora:      conversaciones mal calificadas → nuevos casos del dataset de evaluación
```

### Ejemplo: evaluación batch con azure-ai-evaluation

```python
from azure.ai.evaluation import evaluate, GroundednessEvaluator, RelevanceEvaluator

resultado = evaluate(
    data="preguntas_prueba.jsonl",           # {query, context, response, ground_truth}
    evaluators={
        "groundedness": GroundednessEvaluator(model_config),
        "relevance": RelevanceEvaluator(model_config),
    },
)
print(resultado["metrics"])   # promedios por métrica → compara contra tu umbral
```

### Errores comunes
- Medir solo calidad y descubrir el costo en la factura.
- Evaluar una vez y nunca más (cada cambio de prompt es un release).
- Usar promedios que esconden el 5% de respuestas desastrosas: revisa distribuciones y peores casos.

### Beneficios
Con las tres familias de métricas puedes responder las tres preguntas que importan: ¿responde bien?, ¿es seguro?, ¿es sostenible en costo y latencia? — y detectar cuál de las tres se rompió cuando algo falla.
