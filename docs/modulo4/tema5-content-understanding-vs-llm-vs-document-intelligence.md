# Content Understanding vs modelos LLM multimodales vs Document Intelligence

### Descripción
Para "sacar información de documentos/imágenes/audio/video" Azure ofrece TRES caminos que se traslapan. Elegir mal cuesta dinero o precisión. Esta es la comparación que el examen (y la vida real) te va a pedir.

> [!NOTE]
> Módulos de referencia: [Analyze content with Content Understanding](https://learn.microsoft.com/es-es/training/modules/analyze-content-ai/), [Extract data with Document Intelligence](https://learn.microsoft.com/es-es/training/modules/extract-data-with-document-intelligence/) y [Develop generative AI vision apps](https://learn.microsoft.com/es-es/training/modules/develop-generative-ai-vision-apps/)

### Los tres contendientes

1. **Azure AI Document Intelligence:**
   - Servicio especializado en **documentos**: OCR de alta precisión + modelos preconstruidos (facturas, recibos, identificaciones, W-2, tarjetas de presentación) + modelos custom entrenables con TUS formularios.
   - Salida: JSON estructurado con campos, tablas, pares clave-valor, **coordenadas (bounding boxes)** y scores de confianza.
   - Fortalezas: precisión y trazabilidad documental (sabes DÓNDE estaba cada dato), procesamiento batch masivo, costos predecibles por página.

2. **Azure AI Content Understanding:**
   - Servicio **multimodal** de nueva generación: define un **analyzer** con un esquema de campos (field schema) y lo aplica a **documentos, imágenes, audio Y video** — extrae los campos que definas usando IA generativa por debajo.
   - Salida: JSON con tus campos + metadatos (transcripciones, caras/escenas en video, etc.).
   - Fortalezas: un solo servicio para todas las modalidades, esquemas declarativos sin entrenar modelos, razonamiento generativo sobre el contenido ("clasifica el tono de la llamada").

3. **LLM multimodal directo (GPT-4o/GPT-5 con visión):**
   - Le mandas la imagen/documento en el prompt y pides lo que sea, formato libre o JSON (structured outputs).
   - Fortalezas: máxima flexibilidad y razonamiento (comparar, resumir, decidir), cero configuración.
   - Debilidades: sin bounding boxes ni confianza por campo, costo por tokens alto en volumen, OCR fino inferior a Document Intelligence en escaneos difíciles, y salidas menos deterministas.

### Tabla de decisión

| Escenario | Elige |
|---|---|
| Facturas/recibos/IDs estándar a escala | **Document Intelligence** (prebuilt) |
| Formularios propios con layout fijo, máxima precisión y auditoría | **Document Intelligence** (custom) |
| Necesitas bounding boxes y confianza por campo (compliance) | **Document Intelligence** |
| Extraer el mismo esquema de PDFs + audios + videos | **Content Understanding** |
| Analizar llamadas de call center o videos (transcribir + clasificar + resumir) | **Content Understanding** |
| Campos definidos declarativamente sin entrenar nada | **Content Understanding** |
| Preguntas abiertas sobre una imagen/documento ("¿qué riesgos ves en este contrato?") | **LLM multimodal** |
| Prototipo rápido / volumen bajo / lógica conversacional | **LLM multimodal** |
| Chat sobre documentos (RAG) | LLM + AI Search (y DI/CU para indexar) |

### Regla mental
- **Estructura conocida + volumen + precisión auditable** → Document Intelligence.
- **Esquema propio multi-modalidad (docs, audio, video)** → Content Understanding.
- **Razonamiento abierto y flexible** → LLM multimodal.

### El patrón combinado (arquitectura típica)
```text
Documentos/llamadas/videos
   └─> Content Understanding / Document Intelligence  (extracción estructurada confiable)
         └─> JSON a base de datos / índice de AI Search
               └─> LLM/agente razona SOBRE los datos extraídos (chat, decisiones, resúmenes)
```
Extraes con el servicio especializado y razonas con el LLM: cada pieza en lo que hace mejor.

### Beneficios
Esta separación de responsabilidades da precisión de servicio especializado con inteligencia de LLM, a costo controlado — y es la respuesta "correcta" en la mayoría de escenarios de examen que mezclan documentos e IA generativa.
