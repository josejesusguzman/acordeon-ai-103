# Cómo hacer prompt engineering

### Descripción
El prompt engineering es la técnica más barata y rápida para mejorar la salida de un modelo: no tocas el modelo ni los datos, solo la forma de pedirle las cosas. En Foundry esto se materializa sobre todo en el **system prompt** (instrucciones del sistema) y en la estructura de los mensajes.

> [!NOTE]
> Módulo de referencia: [Optimización del rendimiento del modelo](https://learn.microsoft.com/es-es/training/modules/optimize-generative-ai-model-performance/) (unidad de prompt engineering)

### Técnicas esenciales

1. **Instrucciones claras y específicas:** indica tarea, formato de salida, tono, límites y a quién va dirigido. Vago: "resume esto". Bueno: "Resume en 3 viñetas, máx. 20 palabras cada una, para un director no técnico".
2. **System prompt con rol (persona):** define quién es el asistente y sus reglas:
   ```text
   Eres un asistente de soporte de la empresa Contoso.
   - Responde SOLO sobre productos Contoso.
   - Si no sabes la respuesta, dilo y ofrece escalar a un humano.
   - Responde siempre en español, en tono profesional y breve.
   ```
3. **Few-shot prompting (ejemplos en contexto):** incluye 1-5 pares entrada→salida deseada antes de la pregunta real. Es la forma más efectiva de fijar formato y estilo.
4. **Zero-shot vs few-shot:** empieza zero-shot (sin ejemplos); si el formato falla, agrega ejemplos antes de pensar en fine-tuning.
5. **Chain of thought:** pide razonamiento paso a paso ("piensa paso a paso antes de responder") para problemas lógicos/matemáticos. Con modelos de razonamiento (o-series) no es necesario: ya lo hacen internamente.
6. **Delimitadores y estructura:** separa instrucciones de datos con etiquetas o triple comilla para evitar que el modelo confunda contenido con órdenes (también mitiga prompt injection):
   ```text
   Clasifica el sentimiento del texto entre <texto></texto> como positivo/negativo/neutro.
   <texto>{{entrada_del_usuario}}</texto>
   ```
7. **Pedir salida estructurada:** exige JSON con esquema (structured outputs / `response_format`) cuando otro sistema consumirá la respuesta.
8. **Grounding en el prompt:** incluye la información de contexto relevante y ordena "responde únicamente con la información proporcionada" (base del patrón RAG).
9. **Ajusta parámetros junto con el prompt:** `temperature` baja (0-0.3) para tareas deterministas; `max_tokens` para controlar longitud y costo.

### Ejemplo completo en Python

```python
response = openai_client.chat.completions.create(
    model="gpt-4o",
    temperature=0.2,
    messages=[
        {"role": "system", "content": (
            "Eres un clasificador de tickets de TI. "
            "Devuelve SOLO un JSON: {\"categoria\": ..., \"prioridad\": ...}. "
            "Categorías: red, hardware, software, accesos."
        )},
        # few-shot
        {"role": "user", "content": "No me llega internet en el piso 3"},
        {"role": "assistant", "content": "{\"categoria\": \"red\", \"prioridad\": \"alta\"}"},
        # caso real
        {"role": "user", "content": "Necesito licencia de Visio"}
    ]
)
```

### Errores comunes
- Meter TODO en un mega-prompt en lugar de iterar y medir cambio por cambio.
- No fijar formato de salida y luego "parsear" texto libre con regex frágiles.
- Confiar en el prompt como única barrera de seguridad (el prompt se puede inyectar; usa también Guardrails/Content Safety).
- Olvidar que cada token del prompt cuesta: prompts kilométricos suben latencia y factura.

### Beneficios
Un buen system prompt bien iterado resuelve la mayoría de los problemas de calidad sin costo adicional, y es el prerequisito antes de justificar RAG o fine-tuning.
