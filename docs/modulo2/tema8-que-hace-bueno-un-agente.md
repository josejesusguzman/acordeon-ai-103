# Qué hace bueno a un agente de IA (calidad de retrieval y de respuesta)

### Descripción
Puedes tener el contenido perfectamente indexado y aun así un agente malo. La diferencia está en **cómo el agente usa el conocimiento**: cuándo busca, cómo cita y qué hace cuando no sabe. Esto se controla con instrucciones y se verifica con pruebas sistemáticas.

> [!NOTE]
> Unidad de referencia: [Configure retrieval with Foundry IQ](https://learn.microsoft.com/en-us/training/modules/introduction-foundry-iq/5-configure-retrieval?pivots=text)

### El problema de comportamiento de retrieval
Ante "¿cuál es nuestra política de vacaciones?" hay tres comportamientos posibles y solo uno aceptable:

| Comportamiento | Ejemplo | Problema |
|---|---|---|
| Responde desde su entrenamiento | "La mayoría de empresas dan 2-3 semanas…" | Información genérica, no TU política |
| Busca pero no cita | "Tienes 15 días de PTO" | Correcto pero no verificable |
| **Busca, cita y se fundamenta** | "Tienes 15 días de PTO 【doc:Manual del Empleado 2024】" | ✔ Lo que quieres |

### Instrucciones efectivas de retrieval (las 3 reglas)
Una instrucción vaga ("usa la base de conocimiento") produce comportamiento inconsistente. Las instrucciones efectivas especifican:

1. **Cuándo buscar:** SIEMPRE consultar la knowledge base, nunca responder desde datos de entrenamiento.
2. **Cómo citar:** formato exacto de atribución de fuentes.
3. **Qué hacer si no sabe:** comportamiento de fallback definido.

```python
retrieval_instructions = """Eres un asistente de RRHH.

REGLAS CRÍTICAS:
- SIEMPRE busca en la knowledge base antes de responder.
- NUNCA respondas desde tu conocimiento de entrenamiento.
- Toda respuesta debe incluir citas: 【doc_id:search_id†nombre_fuente】
- Si la knowledge base no contiene la respuesta, responde:
  "No tengo esa información en la documentación actual. Contacta a RRHH."
"""
```

### Las 4 características de una buena respuesta (evaluating response quality)

1. **Grounding:** la información viene de la knowledge base, no del entrenamiento.
2. **Citation:** cada afirmación factual referencia su fuente.
3. **Relevance:** lo recuperado responde de verdad la pregunta.
4. **Completeness:** la respuesta está completa, no en fragmentos.

### Qué probar (tipos de query)

| Tipo de pregunta | Ejemplo | Comportamiento esperado |
|---|---|---|
| Factual directa | "¿Cuál es la política de vacaciones?" | Retrieval directo con citas |
| Requiere síntesis | "¿Diferencias entre tipos de permisos?" | Varias fuentes, respuesta sintetizada con múltiples citas |
| Fuera de la base | "¿Qué clima hace hoy?" | Fallback elegante ("no tengo esa información…") |
| Ambigua | "¿Y los beneficios?" | Pregunta aclaratoria o búsqueda enfocada |

### Instrucciones según el tipo de agente
- **Soporte a clientes:** máxima exactitud; si la documentación no cubre algo → "te conecto con un especialista", jamás adivinar.
- **Asistente interno de investigación:** puede sintetizar entre documentos, indicando nivel de confianza y citando todo.
- **Experto de dominio (compliance):** solo su dominio, citar documento y sección, derivar interpretaciones legales a humanos, indicar fecha de vigencia.

### En producción, monitorea
- **Frecuencia de citación** (¿sigue citando?)
- **Frecuencia de fallback** (¿cuánto dice "no sé"? — mucho = faltan contenidos; nada = quizá alucina)
- **Tipos de query reales** (los usuarios preguntan distinto que tus pruebas)
- **Exactitud de retrieval** (¿los documentos recuperados contienen la respuesta?)

### Beneficios
Un agente "bueno" es un agente **predecible**: siempre busca, siempre cita, y sabe decir "no sé". Eso se logra con instrucciones explícitas + pruebas por tipo de query + monitoreo, no con un mejor modelo.
