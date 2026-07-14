# Tema Complementario 9: Structured outputs y function calling robusto

### Descripción
El 80% de los bugs de integración con LLMs son "el modelo devolvió texto que mi código no pudo parsear". **Structured outputs** (salidas con JSON Schema garantizado) y un function calling bien diseñado eliminan esa clase entera de errores — es la técnica que convierte al LLM en un componente confiable de software.

### Aspectos Clave

1. **La evolución:** pedir JSON "por favor" en el prompt (frágil) → `response_format={"type": "json_object"}` (JSON válido pero esquema no garantizado) → **structured outputs** (`json_schema` con `strict: true`): el motor restringe la generación token a token para que SIEMPRE cumpla el esquema.

2. **Structured outputs con Pydantic (el patrón en Python):**
   ```python
   from pydantic import BaseModel
   from typing import Literal

   class Ticket(BaseModel):
       categoria: Literal["red", "hardware", "software", "accesos"]
       prioridad: Literal["baja", "media", "alta"]
       resumen: str
       requiere_humano: bool

   completion = openai_client.beta.chat.completions.parse(   # .parse, no .create
       model="gpt-4o",
       messages=[{"role": "system", "content": "Clasifica tickets de TI."},
                 {"role": "user", "content": "Se cayó la VPN de todo el edificio"}],
       response_format=Ticket,
   )
   ticket = completion.choices[0].message.parsed   # objeto Ticket tipado
   print(ticket.categoria, ticket.prioridad)        # red alta
   ```
   Sin regex, sin `json.loads` con try/except, sin campos faltantes.

3. **Reglas de diseño de esquemas:**
   - **Enums (`Literal`) para todo lo cerrado:** el modelo no puede inventar categorías.
   - **Descripciones en los campos** (`Field(description=...)`): son prompt para el modelo.
   - Esquemas planos y pequeños > anidamientos profundos (mejor calidad).
   - Un campo `razonamiento: str` ANTES de la decisión mejora la exactitud (chain of thought embebido).
   - Para "no aplica", incluye explícitamente la opción (`Optional` o enum "desconocido"), o el modelo forzará un valor.

4. **Function calling robusto (las tools también son esquemas):**
   - Mismo principio: parámetros con tipos estrictos, enums y descripciones ("cuándo usar esta tool" incluido).
   - `strict: true` en la definición de la función garantiza argumentos válidos contra el esquema.
   - Valida IGUAL dentro de la tool (rangos, existencia de IDs): el esquema garantiza forma, no semántica.
   - Devuelve errores accionables: `{"error": "pedido no existe, formato esperado PED-XXXX"}` permite al modelo autocorregirse.

5. **Dónde brilla esto:**
   - **Extracción de datos** de texto libre a BD (facturas, correos → filas tipadas).
   - **Routing/clasificación** en workflows (el If/Else evalúa campos garantizados — así funcionan los structured outputs en los workflows de Foundry).
   - **Agentes multi-paso:** cada paso produce estado tipado que el siguiente consume.

6. **Limitaciones:** subconjunto de JSON Schema soportado (sin `patternProperties`, etc.); el primer request con un esquema nuevo tiene latencia extra (compila la gramática); y structured output ≠ respuesta correcta: el formato está garantizado, el contenido se evalúa igual.

### Fuentes para profundizar
- [Structured outputs en Azure OpenAI](https://learn.microsoft.com/es-es/azure/ai-foundry/openai/how-to/structured-outputs)
- [Function calling](https://learn.microsoft.com/es-es/azure/ai-foundry/openai/how-to/function-calling)
- [Documentación de Pydantic](https://docs.pydantic.dev/)

### Beneficios
Structured outputs convierte al LLM de "generador de texto que a veces parsea" en una función tipada: `texto → objeto validado`. Es el hábito #1 que distingue código LLM de producción del de prototipo.
