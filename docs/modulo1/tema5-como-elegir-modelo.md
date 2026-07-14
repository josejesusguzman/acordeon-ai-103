# Cómo elegir el mejor modelo de IA para ti

### Descripción
Elegir modelo no es buscar "el mejor del mundo" sino el mejor **para tu caso**: tarea, calidad requerida, costo, latencia, contexto, seguridad y región. Microsoft Learn propone un proceso de filtrado + benchmarks + evaluación propia.

> [!NOTE]
> Módulo de referencia: [Exploración y evaluación del catálogo de modelos](https://learn.microsoft.com/es-es/training/modules/model-catalog-evaluate/)

### Proceso recomendado (4 pasos)

1. **Define la tarea:** ¿chat, razonamiento, código, embeddings, visión, audio, generación de imágenes? El catálogo de Foundry se filtra por *inference task*, proveedor, licencia y opciones de implementación.
2. **Filtra por restricciones duras:**
   - **Región y cuota** disponibles en tu suscripción.
   - **Ventana de contexto** necesaria (¿32k, 128k, 1M tokens?).
   - **Modalidad** (texto, imagen, audio, video).
   - **Licencia y compliance** (¿puedes usar open weights? ¿necesitas SLA?).
3. **Compara con benchmarks del catálogo:** Foundry muestra métricas comparativas (calidad, costo, throughput) y leaderboards. Úsalas para armar una *shortlist* de 2-3 modelos, no para decidir a ciegas.
4. **Evalúa con TUS datos:** corre una evaluación batch con prompts reales de tu dominio y métricas como groundedness/relevance. El ganador en benchmarks genéricos no siempre gana en tu caso.

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

### Beneficios
Un proceso de selección disciplinado evita las dos trampas clásicas: pagar GPT-frontier para tareas que un Phi resuelve, o forzar un modelo barato en tareas donde falla y erosiona la confianza del usuario.
