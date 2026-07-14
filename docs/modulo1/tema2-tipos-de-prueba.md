# Tipos de prueba en soluciones de IA y cuándo usar cada una

### Descripción
Probar una aplicación de IA generativa no es como probar software tradicional: la salida **no es determinista** (el mismo prompt puede producir respuestas distintas) y muchas veces no existe una única respuesta "correcta". Por eso el ciclo de vida de IA generativa combina pruebas clásicas de software con **evaluaciones (evaluations)** específicas de modelos: manuales, asistidas por IA, métricas matemáticas de NLP y flujos de evaluación completos. Este tema desarrolla cada tipo de prueba, sus métricas y el momento correcto para usarlas.

> [!NOTE]
> Módulos de referencia: [Evaluación del rendimiento del modelo (Microsoft Foundry)](https://learn.microsoft.com/es-mx/training/modules/model-catalog-evaluate/5-evaluate-performance) y [Optimización del rendimiento del modelo](https://learn.microsoft.com/es-es/training/modules/optimize-generative-ai-model-performance/)

### ¿Por qué evaluar modelos?

La evaluación cumple cuatro propósitos críticos:

1. **Control de calidad:** detecta problemas (respuestas incorrectas o irrelevantes) durante la evaluación y no en producción, protegiendo a los usuarios y la reputación de la organización.
2. **Satisfacción del usuario:** entiende cómo experimentan la aplicación los usuarios y dónde las mejoras tienen mayor impacto.
3. **Mejora continua:** cada vez que actualizas prompts, agregas funcionalidades o cambias de modelo, la evaluación regular garantiza que la calidad no retroceda (pruebas de regresión).
4. **Cumplimiento y seguridad:** confirma que el modelo respeta las directivas, evita contenido dañino y protege datos y privacidad.

> [!TIP]
> Establece **benchmarks de evaluación al inicio del desarrollo** y vuelve a ejecutarlos después de cada modificación para medir el impacto objetivamente.

### 1. Evaluación manual (revisores humanos)

Consume tiempo, pero captura aspectos subjetivos que las métricas automatizadas no miden: satisfacción, idoneidad contextual, alineación de marca. Tiene tres variantes:

- **Pruebas interactivas en el playground:** exploras el comportamiento del modelo cualitativamente introduciendo prompts variados y anotando problemas (información incorrecta, tono inadecuado, incumplimiento de instrucciones). El playground de Foundry permite probar **modelos en paralelo**, sincronizando system prompt e indicaciones para comparar respuestas lado a lado.
- **Revisión estructurada:** creas un conjunto de casos de prueba representativos y evaluadores humanos califican cada respuesta (típicamente en escala 1-5) según criterios definidos:

  | Criterio | Pregunta que responde |
  |---|---|
  | Relevancia | ¿La respuesta aborda la pregunta o solicitud? |
  | Información | ¿Aporta detalles útiles y suficientes? |
  | Engagement | ¿Es interesante y adecuadamente conversacional? |
  | Precisión | ¿Los hechos y afirmaciones son correctos? |
  | Seguridad | ¿Evita contenido dañino, sesgado o inapropiado? |

  Las calificaciones agregadas sobre varios casos dan una medida cuantitativa de la calidad general.
- **Estudios de usuario:** recopilan feedback de usuarios reales o representativos; revelan problemas que las pruebas controladas no capturan (expresiones confusas, contexto faltante, expectativas no cubiertas).

**Cuándo usarla:** al inicio para elegir modelo y afinar el system prompt, y como complemento permanente de las métricas automatizadas. Es rápida de arrancar pero **no escalable ni reproducible** por sí sola.

### 2. Evaluación automatizada asistida por IA (LLM-as-a-judge)

Un **modelo evaluador** (se especifica un modelo GPT) analiza las respuestas del modelo implementado y asigna puntuaciones según los criterios seleccionados. Escala eficazmente y da medidas objetivas y coherentes. Foundry agrupa las métricas en dos categorías:

**a) Métricas de calidad de generación:**

- **Groundedness (base/fundamentación):** ¿la respuesta se basa en el contexto proporcionado o especula? *Groundedness Pro* da una evaluación **binaria** (fundamentada / no fundamentada), útil cuando la precisión fáctica es obligatoria.
- **Relevance:** ¿aborda adecuadamente la pregunta del usuario?
- **Coherence:** ¿fluye lógicamente y mantiene ideas consistentes?
- **Fluency:** exactitud lingüística y calidad del lenguaje natural.

