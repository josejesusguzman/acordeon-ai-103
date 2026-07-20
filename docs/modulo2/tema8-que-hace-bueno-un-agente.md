# Qué hace bueno a un agente de IA (calidad de retrieval y de respuesta)

### Descripción
Puedes tener el contenido perfectamente indexado y aun así un agente malo. La diferencia está en **cómo el agente usa el conocimiento** — cuándo busca, cómo cita, qué hace cuando no sabe — y en **cómo lo verificas**: leyendo sus respuestas con criterio (señales manuales) y midiendo con los **evaluadores integrados de Microsoft Foundry**. Este tema cubre ambas lentes y cierra con un tutorial para revisarlo en el portal.

> [!NOTE]
> Referencias: [Configure retrieval with Foundry IQ](https://learn.microsoft.com/es-mx/training/modules/introduction-foundry-iq/5-configure-retrieval) y [Referencia de evaluadores integrados](https://learn.microsoft.com/es-mx/azure/foundry/concepts/built-in-evaluators)

### Lente 1: leer las respuestas (señales manuales)

#### El problema de comportamiento de retrieval
Ante "¿cuál es nuestra política de vacaciones?" hay tres comportamientos posibles y solo uno aceptable:

| Comportamiento | Ejemplo | Problema |
|---|---|---|
| Responde desde su entrenamiento | "La mayoría de empresas dan 2-3 semanas…" | Información genérica, no TU política |
| Busca pero no cita | "Tienes 15 días de PTO" | Correcto pero no verificable |
| **Busca, cita y se fundamenta** | "Tienes 15 días de PTO 【doc:Manual del Empleado 2024】" | ✔ Lo que quieres |

#### Las 4 características de una buena respuesta
1. **Grounding:** la información viene de la knowledge base, no del entrenamiento.
2. **Citation:** cada afirmación factual referencia su fuente.
3. **Relevance:** lo recuperado responde de verdad la pregunta.
4. **Completeness:** la respuesta está completa, no en fragmentos.

#### Qué probar (tipos de query)

| Tipo de pregunta | Ejemplo | Comportamiento esperado |
|---|---|---|
| Factual directa | "¿Cuál es la política de vacaciones?" | Retrieval directo con citas |
| Requiere síntesis | "¿Diferencias entre tipos de permisos?" | Varias fuentes, respuesta sintetizada con múltiples citas |
| Fuera de la base | "¿Qué clima hace hoy?" | Fallback elegante ("no tengo esa información…") |
| Ambigua | "¿Y los beneficios?" | Pregunta aclaratoria o búsqueda enfocada |

#### Instrucciones efectivas de retrieval (las 3 reglas)
Una instrucción vaga ("usa la base de conocimiento") produce comportamiento inconsistente. Las efectivas especifican **cuándo buscar** (siempre), **cómo citar** (formato exacto) y **qué hacer si no sabe** (fallback definido):

```python
retrieval_instructions = """Eres un asistente de RRHH.
REGLAS CRÍTICAS:
- SIEMPRE busca en la knowledge base antes de responder.
- NUNCA respondas desde tu conocimiento de entrenamiento.
- Toda respuesta debe incluir citas: 【doc_id:search_id†nombre_fuente】
- Si la knowledge base no contiene la respuesta: "No tengo esa información
  en la documentación actual. Contacta a RRHH."
"""
```

### Lente 2: las métricas de evaluación de Foundry

La lectura manual no escala; los **evaluadores integrados** convierten "se ve bien" en números comparables. Los que importan para agentes:

#### Evaluadores de agente (miden el CICLO completo, no solo el texto)

| Evaluador | Qué responde |
|---|---|
| **Resolución de intenciones** | ¿Entendió qué quería el usuario? |
| **Cumplimiento de tareas (task adherence)** | ¿Siguió sus instrucciones sin desviarse? |
| **Finalización de tareas** | ¿Completó la tarea de extremo a extremo? |
| **Selección de herramientas** | ¿Eligió la tool adecuada? |
| **Precisión de entrada de herramienta** | ¿Los parámetros que pasó eran correctos (tipo, formato, fundamentados)? |
| **Llamada correcta de herramienta** | ¿Las llamadas se ejecutaron sin errores técnicos? |
| **Uso de la salida de herramienta** | ¿Interpretó bien lo que la tool devolvió? |
| **Precisión de llamadas de herramienta** | Calidad global del uso de tools (selección + parámetros + eficacia) |
| **Eficiencia de navegación de tareas** | ¿La secuencia de pasos fue la óptima o dio vueltas? |
| **Satisfacción del cliente** | Satisfacción holística de la conversación (utilidad, claridad, tono, resolución…) |
| **Quality grader** | Varias dimensiones de calidad (relevancia, abstención, integridad, grounding) en un solo evaluador |

#### Evaluadores RAG (miden el CONOCIMIENTO)

**Retrieval** (¿recuperó lo relevante?), **Document retrieval** (precisión vs. ground truth), **Groundedness** (1-5, ¿la respuesta se apoya en el contexto?), **Groundedness Pro** (binario pasa/falla vía Content Safety, sin necesidad de modelo juez), **Relevance** y **Response completeness**.

#### Evaluadores de seguridad específicos de agentes

Además de las 4 categorías clásicas: **acciones prohibidas** (¿hizo algo explícitamente vetado?), **pérdida de datos confidenciales**, **atributos sin fundamento** (¿inventó datos sobre el usuario?) y **ataque indirecto XPIA** (¿cayó ante inyección vía contexto recuperado?).

#### Dos detalles de examen

- **Nivel de evaluación:** `turn` (respuesta individual, el default) vs. `conversation` (la conversación completa — necesario para satisfacción del cliente o tareas multi-paso). No puedes mezclar evaluadores de niveles incompatibles en la misma ejecución.
- **Combinación recomendada oficial para agentes:** precisión de llamadas de herramienta + cumplimiento de tareas + resolución de intenciones + rubric + seguridad de contenido.

### Diagnóstico: del número a la causa

| Síntoma en las métricas | Causa probable | Arréglalo en… |
|---|---|---|
| Intent resolution alta, tool accuracy baja | Descripciones/esquemas de tools confusos | Definición de las tools |
| Groundedness baja, retrieval alta | El agente ignora el contexto recuperado | Instrucciones (reglas de grounding) |
| Retrieval baja | La knowledge base no tiene o no encuentra el contenido | Fuentes/índice (tema 7) |
| Task adherence baja | Instrucciones vagas o contradictorias | System prompt (checklist tema 6 M1) |
| Task completion baja, adherence alta | Sigue las reglas pero no cierra: faltan tools o pasos | Herramientas / orquestación |
| Fallback casi nunca ("no sé" = 0%) | Probablemente alucina en vez de abstenerse | Instrucciones de fallback |
| Fallback altísimo | Faltan contenidos en la base | Knowledge base |

### Tutorial: revisar las métricas de tu agente en Foundry

#### Paso 1: Evaluación en vivo en el agent playground

Mientras pruebas el agente en el playground, las **evaluaciones del playground** corren por defecto sobre cada respuesta. En el selector de **métricas** (esquina superior derecha) eliges qué evaluadores aplicar y ves sus puntuaciones junto a cada respuesta — tu primera lectura de calidad, sin montar nada:

![Agent playground con el selector de métricas de evaluación](https://learn.microsoft.com/es-mx/azure/foundry/media/observability/agent-playground-evaluation-metrics.png)

(Se facturan por consumo: desactívalas si solo estás explorando.)

#### Paso 2: Corre una evaluación batch del agente

En **Evaluations → Create**: elige **Agent** como objetivo, tu dataset (propio o sintético, tema 2 M1) y en **Criteria** deja los evaluadores de la sección *Agents* (intent resolution, task adherence, tool call accuracy…) más los de calidad y seguridad. Envía y espera la ejecución asincrónica.

#### Paso 3: Lee las puntuaciones agregadas

En la lista de ejecuciones ves las **puntuaciones por evaluador**, el objetivo (tu agente), los tokens consumidos por evaluadores y por el agente; pasa el cursor sobre una celda para el desglose:

![Lista de ejecuciones de evaluación con puntuaciones por evaluador](https://learn.microsoft.com/es-mx/azure/foundry/media/observability/evaluation-runs.png)

![Tooltip con el desglose al pasar el cursor](https://learn.microsoft.com/es-mx/azure/foundry/media/observability/evaluation-runs-hover.png)

#### Paso 4: Baja al detalle por fila (aquí vives o mueres)

Abre la ejecución: cada fila muestra consulta, respuesta, ground truth, puntuación **y la explicación del evaluador** de por qué asignó ese número. Las explicaciones son tu herramienta principal: te dicen *qué* le pareció mal al juez, no solo cuánto:

![Resultados fila por fila de una evaluación](https://learn.microsoft.com/es-mx/training/wwl-data-ai/model-catalog-evaluate/media/evaluate-model.png)

#### Paso 5: Agrupa los fallos con Analyze results

Pulsa **Analyze results** → selecciona el agente → **Start analysis**: los fallos se **agrupan por causa** (análisis de clústeres) con sugerencias de mejora de IA. Cuidado con los falsos fallos: un rechazo correcto ante una pregunta insegura puede aparecer como "fallo" — revísalos uno a uno.

#### Paso 6: Monitorea en producción (pestaña Monitoring)

En **Build → tu agente → Monitoring** (con Application Insights conectado) ves el dashboard continuo: puntuaciones de evaluación sobre tráfico real muestreado, tasa de éxito de ejecución (<95% = investiga), latencia (>10 s = cuello de botella) y tokens:

![Dashboard de monitoreo del agente](https://learn.microsoft.com/es-mx/azure/foundry/media/observability/how-to-monitor-agents-dashboard/foundry-metrics-dashboard.png)

Desde el engrane configuras **evaluación continua** (con tasa de muestreo), **evaluaciones programadas**, **red teaming programado** y **alertas**:

![Panel de configuración del monitoreo](https://learn.microsoft.com/es-mx/azure/foundry/media/observability/how-to-monitor-agents-dashboard/monitor-settings-panel.png)

#### Paso 7: Cierra el ciclo

Compara ejecuciones antes/después de cada cambio con la vista estadística (tema 5 M1: exige *Improved*, no *Inconclusive*) y convierte cada fallo real en caso nuevo del dataset.

### En producción, monitorea además
- **Frecuencia de citación** (¿sigue citando?)
- **Frecuencia de fallback** (¿cuánto dice "no sé"? — mucho = faltan contenidos; nada = quizá alucina)
- **Tipos de query reales** (los usuarios preguntan distinto que tus pruebas)
- **Exactitud de retrieval** (¿los documentos recuperados contienen la respuesta?)

### Beneficios
Un agente "bueno" es un agente **predecible y medido**: siempre busca, siempre cita, sabe decir "no sé", y tú puedes demostrarlo con números — resolución de intenciones, adherencia, precisión de tools, groundedness — en vez de con anécdotas. Instrucciones explícitas + evaluadores correctos + monitoreo continuo, no un mejor modelo.
