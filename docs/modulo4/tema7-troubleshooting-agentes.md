# Troubleshooting de agentes: síntomas, causas y remedios

### Descripción
Depurar un agente es distinto a depurar código: el "bug" puede estar en el prompt, el modelo, las tools, el retrieval, los filtros o la infraestructura. Esta guía mapea **síntoma → causas probables → cómo diagnosticar → remedio**, apoyada en tracing.

> [!NOTE]
> Módulo de referencia: [Observe and troubleshoot your apps](https://learn.microsoft.com/es-es/training/modules/observe-troubleshoot-apps/)

### Herramienta #1: tracing
Antes de adivinar, observa. Habilita tracing (OpenTelemetry → Application Insights / Foundry portal): cada *run* muestra los pasos del agente: mensajes, tool calls con argumentos y resultados, tokens y latencia por paso. El 80% del troubleshooting es leer el trace.

### Tabla de síntomas

| Síntoma | Causas probables | Remedio |
|---|---|---|
| **Alucina datos de la empresa** | No busca en la knowledge base; instrucciones vagas | Instrucciones: "SIEMPRE busca antes de responder, NUNCA respondas de memoria" + verificar en el trace que la tool de retrieval se llama |
| **Responde "no tengo información" a preguntas que SÍ están en los docs** | Retrieval falla: mal chunking, falta híbrido/semántico, docs no indexados | Prueba la query directo contra el índice; revisa indexer/skillset; ajusta chunking y búsqueda híbrida |
| **No usa la tool correcta / no usa tools** | Descripción de la tool pobre o ambigua; demasiadas tools | Reescribe descripciones (cuándo usarla + formato params); reduce tools; ejemplos en instrucciones |
| **Llama tools con argumentos inválidos** | Esquema poco estricto; nombres confusos | Tipos/enums estrictos en el schema, validación dentro de la tool con mensajes de error útiles (el agente reintenta mejor) |
| **Se queda en loop llamando tools** | Tool devuelve errores crípticos; objetivo imposible | Errores accionables en la tool; límite de iteraciones (max steps); revisar instrucciones contradictorias |
| **Respuestas lentas** | Modelo grande innecesario; contexto gigante; tools lentas; sin streaming | Trace: identifica el paso lento; modelo menor para pasos simples; recorta contexto/historial; paraleliza tools; streaming en UX |
| **Error 429 (rate limit)** | Cuota TPM insuficiente; picos | Retry con backoff, aumentar cuota/PTU, balancear entre deployments |
| **Error content_filter** | Prompt o respuesta dispara guardrails | Revisa categorías/severidad del filtro; ajusta umbral si es falso positivo; maneja el error con mensaje amable |
| **Respuestas inconsistentes entre corridas** | Temperatura alta; instrucciones ambiguas | `temperature` baja; instrucciones explícitas; structured outputs |
| **Costos disparados** | Historial completo en cada llamada; modelo caro para todo | Trace de tokens por paso; truncar/resumir historial; modelo router o mezcla de modelos |
| **Auth falla en producción (local funcionaba)** | Managed identity sin roles RBAC | Asigna rol al recurso (p. ej. *Azure AI User*); `DefaultAzureCredential` en logs verbose |
| **El agente responde a temas fuera de alcance** | Sin límites en instrucciones | Define alcance y fallback ("si preguntan otra cosa, redirige a…") + evaluación de task adherence |

### Metodología (checklist de diagnóstico)

1. **Reproduce** con la conversación exacta (guarda thread/run IDs).
2. **Lee el trace completo:** ¿qué decidió el agente en cada paso? ¿qué devolvieron las tools?
3. **Aísla la capa:** ¿modelo (razonó mal)?, ¿prompt (instrucción ambigua)?, ¿tool (dato malo)?, ¿retrieval (contexto irrelevante)?, ¿infra (429/timeout)?
4. **Prueba la capa sola:** query directa al índice, tool llamada a mano, prompt en el playground.
5. **Corrige UNA cosa y re-evalúa** con tu dataset (evita arreglar un caso y romper diez).
6. **Agrega el caso al dataset de evaluación** para que no regrese.

### Errores de plataforma comunes (rápido)
- `401/403`: key inválida o falta rol RBAC. — `404`: nombre de deployment mal escrito (¡no es el nombre del modelo!). — `408/timeout`: tools lentas, sube timeout o hazlas async. — `InvalidRequestError` por contexto: excediste la ventana, recorta historial.

### Beneficios
Con tracing + esta tabla pasas de "el agente se porta raro" a hipótesis concretas y verificables — depuras agentes con el mismo rigor con el que depuras software.
