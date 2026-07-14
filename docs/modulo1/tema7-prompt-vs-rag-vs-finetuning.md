# ¿Cuándo usar prompt engineering, RAG o fine-tuning… y cómo combinarlos?

### Descripción
Las tres técnicas atacan problemas distintos. La pregunta clave no es "¿cuál es mejor?" sino "¿qué le falta al modelo?": ¿instrucciones?, ¿conocimiento?, ¿comportamiento?

> [!NOTE]
> Módulo de referencia: [Optimización del rendimiento de modelos de IA generativa](https://learn.microsoft.com/es-es/training/modules/optimize-generative-ai-model-performance/)

### Aspectos Clave

1. **Prompt engineering — le faltan INSTRUCCIONES:**
   - El modelo sabe hacerlo pero no como tú quieres (formato, tono, reglas).
   - Costo: casi cero. Velocidad de iteración: minutos.
   - Límite: no agrega conocimiento nuevo ni cambia el comportamiento profundo.

2. **RAG (Retrieval Augmented Generation) — le falta CONOCIMIENTO:**
   - El modelo no conoce tus datos (documentación interna, catálogo, políticas) o la información cambia constantemente.
   - Recupera documentos relevantes (Azure AI Search / Foundry IQ) y los inyecta en el prompt; permite **citar fuentes** y actualizar conocimiento sin re-entrenar.
   - Costo: infraestructura de indexación + tokens extra por contexto.
   - Límite: no cambia estilo ni habilidades del modelo; depende de la calidad de la búsqueda.

3. **Fine-tuning — le falta COMPORTAMIENTO:**
   - Quieres cambiar el estilo, formato o especialización de manera consistente: jerga médica/legal, salidas muy estructuradas, seguir instrucciones complejas sin prompts largos.
   - Reentrenas un modelo base con cientos/miles de ejemplos (JSONL con pares prompt→respuesta) en Foundry.
   - Costo: preparación de datos + entrenamiento + hosting. Iteración: horas/días.
   - Límite: NO es la forma correcta de inyectar conocimiento factual actualizable (para eso está RAG); el conocimiento queda "congelado".

### Tabla de decisión

| Síntoma | Solución |
|---|---|
| Respuestas correctas pero mal formateadas/tono equivocado | Prompt engineering |
| "No tengo información sobre eso" / alucina datos de TU empresa | RAG |
| Datos que cambian a diario | RAG (nunca fine-tuning) |
| Necesita citar fuentes verificables | RAG |
| Prompt gigante repetido en cada llamada para lograr el estilo | Fine-tuning (acorta el prompt) |
| Dominio con vocabulario muy específico y estable | Fine-tuning |
| Tareas nuevas que el modelo simplemente no hace bien | Fine-tuning (o cambiar de modelo) |

### ¿Cómo combinarlos? (orden recomendado)

1. **Siempre empieza** con prompt engineering + few-shot. Mide con evaluaciones.
2. **Agrega RAG** cuando el problema sea de conocimiento. El 80% de los casos empresariales se resuelven aquí (prompt + RAG).
3. **Agrega fine-tuning** solo si, con buen prompt y buen RAG, el estilo/formato sigue fallando o los prompts son tan largos que disparan costo/latencia.
4. **Combinación completa (RAFT: Retrieval-Augmented Fine-Tuning):** modelo afinado para tu dominio + RAG para hechos actualizados + prompt para reglas de negocio. Es el patrón de máxima calidad.

```text
Pregunta del usuario
   └─> RAG: busca contexto en tus datos
        └─> Prompt: instrucciones + contexto + pregunta
             └─> Modelo (base o fine-tuned) → respuesta con citas
```

### Beneficios
Aplicar la técnica correcta en el orden correcto minimiza costo y tiempo: el prompt es gratis, RAG es mantenible, y el fine-tuning —el más caro— se reserva para cuando de verdad aporta.
