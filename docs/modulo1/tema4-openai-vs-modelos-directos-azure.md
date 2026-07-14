# OpenAI models vs modelos directos de Azure (Phi, MAI, DeepSeek…)

### Descripción
El catálogo de Microsoft Foundry incluye miles de modelos, pero para el examen conviene distinguir dos grandes familias: los **modelos de Azure OpenAI** (GPT-4o, GPT-5, o-series, DALL·E, Whisper…) y los **modelos "directos" de Azure / Foundry Models** vendidos y hospedados por Microsoft u otros proveedores: **Phi** (small language models de Microsoft), **MAI** (modelos propios de Microsoft AI), **DeepSeek**, **Llama**, **Mistral**, **Grok**, etc. La pregunta "¿cuál es mejor?" no se responde con intuición: Foundry ofrece **pruebas comparativas (benchmarks)** con datos objetivos y medibles en cuatro dimensiones — calidad, seguridad, costo y rendimiento — y herramientas visuales (leaderboard, trade-off charts, comparación lado a lado) para decidir.

> [!NOTE]
> Módulo de referencia: [Selección de modelos mediante pruebas comparativas](https://learn.microsoft.com/es-mx/training/modules/model-catalog-evaluate/3-select-models-benchmarks)

### Aspectos Clave

1. **Modelos de Azure OpenAI:**
   - Frontier models de propósito general: mejor razonamiento, multimodalidad madura, function calling robusto.
   - Facturación por tokens; requieren deployment en tu recurso.
   - Úsalos cuando la **calidad máxima** de razonamiento/generación es el criterio dominante.

2. **Modelos directos de Microsoft (Phi, MAI):**
   - **Phi:** familia de *Small Language Models* (SLM). Baratos, rápidos, pueden correr en edge/local (incluso con Foundry Local). Excelentes para tareas acotadas: clasificación, extracción, chat sencillo.
   - **MAI:** modelos de primera parte de Microsoft (texto, voz) integrados en el ecosistema; alternativa de Microsoft a los modelos de OpenAI.

3. **Modelos de terceros "sold directly by Azure" (DeepSeek, Llama, Mistral, Grok…):**
   - Se implementan como **serverless API (pago por token)** o en **cómputo administrado**.
   - **DeepSeek-R1** destaca en razonamiento a bajo costo; Llama/Mistral ofrecen apertura (open weights) y flexibilidad de fine-tuning.
   - Cobertura de responsabilidad y SLA de Microsoft cuando son "Direct from Azure".

4. **Diferencias prácticas que caen en examen:**

| Criterio | Azure OpenAI | Modelos directos (Phi, DeepSeek, etc.) |
|---|---|---|
| Calidad frontier | ✔ Máxima | Variable (R1 compite en razonamiento) |
| Costo por token | Más alto | Generalmente menor |
| Latencia | Media | Phi = muy baja |
| Edge / local | ✖ | ✔ (Phi con Foundry Local / ONNX) |
| Open weights | ✖ | ✔ en Llama, Mistral, Phi |
| Fine-tuning | Sí, en modelos seleccionados | Sí, más flexible en abiertos |

5. **Cómo se consumen:** todos se exponen a través del proyecto de Foundry con una API unificada; cambiar de GPT-4o a Phi-4 puede ser solo cambiar el nombre del deployment:

```python
response = chat_client.complete(
    model="Phi-4",          # o "gpt-4o", "DeepSeek-R1"...
    messages=[{"role": "user", "content": "Clasifica este ticket: 'no puedo entrar al portal'"}]
)
```

### Las 4 dimensiones de benchmark para comparar familias

Las puntuaciones comparativas son **índices normalizados de 0 a 1** (más alto = mejor rendimiento), calculadas con conjuntos de datos públicos y métodos de evaluación estandarizados. Se acceden de dos formas: la **tabla de clasificación (model leaderboard)** en el catálogo para comparar entre modelos, y la pestaña **Benchmarks** de la tarjeta de cada modelo para el detalle individual.

#### a) Calidad

El **índice de calidad** promedia las puntuaciones de precisión en varios datasets que miden razonamiento, conocimiento, respuesta a preguntas, matemáticas y codificación:

| Dataset | Qué mide |
|---|---|
| **Arena-Hard** | Respuesta a preguntas adversariales |
| **BIG-Bench Hard** | Capacidades de razonamiento |
| **GPQA** | Preguntas multidisciplina de nivel posgrado |
| **HumanEval+ / MBPP+** | Generación de código |
| **MATH** | Razonamiento matemático |
| **MMLU-Pro** | Conocimiento general |
| **IFEval** | Seguimiento de instrucciones |

#### b) Seguridad

Cruciales para apps expuestas a usuarios finales y sectores regulados. Tres evaluaciones con **interpretaciones opuestas** (favorito de examen):

| Benchmark | Métrica | Interpretación |
|---|---|---|
| **HarmBench** | ASR (*attack success rate*) | **Menor = más seguro.** Prueba 3 áreas: comportamientos dañinos estándar (ciberdelincuencia, actividades ilegales), contextualmente dañinos (desinformación, acoso) e infracciones de copyright |
| **ToxiGen** | F1 | **Mayor = mejor** detección de discurso de odio implícito hacia grupos minoritarios |
| **WMDP** | Puntuación de conocimiento | **Mayor = más conocimiento potencialmente peligroso** (bioseguridad, ciberseguridad, seguridad química) — no es "mejor" |

#### c) Costo

- **Costo por tokens de entrada:** precio por procesar 1 millón de tokens de entrada.
- **Costo por tokens de salida:** precio por generar 1 millón de tokens de salida.
- **Costo estimado:** combina ambos con una relación típica **3:1** (tres tokens de entrada por cada uno de salida) para dar un solo número comparable. Menor = más rentable.

#### d) Rendimiento (performance)

Determinante para aplicaciones en tiempo real:

- **Latencia:** media, **P50** (mediana), **P90**, **P95**, **P99** (el 99% de las solicitudes se completa más rápido que ese tiempo) y **TTFT** (*time to first token*, clave con streaming).
- **Throughput:** **GTPS** (tokens generados por segundo), **TTPS** (tokens totales por segundo, entrada + salida) y **tiempo entre tokens**.
- El leaderboard resume rendimiento con **TTFT medio (menor = mejor)** y **GTPS (mayor = mejor)**.
- Para trabajos **batch** donde la velocidad importa menos que el costo, prioriza otras dimensiones.

### Tutorial: cómo encontrar el mejor modelo con las herramientas de Foundry

Flujo completo en el portal (https://ai.azure.com), basado en el [ejercicio oficial](https://microsoftlearning.github.io/mslearn-ai-studio/Instructions/Exercises/02-model-catalog-evaluation.html):

#### Paso 1: Define tus criterios antes de abrir el portal

Escribe qué pesa más en tu caso: ¿calidad máxima, costo por token, latencia interactiva, seguridad ante usuarios abiertos? ¿Tu app corresponde a un escenario concreto (razonamiento, código, matemáticas, QA, groundedness)? Sin esto, el leaderboard solo te dirá "cuál es bueno en general", no cuál es mejor **para ti**.

#### Paso 2: Filtra candidatos en el catálogo

En **Discover → Models**, filtra por colección, capacidades (razonamiento, tool calling, multimodal), proveedor y tarea de inferencia. Abre la **model card** de cada candidato y revisa su pestaña **Benchmarks** para ver sus gráficos frente a modelos similares.

![Catálogo de modelos en el portal de Microsoft Foundry](https://learn.microsoft.com/es-mx/training/wwl-data-ai/model-catalog-evaluate/media/model-catalog.png)

#### Paso 3: Abre la tabla de clasificación (leaderboard)

Selecciona **View leaderboard** y ordena por **calidad, seguridad, costo estimado o rendimiento**. Si tu app se asigna a un escenario específico, empieza por la **tabla de clasificación del escenario** (razonamiento, codificación, matemáticas, QA, groundedness) en lugar del índice de calidad general.

![Tabla de clasificación de modelos (leaderboard)](https://learn.microsoft.com/es-mx/training/wwl-data-ai/model-catalog-evaluate/media/model-leaderboard.png)

#### Paso 4: Usa los gráficos de equilibrio (trade-off charts)

Los trade-off charts muestran **dos métricas a la vez**. En el desplegable, compara calidad vs. **Benchmark Cost**, luego vs. **Throughput** y vs. **Safety**, agregando tus candidatos a la comparación (p. ej., `gpt-5.2` vs. `gpt-5-mini` vs. `Phi-4` vs. `DeepSeek-R1`). Los modelos más cercanos a la **esquina superior derecha** rinden bien en ambas métricas. Aquí es donde suele ganar un modelo directo: uno ligeramente menos preciso pero mucho más rápido o barato puede satisfacer mejor tus necesidades.

#### Paso 5: Comparación en paralelo (side-by-side)

En la tabla del leaderboard, marca las casillas de 2 o 3 modelos y pulsa **Compare models** para verlos lado a lado en:

- **Benchmarks de rendimiento:** calidad, seguridad y throughput.
- **Detalles del modelo:** ventana de contexto, fecha de datos de entrenamiento, idiomas.
- **Endpoints soportados:** opciones de implementación y si puede usarse en agentes.
- **Funciones:** function calling, salida estructurada, visión.

Descarta cualquier candidato que no soporte una función que tu app requiere, aunque gane en benchmarks.

#### Paso 6: Valida los finalistas con TUS datos

Los benchmarks usan datasets públicos genéricos; la decisión final se toma con tu caso de uso:

1. **Implementa los 2 finalistas** (Deploy con configuración predeterminada).
2. **Prueba manual en paralelo:** en el playground, usa **Compare models** con prompts reales de tu dominio y compara exactitud, razonamiento y estilo (ver tutorial del tema 2, paso 4).
3. **Evaluación batch:** crea una evaluación (pestaña **Evaluations**) con tu dataset o uno sintético y métricas asistidas por IA — relevance, coherence, safety (ver tema 2, paso 5). Repite la misma evaluación con cada finalista y compara puntuaciones agregadas.

#### Paso 7: Decide y documenta

Elige balanceando las 4 dimensiones contra tus criterios del paso 1. Documenta la decisión (modelo, versión, benchmarks y resultados de tu evaluación) — el catálogo cambia constantemente y querrás repetir la comparación cuando aparezcan modelos nuevos: la evaluación batch guardada es tu prueba de regresión.

### Regla mental

- Tarea compleja, multimodal o crítica → **GPT (Azure OpenAI)**.
- Tarea sencilla, masiva, sensible a costo/latencia o en edge → **Phi**.
- Razonamiento fuerte con presupuesto limitado → **DeepSeek-R1**.
- Necesidad de pesos abiertos o fine-tuning profundo → **Llama / Mistral / Phi**.
- ¿App interactiva en tiempo real? → prioriza **TTFT bajo y GTPS alto**; ¿batch masivo? → prioriza **costo**.
- Interpretación de métricas de seguridad (trampa de examen): **ASR menor = más seguro**, **ToxiGen F1 mayor = mejor**, **WMDP mayor = más conocimiento peligroso**.
- Benchmarks genéricos eligen los **finalistas**; tu **evaluación con datos propios** elige al ganador.

### Tip
Combinar familias (por ejemplo, Phi como "router" barato que escala a GPT-4o solo cuando la tarea lo amerita) reduce costos drásticamente sin sacrificar calidad donde importa. Y decidir con benchmarks + evaluación propia en lugar de intuición te da evidencia objetiva y repetible ante cualquier cambio del catálogo.
