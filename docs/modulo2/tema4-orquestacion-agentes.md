# Tipos de orquestación de agentes y cuándo usar cada una

### Descripción
Cuando un solo agente no basta, necesitas **orquestación multi-agente**: coordinar varios agentes especializados. Microsoft Agent Framework (sucesor de Semantic Kernel + AutoGen) y los workflows de Foundry ofrecen patrones concretos; el examen espera que sepas cuál aplicar a cada escenario.

> [!NOTE]
> Módulos de referencia: [Orquestación de soluciones multiagente](https://learn.microsoft.com/es-es/training/modules/orchestrate-sk-multi-agent-solution/) y [Workflows en Microsoft Foundry](https://learn.microsoft.com/es-es/training/modules/build-agent-workflows-microsoft-foundry/)

### Patrones de orquestación

1. **Secuencial (pipeline):**
   - Los agentes se ejecutan en cadena: la salida de uno es la entrada del siguiente.
   - Ejemplo: agente-resumidor → agente-traductor → agente-redactor de correo.
   - Úsalo cuando el proceso tiene etapas claras y ordenadas (refinamiento paso a paso).

2. **Concurrente (paralelo / fan-out, fan-in):**
   - Varios agentes trabajan sobre la misma entrada a la vez y luego se agregan resultados.
   - Ejemplo: analizar un contrato simultáneamente desde perspectiva legal, financiera y de riesgo.
   - Úsalo cuando las subtareas son independientes y quieres velocidad o múltiples perspectivas.

3. **Group chat (conversación grupal):**
   - Los agentes conversan en un hilo compartido, con un *chat manager* que decide quién habla; puede incluir humanos.
   - Ejemplo: agente-escritor y agente-crítico debatiendo hasta aprobar un texto (writer/reviewer).
   - Úsalo para colaboración iterativa, brainstorming o validación cruzada. Es el más flexible y el más caro/impredecible.

4. **Handoff (delegación dinámica):**
   - Un agente evalúa la solicitud y **transfiere el control** al especialista adecuado, que puede volver a transferir.
   - Ejemplo: triage de soporte → deriva a agente-facturación, agente-técnico o humano.
   - Úsalo cuando no sabes de antemano qué habilidad se necesita (routing dinámico).

5. **Magentic (planner dinámico):**
   - Un *manager* construye y ajusta un plan dinámicamente, delegando a agentes especializados y llevando un task ledger (inspirado en Magentic-One).
   - Ejemplo: "investiga, calcula y redacta un informe de mercado" sin flujo predefinido.
   - Úsalo para problemas abiertos y complejos donde el plan emerge sobre la marcha. Máxima autonomía, máxima necesidad de supervisión.

### ¿Workflow o LLM decide? (regla clave)

| Si el flujo... | Usa |
|---|---|
| Es conocido, repetible, auditable | **Workflow determinista** (sequential/concurrent como grafo) |
| Depende del contenido de cada solicitud | **Handoff / group chat** (el LLM enruta) |
| Es abierto y multi-paso sin receta | **Magentic** |

### Esbozo con Microsoft Agent Framework (Python)

```python
from agent_framework import SequentialBuilder, ConcurrentBuilder

# Pipeline secuencial: physicist -> chemist
workflow = SequentialBuilder().participants([physicist, chemist]).build()

# Paralelo con agregación
workflow = ConcurrentBuilder().participants([legal, finanzas, riesgo]).build()
```

### Criterios para decidir (resumen para examen)
- **¿Orden fijo?** → Secuencial.
- **¿Independencia + velocidad?** → Concurrente.
- **¿Iteración/crítica entre agentes?** → Group chat.
- **¿Enrutar al especialista correcto?** → Handoff.
- **¿Plan dinámico para tarea abierta?** → Magentic.
- **¿Un solo agente con 2-3 tools basta?** → No orquestes: la orquestación agrega costo, latencia y puntos de falla.

### Beneficios
Elegir el patrón correcto mantiene el sistema predecible y depurable: la regla de oro es usar el patrón MENOS autónomo que resuelva el problema.
