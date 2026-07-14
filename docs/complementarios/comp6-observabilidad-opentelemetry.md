# Tema Complementario 6: Observabilidad de aplicaciones LLM con OpenTelemetry

### Descripción
En una app LLM el "log" tradicional no basta: necesitas ver la **conversación completa como trace** — cada llamada al modelo, cada tool, tokens y costos por paso. El estándar para esto es **OpenTelemetry (OTel)** con sus convenciones semánticas para GenAI, y en Azure el destino natural es **Application Insights**.

### Aspectos Clave

1. **Los tres pilares aplicados a LLM:**
   - **Traces:** el flujo de una request: span del agente → spans de cada llamada LLM → spans de tools/retrieval. Es el pilar rey en GenAI.
   - **Metrics:** agregados: tokens/min, costo/hora, latencia p95, tasa de errores y de bloqueos de contenido.
   - **Logs:** eventos puntuales (excepciones, decisiones de negocio).

2. **Convenciones semánticas GenAI de OTel:** atributos estandarizados que las herramientas entienden: `gen_ai.system`, `gen_ai.request.model`, `gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens`, `gen_ai.tool.name`… Instrumentar con el estándar te da dashboards y comparabilidad gratis.

3. **Instrumentación en Azure (el camino corto):**
   ```python
   # pip install azure-monitor-opentelemetry opentelemetry-instrumentation-openai-v2
   from azure.monitor.opentelemetry import configure_azure_monitor
   from opentelemetry.instrumentation.openai_v2 import OpenAIInstrumentor
   import os

   os.environ["OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT"] = "true"  # ¡cuidado con PII!
   configure_azure_monitor(connection_string=APPINSIGHTS_CONN)   # exporta a App Insights
   OpenAIInstrumentor().instrument()                             # auto-instrumenta llamadas OpenAI

   # ... tu código normal: cada chat.completions.create genera spans automáticamente
   ```
   El SDK de Foundry (`azure-ai-projects`/agentes) también emite tracing que el portal de Foundry visualiza por *run*.

4. **Spans propios para tu lógica:**
   ```python
   from opentelemetry import trace
   tracer = trace.get_tracer("mi-agente")

   with tracer.start_as_current_span("retrieval") as span:
       docs = buscar(query)
       span.set_attribute("retrieval.k", len(docs))
       span.set_attribute("retrieval.top_score", docs[0].score)
   ```
   Regla: un span por decisión importante (retrieval, cada tool, validaciones), con atributos que luego quieras filtrar.

5. **Qué mirar en producción (dashboard mínimo):**
   - Latencia p50/p95 por paso (¿el lento es el LLM o el retrieval?).
   - Tokens y costo por conversación (tendencia diaria).
   - Errores por tipo: 429, content_filter, timeouts de tools.
   - Tasa de fallback/escalamiento y feedback negativo, enlazados al trace exacto.

6. **Privacidad:** capturar el contenido de los mensajes es oro para depurar y un riesgo de compliance: activa la captura solo en entornos controlados o redacta PII antes de exportar; App Insights permite gobernar retención y acceso.

### El flujo de depuración con traces
```text
Usuario reporta respuesta mala → buscas su trace por operation_id
→ ves: retrieval devolvió docs irrelevantes (score bajo)
→ causa raíz: query sin reescritura conversacional → arreglas → agregas caso al dataset de evaluación
```
Sin el trace, ese diagnóstico son horas de adivinanza; con él, minutos.

### Fuentes para profundizar
- [Semantic conventions para GenAI — OpenTelemetry](https://opentelemetry.io/docs/specs/semconv/gen-ai/)
- [Tracing de agentes en Foundry](https://learn.microsoft.com/es-es/azure/ai-foundry/how-to/develop/trace-application)
- [Azure Monitor OpenTelemetry](https://learn.microsoft.com/es-es/azure/azure-monitor/app/opentelemetry-enable)

### Beneficios
La observabilidad convierte tu app LLM en una caja de cristal: cada queja de usuario tiene un trace que la explica, cada peso de la factura tiene un span que lo justifica.
