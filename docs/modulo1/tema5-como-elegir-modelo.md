# Cómo elegir el mejor modelo de IA para ti

### Descripción
Elegir modelo no es buscar "el mejor del mundo" sino el mejor **para tu caso**: tarea, calidad requerida, costo, latencia, contexto, seguridad y región. Microsoft propone un proceso disciplinado: filtrar por restricciones → comparar con benchmarks públicos (leaderboard, trade-off charts, **comparador en paralelo**) → validar con evaluaciones sobre tus propios datos → **comparar las ejecuciones de evaluación** con pruebas estadísticas para decidir con evidencia y no con intuición.

> [!NOTE]
> Referencias: [Comparación de modelos con la tabla de clasificación](https://learn.microsoft.com/es-mx/azure/foundry/how-to/benchmark-model-in-catalog), [Visualización de resultados de evaluación](https://learn.microsoft.com/es-mx/azure/foundry/how-to/evaluate-results) y el módulo [Selección y evaluación de modelos](https://learn.microsoft.com/es-mx/training/modules/model-catalog-evaluate/)

### Proceso de decisión completo (7 pasos)

1. **Define la tarea:** ¿chat, razonamiento, código, embeddings, visión, audio, generación de imágenes? El catálogo de Foundry se filtra por *inference task*, proveedor, licencia y opciones de implementación.
2. **Filtra por restricciones duras:**
   - **Región y cuota** disponibles en tu suscripción.
   - **Ventana de contexto** necesaria (¿32k, 128k, 1M tokens?).
   - **Modalidad** (texto, imagen, audio, video) y funciones requeridas (function calling, salida estructurada, uso en agentes).
   - **Licencia y compliance** (¿puedes usar open weights? ¿necesitas SLA?).
3. **Consulta el leaderboard:** en **Discover**, la página de información general muestra una instantánea de la tabla de clasificación; con **Ir a la tabla de clasificación** ves la lista completa, ordenable por calidad, seguridad, costo estimado y rendimiento, con gráficos de barras expandibles del top 10 por métrica. Si tu app corresponde a un escenario (razonamiento, codificación, respuesta a preguntas…), usa las **tablas de clasificación por escenario** en lugar del índice general.
4. **Balancea con los trade-off charts:** el modelo de mayor calidad rara vez es el más barato. El gráfico de compensación traza dos métricas a la vez (desplegable **Comparar calidad con** → costo, rendimiento o seguridad); agrega o quita modelos con el selector de la derecha y pasa el cursor sobre un punto para ver sus puntuaciones exactas. Los modelos cercanos a la **esquina superior derecha** rinden bien en ambos ejes.
5. **Usa el comparador en paralelo** para tu shortlist (detalle en la siguiente sección).
6. **Evalúa con TUS datos:** los benchmarks usan datasets públicos estandarizados y pueden no reflejar tu dominio. Corre una **evaluación batch por cada finalista** con el mismo dataset (propio o sintético) y las mismas métricas (ver tema 2, paso 5).
7. **Compara las ejecuciones de evaluación** con la vista de comparación estadística del portal (tutorial al final) y elige al ganador con significancia estadística, no a ojo.

### El comparador de modelos a fondo

La vista de comparación en paralelo evalúa **hasta 3 modelos** simultáneamente:

1. En la tabla de clasificación, marca las casillas de 2 o 3 modelos.
2. Pulsa **Comparar** para abrir la vista en paralelo.
3. Revisa las cuatro pestañas:
   - **Pruebas comparativas de rendimiento:** puntuaciones de calidad, seguridad y rendimiento en datasets públicos.
   - **Detalles del modelo:** ventana de contexto, fecha de datos de entrenamiento, idiomas admitidos.
   - **Puntos de conexión admitidos:** API serverless, cómputo administrado, uso en agentes.
   - **Soporte de características:** function calling, salida estructurada, visión.
4. Desde ahí: **Ver detalles** para profundizar o **Implementar** para desplegar al elegido.

![Vista de comparación de modelos en Microsoft Foundry](https://learn.microsoft.com/es-mx/azure/foundry/how-to/media/observability/model-benchmarks/compare-model-overview.png)

Para el detalle individual, abre la pestaña **Pruebas comparativas (Benchmarks)** de la model card: muestra puntuaciones agregadas, **gráficos comparativos** contra modelos relacionados y la **tabla de comparación de métricas**. Expande cualquier gráfico para elegir métrica y dataset específicos (con **Leer más** para las definiciones).

![Tabla de comparación de métricas en la pestaña Benchmarks](https://learn.microsoft.com/es-mx/azure/foundry/how-to/media/observability/model-benchmarks/benchmark-overview.png)

**Limitaciones a saber (caen en examen):** no todos los modelos tienen benchmarks publicados (sin pestaña Benchmarks = sin datos aún); el máximo del comparador son 3 modelos; las puntuaciones son índices normalizados (calidad/seguridad: mayor = mejor; costo: menor = mejor); la tabla de clasificación está en versión preliminar; y la opción "Probar con sus propios datos" solo existe en Foundry clásico — en el portal nuevo eso se hace con **evaluaciones**.

### Criterios y su modelo típico

| Necesidad dominante | Elección típica |
|---|---|
| Razonamiento complejo, agentes exigentes | GPT-5 / o-series / DeepSeek-R1 |
| Chat de propósito general costo-efectivo | GPT-4o-mini |
| Volumen masivo, baja latencia, edge | Phi (SLM) |
| Embeddings para RAG | text-embedding-3-large |
| Visión (analizar imágenes) | GPT-4o / GPT-5 (multimodal) |
| Generar imágenes | DALL·E 3 / GPT-image / FLUX |
| Voz en tiempo real | gpt-realtime / Voice Live |
| Video | Sora |

### Trade-offs que el examen adora

1. **Calidad vs costo:** modelos más grandes = más caros; usa el pequeño que apruebe tu evaluación, no el grande "por si acaso".
2. **Latencia vs razonamiento:** los modelos de razonamiento (o-series, R1) "piensan" más y tardan más; para UX conversacional fluida suele ganar un modelo estándar.
3. **Serverless vs managed compute:** pago por token (simple, elástico) vs GPU dedicada (control, modelos custom, costo fijo).
4. **Model Router:** Foundry ofrece enrutamiento automático que elige por ti el modelo óptimo por prompt para bajar costos manteniendo calidad; opción legítima cuando dudas.

### Tutorial: ver las gráficas de comparación de evaluaciones en Foundry

Este es el paso final del proceso: ya corriste una evaluación por cada modelo finalista (mismo dataset, mismos criterios) y ahora las comparas en el portal.

> [!IMPORTANT]
> Requisitos: rol **Foundry User** en el proyecto y al menos dos ejecuciones de evaluación **completadas**. Para que la comparación sea justa, las ejecuciones deben usar el mismo dataset y los mismos evaluadores.

#### Paso 1: Abre la lista de ejecuciones

En el [portal de Foundry](https://ai.azure.com), entra a tu proyecto y selecciona **Evaluación** en el panel izquierdo. La lista muestra cada ejecución con: nombre, **objetivo** (modelo o agente evaluado), dataset (descargable como CSV), estado (*En ejecución / Completado / Error*), **tokens de evaluación** (los que consumieron los evaluadores), **tokens de destino** (los que consumió el modelo evaluado) y las **puntuaciones agregadas** por evaluador.

![Lista de ejecuciones de evaluación con sus puntuaciones](https://learn.microsoft.com/es-mx/azure/foundry/media/observability/evaluation-runs.png)

#### Paso 2: Inspecciona las puntuaciones

Mantén el puntero sobre cualquier celda de puntuación para ver el desglose de uso de tokens y contexto adicional. El enlace **Más información sobre las métricas** muestra definiciones y fórmulas de cada evaluador.

![Tooltip con desglose de tokens al pasar el cursor sobre una puntuación](https://learn.microsoft.com/es-mx/azure/foundry/media/observability/evaluation-runs-hover.png)

#### Paso 3: Revisa el detalle por fila

Selecciona el nombre de una ejecución para abrir sus resultados fila por fila: consulta, respuesta generada, verdad fundamental (ground truth), puntuación del evaluador y la **explicación** de por qué se asignó esa puntuación.

![Resultados de una ejecución de evaluación](https://learn.microsoft.com/es-mx/training/wwl-data-ai/model-catalog-evaluate/media/evaluate-model.png)

#### Paso 4: Genera la vista de comparación

1. En la página de Evaluación, selecciona **dos o más ejecuciones** (una por modelo candidato).
2. Pulsa **Compare**: se abre la comparación en paralelo de todas las ejecuciones seleccionadas.

#### Paso 5: Establece una línea base (baseline)

Marca una ejecución como **línea base** (por ejemplo, tu modelo actual en producción o el candidato favorito). La vista mostrará cómo cada otra ejecución **se desvía** de ese punto de referencia.

#### Paso 6: Interpreta las pruebas t estadísticas

Cada celda de la comparación trae un resultado de significancia estadística con código de color; pasa el cursor para ver el **tamaño de muestra** y el **valor p**:

| Leyenda | Significado |
|---|---|
| **ImprovedStrong** | Altamente significativo (p ≤ 0.001) y en la dirección deseada |
| **ImprovedWeak** | Significativo (0.001 < p ≤ 0.05) y en la dirección deseada |
| **DegradedStrong** | Altamente significativo (p ≤ 0.001) y en dirección incorrecta |
| **DegradedWeak** | Significativo (0.001 < p ≤ 0.05) y en dirección incorrecta |
| **ChangedStrong / ChangedWeak** | Significativo, con dirección deseada neutral |
| **Inconclusive** | Muy pocos ejemplos o p ≥ 0.05 — no concluyas nada |

Decide con esto: un modelo "mejor por 0.2 puntos" con resultado *Inconclusive* **no** es mejor; exige *ImprovedWeak* o superior antes de declarar ganador.

> [!NOTE]
> La vista de comparación **no se guarda**: si sales de la página, vuelve a seleccionar las ejecuciones y pulsa **Comparar** de nuevo.

#### Solución de problemas rápida

| Síntoma | Acción |
|---|---|
| Faltan métricas en una ejecución | No se seleccionaron al crearla; re-ejecuta con las métricas necesarias |
| Métricas de seguridad todas en cero | Categoría deshabilitada o modelo no compatible con esos evaluadores |
| Groundedness inesperadamente bajo | Revisa la recuperación/contexto (RAG incompleto), no solo el modelo |
| La ejecución queda pendiente | Carga alta del servicio; verifica cuota y reenvía |

### Regla mental

- Benchmarks públicos = **shortlist** (2-3 modelos); evaluación con tus datos + comparación estadística = **decisión**.
- Comparador en paralelo: máximo **3 modelos**, 4 pestañas (benchmarks, detalles, endpoints, características).
- Trade-off chart: esquina **superior derecha** = bueno en ambas métricas.
- Comparación de evaluaciones: **línea base + prueba t**; p ≥ 0.05 = *Inconclusive* = no hay diferencia demostrada.
- Sin pestaña Benchmarks = benchmarks no publicados para ese modelo, no que sea malo.


