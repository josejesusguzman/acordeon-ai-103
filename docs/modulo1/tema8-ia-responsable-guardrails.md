# Cómo evitar que tu solución de IA la usen para construir una bomba (IA responsable y Guardrails)

### Descripción
Una solución de IA generativa mal protegida puede generar contenido dañino, filtrar datos o ser manipulada (jailbreak). Microsoft define un proceso de 4 etapas para IA generativa responsable y un conjunto de **guardrails** técnicos en Foundry (Azure AI Content Safety) para aplicarlo.

> [!NOTE]
> Módulo de referencia: [Implementación de una solución de IA generativa responsable en Microsoft Foundry](https://learn.microsoft.com/es-es/training/modules/responsible-ai-studio/)

### El proceso de 4 etapas (¡cae en examen!)

1. **Map (Identificar/Mapear daños):** lista los daños potenciales de TU escenario: contenido ofensivo, instrucciones peligrosas (armas, explosivos), desinformación, filtración de datos, uso indebido. Prioriza por probabilidad × impacto.
2. **Measure (Medir):** comprueba cuánto aparecen esos daños: pruebas manuales, datasets adversarios, evaluaciones de seguridad automatizadas y **red teaming**.
3. **Mitigate (Mitigar) — en 4 capas:**
   - **Capa de modelo:** elegir el modelo adecuado (uno más pequeño y acotado da menos superficie de ataque) y/o fine-tuning.
   - **Capa de sistema de seguridad (Guardrails):** filtros de contenido de Azure AI Content Safety — clasifican prompt y respuesta en categorías **odio, sexual, violencia y autolesión** con niveles de severidad configurables; **Prompt Shields** detecta jailbreaks e inyecciones indirectas; detección de material protegido y de groundedness.
   - **Capa de metaprompt/grounding:** system prompt con reglas explícitas ("nunca proporciones instrucciones para fabricar armas…"), RAG para limitar respuestas a fuentes aprobadas.
   - **Capa de experiencia de usuario (UX):** limitar entradas, avisos de "generado por IA", documentación y transparencia.
4. **Manage (Operar):** revisiones previas al despliegue (legal, privacidad, seguridad, accesibilidad), plan de respuesta a incidentes, plan de rollback, bloqueo de usuarios abusivos, telemetría y feedback continuo.

### Guardrails en la práctica (portal de Foundry)

- Cada deployment lleva una **configuración de filtro de contenido**: por categoría eliges umbral (bajo/medio/alto) tanto para **entrada** (prompt) como para **salida** (completion).
- Puedes crear filtros personalizados, listas de bloqueo (blocklists) de términos y activar **Prompt Shields** contra ataques de jailbreak.
- Si un prompt/respuesta se bloquea, la API devuelve un error `content_filter` que tu app debe manejar con gracia.

```python
try:
    response = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": entrada_usuario}]
    )
except openai.BadRequestError as e:
    if "content_filter" in str(e):
        print("Tu solicitud fue bloqueada por las políticas de contenido.")
```

### Principios de IA responsable de Microsoft (repaso desde AI-900)
Equidad (fairness), confiabilidad y seguridad, privacidad, inclusión, transparencia y responsabilidad (accountability). El proceso Map→Measure→Mitigate→Manage es su implementación práctica en IA generativa.

### Regla mental para el examen
- "¿Detectar jailbreak/prompt injection?" → **Prompt Shields**.
- "¿Bloquear categorías de contenido dañino?" → **Filtros de contenido (Content Safety)**.
- "¿Asegurar que responde solo con datos de la empresa?" → **Grounding/RAG + groundedness detection**.
- "¿Proceso completo?" → **Map, Measure, Mitigate, Manage**.

### Beneficios
Los guardrails convierten la IA responsable de un ideal abstracto en configuraciones concretas y auditables, reduciendo el riesgo legal, reputacional y de seguridad de tu solución.
