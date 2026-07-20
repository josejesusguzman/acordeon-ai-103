# ¿Cuándo usar prompt engineering, RAG o fine-tuning… y cómo combinarlos?

### Descripción
Las tres técnicas atacan problemas distintos. La pregunta clave no es "¿cuál es mejor?" sino "¿qué le falta al modelo?": ¿instrucciones?, ¿conocimiento?, ¿comportamiento? Este tema explica **cómo se hace cada técnica en Microsoft Foundry**, en qué se diferencian y cierra con un tutorial práctico en el portal.

> [!NOTE]
> Módulo de referencia: [Optimización del rendimiento de modelos de IA generativa con Microsoft Foundry](https://learn.microsoft.com/es-mx/training/modules/optimize-generative-ai-model-performance/) (unidades 2, 3 y 4)

### Aspectos Clave

1. **Prompt engineering — le faltan INSTRUCCIONES:** el modelo sabe hacerlo pero no como tú quieres (formato, tono, reglas). Costo casi cero, iteración en minutos. Límite: no agrega conocimiento nuevo ni cambia el comportamiento profundo.
2. **RAG — le falta CONOCIMIENTO:** el modelo no conoce tus datos (documentación interna, catálogo, políticas) o la información cambia constantemente. Límite: no cambia estilo ni habilidades; depende de la calidad de la búsqueda.
3. **Fine-tuning — le falta COMPORTAMIENTO:** quieres estilo, formato o especialización consistentes sin prompts kilométricos. Límite: NO es la forma correcta de inyectar conocimiento factual actualizable (para eso está RAG); el conocimiento queda "congelado".

### Cómo se hace cada técnica

#### 1. Prompt engineering (detalle completo en tema 6)

Diseñas y refinas los mensajes sin tocar el modelo: system prompt con el checklist oficial (rol → límites → formato → política "cuando no estés seguro"), patrones (persona, plantilla, few-shot, chain of thought, delimitadores) y parámetros (`temperature` o `top_p`, no ambos). En Foundry se itera en el **playground** (campo **Instructions**) y se mide con evaluaciones (tema 2).

#### 2. RAG: fundamentar el modelo con tus datos

**El problema:** sin fundamentos (*ungrounded*), el modelo solo tiene sus datos de entrenamiento — con fecha de corte y sin tu información privada. Pregúntale "¿qué hoteles ofrecen en París?" y te inventará nombres creíbles pero ficticios.

![Modelo sin fundamentar responde solo con datos de entrenamiento](https://learn.microsoft.com/es-mx/training/wwl-data-ai/optimize-generative-ai-model-performance/media/ungrounded.png)

**La solución:** fundamentar (*grounding*) = darle datos de una fuente confiable junto con la pregunta. Con el catálogo real como contexto, responde con hoteles, precios y disponibilidad reales.

![Comparación: modelo sin fundamentar vs fundamentado con datos](https://learn.microsoft.com/es-mx/training/wwl-data-ai/optimize-generative-ai-model-performance/media/grounded.png)

**El patrón RAG en 3 pasos:**

![Patrón RAG: recuperar, aumentar, generar](https://learn.microsoft.com/es-mx/training/wwl-data-ai/optimize-generative-ai-model-performance/media/rag-pattern.png)

1. **Recuperar (Retrieve):** busca en tu origen de datos la información relevante para la pregunta.
2. **Aumentar (Augment):** agrega lo recuperado al prompt como contexto.
3. **Generar (Generate):** el modelo responde fundamentado en ese contexto.

**Embeddings y búsqueda vectorial (el motor de "recuperar"):** un *embedding* es la representación matemática de un texto como vector. Frases con palabras distintas pero significado similar ("los niños jugaron en el parque" / "los chicos corrieron por el jardín de juegos") quedan **cerca** en el espacio vectorial; la **similitud coseno** (cercana a 1 = muy similares) permite encontrar documentos relevantes aunque no coincidan las palabras exactas.

![Textos representados como vectores en el espacio multidimensional](https://learn.microsoft.com/es-mx/training/wwl-data-ai/optimize-generative-ai-model-performance/media/vector-embeddings.jpg)

**Con Azure AI Search en Foundry:** 1) agrega tus datos (Blob Storage, Data Lake Gen2, OneLake o carga directa); 2) crea un **índice** usando un modelo de embeddings; 3) consulta el índice en cada pregunta. Soporta búsqueda por **palabra clave**, **semántica**, **vectorial** e **híbrida** (combina las tres — la recomendada para IA generativa).

![Índice de Azure AI Search consultado para recuperar datos](https://learn.microsoft.com/es-mx/training/wwl-data-ai/optimize-generative-ai-model-performance/media/index.png)

**En código (SDK + Responses API):**

```python
client = project.get_openai_client()
response = client.responses.create(
    model="gpt-4o",
    input=[
        {"role": "system", "content": "You are a helpful travel advisor. "
         "Use the following hotel data to answer: " + retrieved_context},
        {"role": "user", "content": "Which hotels do you offer in Paris?"},
    ],
)
```

> [!TIP]
> Para agentes que necesitan conocimiento sin administrar tu propia infraestructura de búsqueda, considera **Foundry IQ** (almacén de conocimiento administrado). Y en la Responses API, las herramientas **`file_search`** (sobre un vector store con tus PDFs) y **`web_search`** implementan RAG sin montar Azure AI Search (ver tutorial).

#### 3. Fine-tuning: entrenar el comportamiento

Tomas un modelo preentrenado y lo **entrenas adicionalmente** con un dataset pequeño y específico; sus pesos internos se ajustan para responder según tus patrones. Foundry usa **LoRA** (*Low-Rank Adaptation*): actualiza solo un subconjunto de parámetros — más rápido y barato que reentrenar todo, manteniendo la calidad.

**Los 3 tipos de fine-tuning en Foundry (favorito de examen):**

| Técnica | Cómo funciona | Cuándo usarla |
|---|---|---|
| **SFT** (Supervised Fine-Tuning) | Entrena con pares prompt→respuesta etiquetados | Tareas con forma clara y bien definida de resolverse |
| **RFT** (Reinforcement Fine-Tuning) | Feedback iterativo con un calificador que premia mejores respuestas | Tareas complejas/dinámicas con muchas soluciones posibles; mejora el razonamiento |
| **DPO** (Direct Preference Optimization) | Pares de respuesta preferida vs. no preferida | Alinear a preferencias humanas; más ligero computacionalmente que RL tradicional |

Se pueden combinar: primero SFT para crear el modelo custom, luego DPO para alinearlo.

**Cuándo se justifica:** tono/voz de marca consistente, formatos de salida confiables cuando few-shot no basta, **reducir prompts largos** (el patrón queda "dentro" del modelo → menos tokens y latencia), **destilación** (transferir capacidades de un modelo grande a uno pequeño y barato) y mejorar la **selección de herramientas** en apps con tool calling.

**Preparar los datos (JSONL):** cada línea es una conversación completa:

```json
{"messages": [{"role": "system", "content": "You are a friendly travel advisor for Margie's Travel."}, {"role": "user", "content": "What's a good beach destination in Europe?"}, {"role": "assistant", "content": "For a beautiful European beach experience, consider the Algarve in southern Portugal! ..."}]}
```

Reglas: **mismo system message en todos los ejemplos** (y úsalo también en la inferencia del modelo afinado — dejarlo vacío degrada la precisión); ejemplos representativos y de alta calidad; **al menos cientos** de ejemplos; las respuestas del asistente deben reflejar exactamente el estilo/formato deseado.

**Desafíos:** costo de entrenamiento + hosting por hora; datos pobres → overfitting o sesgo; re-entrenar cuando cambian los datos o sale un modelo base nuevo; experimentar con hiperparámetros (épocas, batch size, learning rate); el modelo especializado puede empeorar en tareas generales (*drift*).

> [!IMPORTANT]
> Fine-tuning es capacidad avanzada: **siempre establece primero la línea base** de un modelo estándar con evaluaciones. Sin baseline no puedes saber si mejoraste o empeoraste.

### Diferencias entre las tres técnicas

| | Prompt engineering | RAG | Fine-tuning |
|---|---|---|---|
| **Qué modifica** | El mensaje | El contexto (datos inyectados) | Los pesos del modelo |
| **Qué resuelve** | Instrucciones, formato, tono | Conocimiento propio y actualizado | Comportamiento consistente |
| **Necesita** | Iterar prompts | Índice/vector store + embeddings | Dataset JSONL (cientos de ejemplos) |
| **Costo** | ~Cero | Indexación + tokens de contexto | Entrenamiento + hosting por hora |
| **Iteración** | Minutos | Horas (indexar) | Horas/días (entrenar) |
| **Actualizar información** | N/A | Reindexar (al momento) | Reentrenar (lento y caro) |
| **Citar fuentes** | No | ✔ Sí | No |
| **Conocimiento** | No agrega | ✔ Agrega y actualizable | Queda congelado |

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

### Tutorial en Microsoft Foundry

Dos partes, basadas en los ejercicios oficiales [Create a generative AI app that uses tools](https://microsoftlearning.github.io/mslearn-ai-studio/Instructions/Exercises/04a-use-own-data.html) (~30 min) y [Fine-tune a language model](https://microsoftlearning.github.io/mslearn-ai-studio/Instructions/Exercises/04b-finetune-model.html) (~90 min). Requieren suscripción de Azure.

#### Parte A: Prompt + grounding (RAG con tools) en el playground

1. **Proyecto y modelo:** en https://ai.azure.com crea un proyecto (opción **New Foundry** activada) y desde **Discover → Models** implementa `gpt-5.2` con configuración predeterminada; se abrirá en el playground.

   ![Interfaz de implementación de modelo en Foundry](https://learn.microsoft.com/es-mx/training/wwl-data-ai/model-catalog-evaluate/media/deploy-model.png)

2. **Prompt engineering:** en el campo **Instructions** escribe:

   ```
   You are a travel assistant that provides information on travel services available from Margie's Travel.
   ```

   ![Playground de chat en Microsoft Foundry](https://learn.microsoft.com/es-mx/training/wwl-data-ai/model-catalog-evaluate/media/chat-playground.png)

3. **Comprueba el límite del prompt:** pregunta `What are some recommended tourist activities in New York next month?`. La respuesta será genérica: el modelo no tiene información actual — ninguna instrucción se la puede dar.
4. **Agrega grounding:** en la sección **Tools** (bajo las instrucciones), pulsa **Add** y agrega **web_search**. Repite la misma pregunta: ahora el modelo recupera información actual. Acabas de ver la diferencia prompt vs. RAG en vivo.
5. **RAG con tus documentos (en código):** con la Responses API, crea un **vector store**, sube tus PDFs y pásalo a la herramienta `file_search`:

   ```python
   vector_store = openai_client.vector_stores.create(name="travel-brochures")
   openai_client.vector_stores.file_batches.upload_and_poll(
       vector_store_id=vector_store.id,
       files=[open(f, "rb") for f in glob.glob("brochures/*.pdf")])

   response = openai_client.responses.create(
       model=model_deployment,
       instructions="Answer questions about Margie's Travel using the provided brochures. "
                    "Search the web for general destination info.",
       input=input_text,
       tools=[{"type": "file_search", "vector_store_ids": [vector_store.id]},
              {"type": "web_search"}])
   ```

   Pregunta "What hotels does Margie's Travel offer there?" y responderá desde **tus folletos** (file_search), mientras que "What's happening in San Francisco next month?" usará web_search.

#### Parte B: Fine-tuning en el portal

1. **Región correcta:** crea el proyecto en una región que soporte fine-tuning del modelo (p. ej., North Central US o Sweden Central para gpt-5) e implementa el modelo **base** `gpt-5` (será tu línea base).
2. **Dataset:** descarga el [JSONL de ejemplo](https://microsoftlearning.github.io/mslearn-ai-studio/data/travel-finetune-hotel.jsonl) del laboratorio (asistente de viajes con tono consistente). Ábrelo y observa: mismo system message en cada línea, respuestas con el estilo deseado.
3. **Crea el trabajo:** en el panel izquierdo selecciona **Fine-tune** → botón **Fine-tune** y configura:
   - **Base model:** gpt-5 | **Customization method:** Supervised (SFT) | **Training type:** Standard
   - **Training data:** Upload new dataset → tu archivo .jsonl
   - **Suffix:** `ft-travel` (identificará al modelo resultante)
   - **Automatically deploy model after job completion:** activado | **Deployment type:** Developer
   - Hiperparámetros: déjalos por defecto la primera vez.
4. **Submit y monitorea:** el entrenamiento tarda 60+ minutos; revisa el avance en las pestañas **Monitor** y **Logs** del trabajo.
5. **Mientras esperas, establece la línea base:** chatea con el `gpt-5` base usando estas instrucciones y anota tono y estilo de sus respuestas:

   ```
   You are an AI travel assistant that helps people plan their trips. Your objective is to
   offer support for travel-related inquiries, such as visa requirements, weather forecasts,
   local attractions, and cultural norms.
   You should not provide any hotel, flight, rental car or restaurant recommendations.
   Ask engaging questions to help someone plan their trip.
   ```

   Prueba: `Where in Rome should I stay?`, `What are some local delicacies I should try?`, `What's the best way to get around the city?`
6. **Prueba el modelo afinado:** al completarse, verifica en **Deployments** que el modelo `ft-travel` esté implementado (si el auto-deploy falló, impleméntalo desde el trabajo), ábrelo en el playground con **las mismas instrucciones** y repite las mismas preguntas. Compara: el afinado mantiene el tono entusiasta y las restricciones con más consistencia que el base.
7. **Cierra el ciclo:** corre la misma evaluación batch sobre base y afinado y compáralas estadísticamente (temas 2 y 5). **Limpieza:** borra el grupo de recursos en https://portal.azure.com para evitar costos (el hosting del modelo afinado cobra por hora).

### Beneficios
Aplicar la técnica correcta en el orden correcto minimiza costo y tiempo: el prompt es gratis, RAG es mantenible, y el fine-tuning —el más caro— se reserva para cuando de verdad aporta.