**b) Métricas de riesgo y seguridad (safety):**

- Contenido de **autolesión**, **odio e injusticia** (sesgo, discriminación), **violencia** y **sexual**.
- **Material protegido:** posible reproducción de contenido con derechos de autor.
- **Ataque indirecto (jailbreak):** vulnerabilidad a intentos de manipulación.

Los resultados de daño de contenido se agregan como **tasa de defectos (defect rate)**: porcentaje de respuestas que superan un umbral de gravedad (normalmente *medio*). Para material protegido y ataque indirecto se calcula como `(instancias verdaderas / instancias totales) × 100`.

**Cuándo usarla:** en soluciones RAG y agentes donde no hay una única respuesta correcta; las safety metrics **siempre antes de producción** y de forma continua si el público es abierto.

### 3. Métricas NLP clásicas (matemáticas, sin modelo evaluador)

Comparan el texto generado contra **ground truth** (respuestas esperadas o textos de referencia):

| Métrica | Qué mide | Uso típico |
|---|---|---|
| **F1-score** | Proporción de palabras compartidas, equilibrando precisión (evitar palabras incorrectas) y recall (incluir las importantes) | Clasificación de texto, recuperación de información |
| **BLEU** | Coincidencia de n-gramas contra texto de referencia | Traducción automática |
| **METEOR** | Extiende BLEU considerando sinónimos, lematización y parafraseo | Traducción, comparación más flexible |
| **ROUGE** | Prioriza recall sobre precisión (cubrir los puntos clave) | Resumen (summarization) |
| **GLEU** | Variante de BLEU a nivel de oración | Evaluación oración por oración |

**Cuándo usarlas:** tareas cerradas con respuesta correcta definida (traducción, resumen, extracción). Son **poco adecuadas para generación abierta**, donde existen muchas respuestas válidas.

### 4. Evaluaciones completas (batch) en el portal de Foundry

La característica **Evaluation** ejecuta evaluaciones sistemáticas con datasets de prueba y varias métricas simultáneamente. Puedes basar la evaluación en:

- **Modelo:** evalúa un deployment con los prompts que especifiques; el sistema genera las salidas durante la evaluación.
- **Agente:** evalúa las respuestas de un agente ante prompts definidos por el usuario.
- **Dataset:** evalúa salidas **pregeneradas** ya presentes en el conjunto de datos.

Para el dataset de entrada tienes tres opciones: **subir uno nuevo** (CSV o JSONL con casos de prueba), **usar uno existente** del proyecto, o **generar datos sintéticos** (el sistema crea casos a partir de una descripción del tema, número de filas y archivos opcionales de contexto). El trabajo se ejecuta de forma **asincrónica** procesando cada fila con las métricas seleccionadas; al terminar muestra **puntuaciones agregadas** y el detalle por cada prompt de prueba.

La **biblioteca del evaluador (Evaluator library)** centraliza los evaluadores disponibles: evaluadores seleccionados por Microsoft para calidad, seguridad y rendimiento; prompts de anotación (para entender cómo se calcula cada métrica); definiciones y niveles de gravedad de los evaluadores de seguridad; y **evaluadores personalizados** con administración de versiones.

**Cuándo usarlas:** antes de elegir el modelo definitivo y después de cada cambio de prompt, modelo o datos (regresión).

### 5. Red teaming y pruebas adversarias

Intentos deliberados de romper el sistema: *prompt injection*, *jailbreak*, extracción de datos. Se pueden automatizar con simuladores adversarios de Azure AI Evaluation. **Cuándo usarlas:** en la fase de *medición de daños* del proceso de IA responsable, antes del lanzamiento.

### 6. Pruebas unitarias y de integración tradicionales

Pruebas de tu código (parsers, tools, APIs, orquestación) con el LLM **simulado (mock)** para que sean deterministas y baratas. **Cuándo usarlas:** siempre; el LLM no exime de probar el software alrededor de él.

### 7. Pruebas A/B y monitoreo en producción

Comparar variantes con usuarios reales y monitorear métricas operativas (latencia, costo, feedback del usuario). **Cuándo usarlas:** post-despliegue; cierran el ciclo de mejora continua.

### Iteración basada en los resultados

