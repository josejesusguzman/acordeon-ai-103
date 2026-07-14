# Cuándo usar un modelo LLM y cuándo solo Azure AI Language

### Descripción
Un LLM "puede" hacer casi todo lo que hace Azure AI Language… a 10-100× el costo, con más latencia y sin garantías deterministas. La habilidad profesional (y de examen) es saber cuándo basta el servicio especializado y cuándo se justifica el LLM.

### Aspectos Clave

1. **Azure AI Language gana cuando:**
   - La tarea es **estándar y cerrada**: idioma, sentimiento, NER, PII, key phrases, resumen.
   - Procesas **volumen masivo** (miles/millones de documentos): el costo por transacción es una fracción del costo por tokens.
   - Necesitas **determinismo y auditabilidad**: mismas entradas → mismas salidas, con scores de confianza. Clave en compliance (PII).
   - Requieres **baja latencia** estable.
   - Los datos son sensibles y quieres minimizar superficie: un clasificador especializado no "conversa" ni puede ser jailbreakeado.

2. **El LLM gana cuando:**
   - La tarea requiere **comprensión profunda o razonamiento**: responder preguntas abiertas, seguir instrucciones complejas, sintetizar múltiples documentos.
   - Necesitas **generación**: redactar, reescribir, traducir con estilo, resumir con enfoque específico.
   - Las **categorías cambian seguido** o no tienes datos etiquetados: con un LLM defines categorías en el prompt (zero-shot), sin entrenar.
   - Es **conversacional**: chatbots con contexto, agentes con tools.
   - Entrada **multimodal** (texto + imagen).

3. **Tabla de decisión:**

| Tarea | Recomendación |
|---|---|
| Enmascarar PII en logs (millones/día) | Azure AI Language (PII) |
| Sentimiento de reseñas a escala | Azure AI Language |
| Extraer personas/fechas/montos estándar | Azure AI Language (NER) |
| Clasificar en categorías propias CON dataset etiquetado | Custom text classification (Language) |
| Clasificar en categorías que cambian cada mes SIN dataset | LLM (zero/few-shot) |
| Responder preguntas sobre documentos | LLM + RAG |
| Redactar respuesta personalizada al cliente | LLM |
| Bot con intents fijos y acciones deterministas | CLU (Language) |
| Asistente conversacional abierto | LLM / agente |

4. **El patrón híbrido (lo mejor de ambos):**
   ```text
   Texto ──> Language: PII redaction (proteger datos)
              └─> Language: idioma + entidades (metadatos baratos)
                   └─> LLM: solo los casos que requieren razonamiento/generación
   ```
   - Ejemplo real: contact center que usa Language para sentimiento y PII en el 100% de llamadas, y el LLM solo para resumir/redactar en el 10% que lo necesita.
   - Bonus: Azure AI Language ahora expone sus capacidades como **servidor MCP**, de modo que un AGENTE puede llamar a Language como tool — el LLM orquesta, el servicio especializado ejecuta ([módulo](https://learn.microsoft.com/es-es/training/modules/develop-text-analysis-agent-language-mcp/)).

5. **Regla del costo:** si puedes escribir la tarea como "clasificar/extraer con categorías conocidas", el servicio especializado casi siempre es más barato, rápido y estable. Si la tarea empieza con "entiende y decide/redacta…", es LLM.

### Beneficios
Usar Language para lo repetitivo y el LLM para lo inteligente reduce costos en un orden de magnitud, mejora latencia y deja el comportamiento crítico (privacidad) en componentes deterministas.
