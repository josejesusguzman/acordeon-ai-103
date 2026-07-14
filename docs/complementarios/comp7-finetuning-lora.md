# Tema Complementario 7: Fine-tuning avanzado — LoRA, datasets y cuándo vale la pena

### Descripción
El Módulo 1 explica *cuándo* hacer fine-tuning; este tema explica *cómo* funciona por dentro (LoRA/QLoRA), cómo preparar datos que sí funcionen y las variantes (DPO, destilación) que un ingeniero intermedio debe conocer.

### Aspectos Clave

1. **Full fine-tuning vs PEFT:** re-entrenar TODOS los pesos de un modelo es carísimo. Lo estándar es **PEFT** (Parameter-Efficient Fine-Tuning), y su técnica reina es **LoRA**.

2. **LoRA (Low-Rank Adaptation):**
   - Congela los pesos originales W y aprende dos matrices pequeñas de bajo rango (A y B) tal que el ajuste es ΔW = A·B. Se entrena <1% de los parámetros.
   - Resultado: entrenamiento barato, "adapters" de pocos MB que se montan sobre el modelo base (puedes tener varios adapters por caso de uso).
   - **QLoRA:** lo mismo pero con el modelo base cuantizado a 4 bits → fine-tuning de modelos grandes en una sola GPU. Azure OpenAI usa LoRA por debajo en su fine-tuning administrado.

3. **El dataset es el 90% del éxito:**
   - Formato Azure OpenAI: JSONL de conversaciones:
     ```json
     {"messages": [{"role": "system", "content": "Eres el asistente legal de Contoso..."},
                    {"role": "user", "content": "¿Puedo rescindir este contrato?"},
                    {"role": "assistant", "content": "Según la cláusula 4.2... [estilo y formato EXACTOS que quieres]"}]}
     ```
   - Guías: 50-100 ejemplos ya mueven la aguja para estilo/formato; calidad > cantidad (un ejemplo malo enseña errores); cubre casos difíciles y negativos ("cuando pregunten X, rechaza así"); separa 10-20% para validación.
   - El system prompt del entrenamiento debe ser EL MISMO que usarás en producción.

4. **Variantes que debes ubicar:**
   - **SFT (Supervised Fine-Tuning):** el clásico con ejemplos etiquetados; lo anterior.
   - **DPO (Direct Preference Optimization):** entrenas con pares "respuesta preferida vs rechazada"; alinea estilo/criterio sin reward model. Disponible en Azure OpenAI.
   - **Destilación:** el modelo grande (maestro) genera respuestas de calidad; con ellas afinas un modelo pequeño (alumno) que sale 10× más barato de operar. Patrón estrella para bajar costos (los *stored completions* de Azure facilitan capturar los datos).

5. **Proceso en Azure (administrado):**
   1. Sube JSONL de entrenamiento/validación al proyecto de Foundry.
   2. Crea el job (modelo base, épocas, learning rate multiplier).
   3. Revisa curvas de pérdida (train vs validation: si validation sube, overfitting).
   4. Despliega el modelo afinado como deployment propio.
   5. **Evalúa contra el baseline** con tu dataset de evaluación: el fine-tuned debe ganar en la métrica objetivo sin degradar el resto.

6. **Errores comunes:** usar fine-tuning para inyectar conocimiento factual (usa RAG), datasets con formato inconsistente, no fijar el mismo system prompt, y olvidar el costo de *hosting* del modelo afinado (además del entrenamiento).

### Fuentes para profundizar
- [Fine-tuning en Azure OpenAI](https://learn.microsoft.com/es-es/azure/ai-foundry/openai/how-to/fine-tuning)
- Paper LoRA: Hu et al., "LoRA: Low-Rank Adaptation of Large Language Models" (2021)
- Paper QLoRA: Dettmers et al. (2023) · Paper DPO: Rafailov et al. (2023)

### Beneficios
Entender LoRA y la preparación de datos te permite estimar de verdad el esfuerzo de un fine-tuning (casi todo es curar ejemplos) y elegir la variante correcta: SFT para formato, DPO para preferencias, destilación para costos.