Los resultados de la evaluación dictan el siguiente paso, en orden creciente de complejidad y costo:

- **Calidad baja** → 1) *prompt engineering* (refinar system prompt e instrucciones), 2) probar **otros modelos** optimizados para tu caso, 3) integrar **RAG** para fundamentar respuestas en tus datos, 4) **fine-tuning** en tu dominio (si el modelo lo admite).
- **Problemas de seguridad** → filtros de contenido (**Azure AI Content Safety**), endurecimiento del prompt (instrucciones de seguridad en el system prompt) y **validación de salida** antes de mostrarla al usuario.

### Tutorial paso a paso: cómo hacer estas pruebas en Microsoft Foundry

Basado en el ejercicio oficial [Explore and compare models](https://microsoftlearning.github.io/mslearn-ai-studio/Instructions/Exercises/02-model-catalog-evaluation.html) (~45 min). Cubre en orden: prueba comparativa (benchmarks) → prueba manual en playground → evaluación automatizada batch.

> [!IMPORTANT]
> Necesitas una suscripción de Azure con permisos para crear recursos. Algunas funciones están en versión preliminar.

#### Paso 1: Crear un proyecto de Foundry

1. Abre el portal en https://ai.azure.com e inicia sesión con tus credenciales de Azure.
2. Activa la opción **New Foundry** en la barra superior si no está habilitada.
3. Crea un proyecto con nombre único. En **Advanced options** especifica: recurso de Foundry (nombre por defecto), suscripción, grupo de recursos y una región recomendada para AI Foundry.
4. Espera a que se cree el proyecto y visualiza su página de inicio.

#### Paso 2: Explorar el catálogo y comparar modelos con benchmarks

1. En la página **Discover**, selecciona la pestaña **Models** para ver el catálogo (más de 1900 modelos). Puedes filtrar por colección, capacidades, proveedor y tarea de inferencia.

   ![Catálogo de modelos en el portal de Microsoft Foundry](https://learn.microsoft.com/es-mx/training/wwl-data-ai/model-catalog-evaluate/media/model-catalog.png)

2. Busca `gpt-5.2` y abre su **model card**: revisa la descripción y la pestaña **Benchmarks** para ver su desempeño en calidad, seguridad, costo y rendimiento frente a modelos similares.
3. Regresa al catálogo y selecciona **View leaderboard**. Revisa los modelos mejor clasificados por calidad, seguridad, costo estimado y throughput.

   ![Tabla de clasificación de modelos (leaderboard)](https://learn.microsoft.com/es-mx/training/wwl-data-ai/model-catalog-evaluate/media/model-leaderboard.png)

4. Baja al **Trade-off chart** y compara dos métricas a la vez: selecciona **Benchmark Cost** en el desplegable para ver calidad vs. costo entre `gpt-5.2` y `gpt-5-mini`; repite con **Throughput** y **Safety**. Los modelos más cercanos a la esquina superior derecha rinden bien en ambas métricas.
5. En la tabla sobre los gráficos, marca las casillas de `gpt-5.2` y `gpt-5-mini` y pulsa **Compare models** para verlos lado a lado: benchmarks, ventana de contexto, endpoints y funciones soportadas.

#### Paso 3: Implementar los modelos a probar

1. En el catálogo, abre `gpt-5.2` y selecciona **Deploy** con la configuración predeterminada. Al terminar, el modelo se abre en el playground; **anota el nombre del deployment** (lo necesitarás en la evaluación).

   ![Interfaz de implementación de modelo en Foundry](https://learn.microsoft.com/es-mx/training/wwl-data-ai/model-catalog-evaluate/media/deploy-model.png)

2. En el playground, en la lista **Model**, selecciona **Browse more models**, busca `gpt-5-mini` e impleméntalo igual.

#### Paso 4: Prueba manual — comparación en paralelo en el playground

1. En el playground, con `gpt-5-mini` seleccionado en **Models**, elige el deployment de `gpt-5.2` en la lista **Compare models** (lado derecho). Se abren dos paneles de chat, uno por modelo.

   ![Playground de chat en el portal de Microsoft Foundry](https://learn.microsoft.com/es-mx/training/wwl-data-ai/model-catalog-evaluate/media/chat-playground.png)

2. Envía a ambos un prompt de razonamiento, por ejemplo:

   ```
   I have a fox, a chicken, and a bag of grain that I need to take over a river in a boat.
   I can only take one thing at a time. If I leave the chicken and the grain unattended,
   the chicken will eat the grain. If I leave the fox and the chicken unattended, the fox
   will eat the chicken. How can I get all three things across the river?
   ```

3. Envía el follow-up `Explain your reasoning.` y compara exactitud, calidad del razonamiento y estilo de respuesta. Esto es la **prueba manual exploratoria** de la sección 1.

#### Paso 5: Evaluación automatizada con dataset sintético

1. En el playground, selecciona la pestaña **Evaluations** y pulsa **Create** para abrir el asistente.
2. **Target:** selecciona **Model** y marca únicamente el deployment de `gpt-5.2` (desmarca cualquier otro). **Next**.
3. **Data:** en **Dataset source** elige **Synthetic generation** y pulsa **Generate** con: modelo `gpt-5.2`, **45 filas** y este prompt:

   ```
   Create various travel related questions, and include some content safety and security tests
   ```

   (Si ya tuvieras casos de prueba propios, aquí subirías un CSV/JSONL o usarías un dataset existente.) **Next**.
4. **Configure models:** define el **Developer prompt** del modelo evaluado:

   ```
   You are a helpful travel assistant that provides accurate, detailed, and practical
   travel advice to help users plan their trips.
   ```

5. **Criteria:** revisa los evaluadores sugeridos (usan un LLM como juez: relevance, coherence, fluency, safety…). Elimina los criterios de *Agents* y deja el resto. **Next**.
6. **Review:** verifica la configuración, nómbrala `travel-assistant-eval` y pulsa **Submit**. El trabajo corre de forma asincrónica y puede tardar varios minutos.

#### Paso 6: Revisar y analizar los resultados

1. Al completarse, abre la ejecución: verás las **puntuaciones agregadas** por métrica y una tabla con el detalle fila por fila (desplázate a la derecha para ver todas las métricas).

   ![Resultados de la evaluación en Foundry](https://learn.microsoft.com/es-mx/training/wwl-data-ai/model-catalog-evaluate/media/evaluate-model.png)

2. Pulsa **Analyze results**, selecciona `gpt-5.2` y **Start analysis**: los fallos se agrupan por causa, con sugerencias de IA para mejorar (muchos "fallos" serán rechazos correctos ante preguntas de seguridad — revísalos uno a uno).
3. Con estos resultados aplica la sección *Iteración basada en los resultados*: ajusta prompt, cambia de modelo, agrega RAG o endurece la seguridad, y **vuelve a ejecutar la evaluación** para comparar.

#### Paso 7: Limpiar recursos

En https://portal.azure.com abre el grupo de recursos del ejercicio y selecciona **Delete resource group** para no generar costos innecesarios.

### Ejemplo con el SDK azure-ai-evaluation

```python
from azure.ai.evaluation import RelevanceEvaluator, GroundednessEvaluator

model_config = {
    "azure_endpoint": "https://<tu-recurso>.openai.azure.com",
    "azure_deployment": "gpt-4o",   # modelo evaluador (LLM-as-a-judge)
}

relevance = RelevanceEvaluator(model_config)
resultado = relevance(
    query="¿Cuántos días de vacaciones tengo?",
    response="Tienes 15 días de vacaciones al año según la política interna."
)
print(resultado)  # {'relevance': 5.0, ...}

groundedness = GroundednessEvaluator(model_config)
resultado = groundedness(
    context="La política interna otorga 15 días hábiles de vacaciones anuales.",
    response="Tienes 15 días de vacaciones al año según la política interna."
)
print(resultado)  # {'groundedness': 5.0, ...}
```

### Regla mental para el examen

- ¿Estás **eligiendo** modelo? → playground (comparación en paralelo) + benchmarks + evaluación batch.
- ¿Estás **iterando** prompt/RAG? → evaluación asistida por IA (groundedness, relevance, coherence, fluency).
- ¿Hay **ground truth** y tarea cerrada (traducción, resumen)? → métricas NLP (F1, BLEU, METEOR, ROUGE, GLEU).
- ¿Vas a **producción**? → safety evaluations (tasa de defectos) + red teaming.
- ¿Ya estás **en producción**? → monitoreo continuo y A/B.
- ¿Es **código** alrededor del LLM? → pruebas unitarias/integración con mocks, siempre.
