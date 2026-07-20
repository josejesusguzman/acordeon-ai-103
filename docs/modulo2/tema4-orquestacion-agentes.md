# Tipos de orquestación de agentes y cuándo usar cada una

### Descripción
Cuando un solo agente no basta, necesitas **orquestación multi-agente**: coordinar varios agentes especializados. Microsoft Agent Framework (sucesor de Semantic Kernel + AutoGen) y los workflows de Foundry ofrecen cinco patrones concretos — secuencial, concurrente, group chat, handoff y Magentic — y el examen espera que sepas **cuál aplicar a cada escenario y por qué**. Este tema desarrolla cada patrón con sus señales de uso, riesgos y un caso de estudio.

> [!NOTE]
> Módulos de referencia: [Orquestación de soluciones multiagente](https://learn.microsoft.com/es-es/training/modules/orchestrate-sk-multi-agent-solution/) y [Workflows en Microsoft Foundry](https://learn.microsoft.com/es-es/training/modules/build-agent-workflows-microsoft-foundry/)

### Antes de orquestar: ¿de verdad necesitas multi-agente?

La orquestación agrega costo (cada agente = más llamadas al LLM), latencia, puntos de falla y complejidad de depuración. **Señales de que sí la necesitas:**

- Las instrucciones de tu agente único ya son kilométricas y se contradicen entre secciones.
- El agente tiene tantas tools (10+) que elige mal cuál usar.
- La tarea mezcla habilidades en conflicto (creatividad vs. rigor; redactar vs. auditar).
- El contexto se desborda: un solo hilo no puede cargar todo el conocimiento necesario.
- Distintos equipos son dueños de distintas capacidades y necesitan evolucionarlas por separado.

Si nada de esto ocurre, **un agente con 2-3 tools bien definidas gana**: más barato, más rápido, más predecible.

### Los 5 patrones a fondo

#### 1. Secuencial (pipeline)

**Cómo funciona:** los agentes se ejecutan en cadena; la salida de uno es la entrada del siguiente.

**Cuándo usarlo:** el proceso tiene etapas claras, ordenadas y siempre las mismas; cada etapa **refina** el trabajo de la anterior; necesitas auditar qué produjo cada paso.

**Por qué:** máxima predictibilidad y depuración trivial — si el resultado sale mal, inspeccionas la salida intermedia de cada etapa y encuentras al culpable. Cada agente tiene instrucciones cortas y enfocadas.

**Riesgos:** un error temprano se propaga a todo el pipeline (valida salidas intermedias); la latencia es la suma de todas las etapas.

**Caso de estudio — "Editorial Andina":** una editorial publica artículos técnicos en 3 idiomas. Problema: la localización manual tarda días y las traducciones pierden precisión técnica. Solución: pipeline de 4 agentes — **Resumidor** (condensa el artículo a los puntos clave aprobados) → **Traductor** (traduce manteniendo terminología del glosario) → **Adaptador cultural** (ajusta ejemplos y referencias al mercado local) → **QA editorial** (verifica formato, citas y devuelve un reporte de confianza). Cada salida intermedia se guarda: cuando una traducción sale mal, el equipo ve exactamente en qué etapa se rompió. Resultado: localización el mismo día, con puntos de control auditables.

#### 2. Concurrente (paralelo / fan-out, fan-in)

**Cómo funciona:** varios agentes procesan la **misma entrada** a la vez (fan-out) y un paso final agrega sus resultados (fan-in).

**Cuándo usarlo:** las subtareas son **independientes entre sí** (ninguna necesita la salida de otra); quieres velocidad o múltiples perspectivas simultáneas del mismo insumo.

**Por qué:** la latencia total ≈ la del agente más lento (no la suma); cada especialista analiza sin ser contaminado por las conclusiones de los demás — perspectivas verdaderamente independientes.

**Riesgos:** el costo se multiplica por el número de agentes en cada ejecución; el paso de **agregación** es crítico (¿qué pasa si legal dice "sí" y riesgo dice "no"? define reglas de conflicto).

**Caso de estudio — "LexCorp":** un despacho revisa contratos de proveedores. Problema: la revisión secuencial entre departamentos tardaba 2 semanas; cada área esperaba a la anterior. Solución: fan-out del contrato a 4 agentes simultáneos — **Legal** (cláusulas anómalas vs. plantillas aprobadas), **Financiero** (términos de pago y penalizaciones), **Riesgo** (exposición y límites de responsabilidad) y **Compliance** (regulación aplicable) — y un **Agregador** que consolida un dictamen con semáforo por área. Regla de conflicto: cualquier rojo bloquea y escala a humano. Resultado: dictamen preliminar en minutos; los abogados solo revisan los rojos y amarillos.

#### 3. Group chat (conversación grupal)

**Cómo funciona:** los agentes comparten un hilo de conversación; un **chat manager** decide quién habla en cada turno y cuándo terminar; puede incluir humanos como participantes.

**Cuándo usarlo:** el valor sale de la **iteración y la crítica cruzada** (borrador → crítica → mejora → aprobación); brainstorming; validación entre pares con criterios distintos.

**Por qué:** es el único patrón donde los agentes se responden entre sí y refinan colectivamente; el resultado suele superar lo que cualquier agente lograría solo.

**Riesgos:** el más caro e impredecible — sin límites, dos agentes pueden debatir para siempre. Configura **máximo de turnos**, criterio de terminación explícito y un manager con instrucciones firmes.

**Caso de estudio — "Agencia Nébula":** una agencia produce copys de campaña. Problema: los borradores de IA eran genéricos y el ida y vuelta con revisores humanos tomaba días. Solución: group chat con **Redactor** (genera propuestas), **Crítico** (ataca debilidades: claims sin sustento, clichés) y **Guardián de marca** (verifica voz y lineamientos, tema brand-review), moderados por un manager que corta en máximo 6 turnos o cuando el Guardián aprueba. El director creativo entra al hilo solo para la aprobación final. Resultado: copys que llegan al humano ya depurados por dos rondas de crítica interna.

#### 4. Handoff (delegación dinámica)

**Cómo funciona:** un agente evalúa la solicitud y **transfiere el control completo** al especialista adecuado, que puede a su vez volver a transferir.

**Cuándo usarlo:** **no sabes de antemano qué habilidad se necesita**; cada especialista tiene sus propias tools y permisos; el clásico triage/routing.

**Por qué:** cada solicitud la atiende el agente con las instrucciones y herramientas exactas para ese dominio — sin diluir a un generalista; los permisos se segmentan por especialista (el de facturación no toca el sistema técnico).

**Riesgos:** ping-pong infinito entre agentes (limita el número de transferencias); pérdida de contexto al transferir (comparte el hilo o un resumen estructurado); clasificaciones erróneas del triage (mide su precisión como métrica propia).

**Caso de estudio — "Banco Meridiano":** el banco recibe 50,000 chats/día mezclando aclaraciones, fallas de app y fraudes. Problema: un bot generalista respondía mediocre en todo y el fraude exige trato inmediato y regulado. Solución: agente de **Triage** (clasifica y transfiere; nunca responde de fondo) → **Facturación** (tools de consulta de movimientos), **Soporte técnico** (diagnóstico de la app) o, si detecta indicios de fraude, **handoff directo a humano** con prioridad — sin intentar resolverlo la IA. Límite de 2 transferencias; después, humano. Resultado: cada dominio con su especialista y el caso crítico (fraude) jamás se queda atorado en la IA.

#### 5. Magentic (planner dinámico)

**Cómo funciona:** un **manager** construye un plan, lo delega a agentes especializados, lleva un **task ledger** (registro de tareas y hallazgos) y **re-planifica sobre la marcha** según los resultados (inspirado en Magentic-One).

**Cuándo usarlo:** problemas **abiertos y multi-paso sin receta** — el plan no se conoce de antemano y emerge durante la ejecución; investigación, análisis exploratorio, informes complejos.

**Por qué:** es el único patrón que se adapta cuando el camino cambia a mitad de tarea ("los datos de X no existen; buscaré Y"); máxima autonomía.

**Riesgos:** máxima imprevisibilidad de costo y tiempo — ponle **presupuesto** (máximo de rondas/tokens), checkpoints con aprobación humana y observabilidad completa (tracing, tema 1 del módulo). Es el patrón que más necesita el pipeline de evaluación.

**Caso de estudio — "Consultora Vector":** un cliente pide "análisis del mercado de baterías en Latinoamérica con proyección a 5 años" — cada encargo es distinto y no hay flujo fijo. Solución: manager Magentic que planifica y delega a **Investigador** (busca fuentes), **Analista de datos** (procesa cifras y proyecta) y **Redactor** (arma el informe), ajustando el plan cuando una fuente no existe o los datos contradicen la hipótesis. Checkpoint obligatorio: el consultor humano aprueba el plan inicial y el borrador antes de entrega; presupuesto de 20 rondas. Resultado: informes en horas en lugar de semanas, con el ledger como bitácora auditable de cómo se llegó a cada conclusión.

### ¿Workflow o LLM decide? (regla clave)

| Si el flujo... | Usa | Quién controla |
|---|---|---|
| Es conocido, repetible, auditable | **Workflow determinista** (sequential/concurrent como grafo) | Tu código |
| Depende del contenido de cada solicitud | **Handoff / group chat** | El LLM enruta |
| Es abierto y multi-paso sin receta | **Magentic** | El LLM planifica |

### Tabla comparativa

| Patrón | Predictibilidad | Costo relativo | Latencia | Cuándo NO usarlo |
|---|---|---|---|---|
| Secuencial | Máxima | Bajo | Suma de etapas | Si las etapas no dependen entre sí (usa concurrente) |
| Concurrente | Alta | Medio (× agentes) | La del más lento | Si una tarea necesita la salida de otra |
| Group chat | Baja | Alto | Variable | Si no hay iteración real que aporte (puro overhead) |
| Handoff | Media | Bajo-medio | 1 especialista | Si siempre se necesita el mismo especialista (hazlo fijo) |
| Magentic | Mínima | Alto e imprevisible | Variable | Si el proceso es conocido (cualquier otro patrón es mejor) |

### Esbozo con Microsoft Agent Framework (Python)

```python
from agent_framework import SequentialBuilder, ConcurrentBuilder, MagenticBuilder

# 1. Pipeline secuencial: resumidor -> traductor -> qa
workflow = SequentialBuilder().participants([resumidor, traductor, qa]).build()

# 2. Paralelo con agregación
workflow = ConcurrentBuilder().participants([legal, finanzas, riesgo]).build()

# 5. Magentic: manager que planifica y delega
workflow = (MagenticBuilder()
    .participants(investigador=investigador, analista=analista, redactor=redactor)
    .build())
```

(Group chat y handoff se configuran con sus builders equivalentes definiendo el manager/las reglas de transferencia; en Foundry, los **workflows visuales** permiten armar los grafos deterministas sin código.)

### Criterios para decidir (resumen para examen)

- **¿Orden fijo?** → Secuencial.
- **¿Independencia + velocidad?** → Concurrente.
- **¿Iteración/crítica entre agentes?** → Group chat.
- **¿Enrutar al especialista correcto?** → Handoff.
- **¿Plan dinámico para tarea abierta?** → Magentic.
- **¿Un solo agente con 2-3 tools basta?** → No orquestes: la orquestación agrega costo, latencia y puntos de falla.

### Tutorial: construir cada tipo de orquestación en Microsoft Foundry

Basado en el módulo [Creación de flujos de trabajo controlados por agentes](https://learn.microsoft.com/es-mx/training/modules/build-agent-workflows-microsoft-foundry/). El portal ofrece un **diseñador visual de workflows** con patrones predefinidos (secuencial, human-in-the-loop, group chat); los patrones concurrente y Magentic se construyen en código con Agent Framework.

#### Paso 0: Abre el diseñador y conoce los nodos

1. En tu proyecto de https://ai.azure.com, ve a la sección de **Workflows** y crea uno nuevo: desde **lienzo en blanco** o eligiendo un **patrón predefinido**.
2. El workflow es una secuencia de **nodos conectados**; al seleccionar **+** ves los tipos disponibles:
   - **Invoke agent:** invoca un agente del proyecto (o crea uno nuevo ahí mismo).
   - **Flow:** controla la ejecución — **If/Else** (bifurca por condición), **Go to** (salta a otro nodo), **For Each** (itera una lista).
   - **Data transformation:** **Set/Reset variable** y **Parse value** (extraer datos de salidas estructuradas).
   - **Basic chat:** envía mensajes o hace preguntas al usuario.
   - **End:** concluye el workflow (con resultado opcional).

   ![Diseñador de workflows con los tipos de nodo disponibles](https://learn.microsoft.com/es-mx/training/wwl-data-ai/build-agent-workflows-microsoft-foundry/media/workflow-add-node.png)

> [!IMPORTANT]
> Los workflows **no se guardan automáticamente**: guarda tus cambios con frecuencia. Se ejecutan en contexto conversacional, así que puedes probarlos desde la ventana de chat viendo cómo la entrada recorre nodo por nodo.

#### Orquestación secuencial (pipeline)

1. Crea el workflow desde el **patrón Sequential** (o encadena nodos a mano).
2. Agrega un nodo **Invoke agent** por etapa (p. ej. resumidor → traductor → QA), en orden: la salida de cada nodo pasa al siguiente.
3. En la **configuración de acción** de cada nodo, guarda la salida en una **variable** para que las etapas posteriores (o un If/Else de validación) puedan usarla.
4. Cierra con un nodo **End** que devuelva el resultado final. Prueba en el chat y verifica la salida intermedia de cada etapa.

![Configuración para almacenar la salida del agente en una variable](https://learn.microsoft.com/es-mx/training/wwl-data-ai/build-agent-workflows-microsoft-foundry/media/agent-action-settings.png)

#### Orquestación handoff (enrutamiento con salida estructurada)

El diseñador implementa el routing de forma **determinista y auditable**:

1. Primer nodo **Invoke agent**: un agente **clasificador** cuyo formato de respuesta defines como **salida estructurada (esquema JSON)** en los parámetros de la pestaña **Details** — p. ej. `{"categoria": "facturacion" | "tecnico" | "fraude"}`. Así su salida es predecible para el flujo de control.

   ![Parámetros del nodo Invoke agent con el esquema de respuesta](https://learn.microsoft.com/es-mx/training/wwl-data-ai/build-agent-workflows-microsoft-foundry/media/agent-parameters.png)

2. Guarda esa salida en una variable y (si hace falta) extráela con **Parse value**.
3. Agrega un nodo **If/Else** que evalúe la categoría y enrute a la rama del especialista correspondiente (cada rama con su propio **Invoke agent** y sus herramientas); usa **Go to** para saltar entre secciones del flujo.
4. Los agentes son **reutilizables** entre workflows: el mismo clasificador puede servir a varios flujos con distintos resolutores.

#### Orquestación group chat

1. Crea el workflow desde el **patrón Group chat**: varios agentes comparten la conversación y el control se mueve entre ellos según contexto, reglas o resultados intermedios (p. ej. redactor + crítico + revisor de marca).
2. Configura cada agente participante (instrucciones, herramientas, guardrails) desde su editor **Invoke agent**.
3. Define el criterio de terminación para no debatir infinito, y prueba en el chat viendo cómo se pasan el turno.

![Workflow de group chat en el diseñador de Microsoft Foundry](https://learn.microsoft.com/es-mx/training/wwl-data-ai/build-agent-workflows-microsoft-foundry/media/group-chat-workflow.png)

#### Human-in-the-loop (complemento de cualquier patrón)

1. Inserta un nodo de **Basic chat** que formule la pregunta al usuario (p. ej. "¿Apruebas este dictamen?") y guarde la respuesta en una variable.
2. Sigue con un **If/Else**: si aprueba, continúa; si no, re-trabaja o escala. El workflow **pausa la ejecución** hasta recibir la entrada — el patrón oficial para aprobaciones y casos de baja confianza.

#### Concurrente y Magentic (en código, con Agent Framework)

Estos dos patrones no tienen plantilla en el diseñador visual; se construyen con el SDK (ver el esbozo de código de la sección anterior): `ConcurrentBuilder` para fan-out/fan-in y `MagenticBuilder` para el planner dinámico. Los agentes que ya creaste en el proyecto Foundry se reutilizan como participantes, y conservan la observabilidad del proyecto (tracing y evaluaciones del tema 1 de este módulo).

#### Cierre: probar y mantener

Prueba cada rama desde la ventana de chat antes de sumar complejidad; usa expresiones **Power Fx** para manipular datos y condiciones avanzadas; y versiona el diseño guardando cada iteración (los workflows también pueden invocarse desde código para integrarlos a tu app).

### Beneficios
Elegir el patrón correcto mantiene el sistema predecible y depurable: la regla de oro es usar el patrón **MENOS autónomo** que resuelva el problema — y subir de nivel (secuencial → concurrente → handoff → group chat → Magentic) solo cuando el problema lo exija.
