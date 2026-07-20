# Pipeline hasta producción de un agente de IA con Azure

### Descripción
Llevar un agente del playground a producción es un pipeline con etapas bien definidas: desarrollo → conocimiento → evaluación (gate) → CI/CD → despliegue → seguridad → observabilidad. Microsoft lo formaliza como el ciclo de vida de observabilidad de IA: **selección del modelo → evaluación previa a producción → supervisión posterior a producción**. Aquí está el mapa completo, un tutorial paso a paso y tips prácticos de cada etapa.

> [!NOTE]
> Referencias: [Desarrollo de agentes de IA en Azure](https://learn.microsoft.com/es-es/training/paths/develop-ai-agents-azure/), [Observabilidad en IA generativa](https://learn.microsoft.com/es-mx/azure/ai-foundry/concepts/observability) y [Deploy and govern agentic AI solutions](https://learn.microsoft.com/es-es/training/paths/deploy-govern-agentic-ai-solutions-azure/)

### Vista general

![Ciclo de vida de las aplicaciones de IA: selección de modelo, creación y operación](https://learn.microsoft.com/es-mx/azure/foundry/media/evaluations/lifecycle.png)

```text
Código+Prompt (Git) ──> CI: tests ──> Evaluaciones (calidad+seguridad)
      │                                   │ pasa umbral
      ▼                                   ▼
  IaC (Bicep) ────────────────> Deploy (Foundry Agent / Container Apps)
                                          │
                                          ▼
                     Observabilidad (tracing, métricas, feedback)
                                          │
                                          └────> vuelta al desarrollo
```

### Tutorial paso a paso

#### Paso 1: Prepara proyecto y entorno

1. Crea un **proyecto de Foundry** para desarrollo (habrá otro para staging y otro para prod).
2. Implementa el modelo elegido (proceso de selección: módulo 1, temas 4-5).

   ![Interfaz de implementación de modelo en el portal de Foundry](https://learn.microsoft.com/es-mx/training/wwl-data-ai/model-catalog-evaluate/media/deploy-model.png)
3. Repositorio Git con la estructura: código del agente, prompts/instrucciones, dataset de evaluación, IaC.
4. Autenticación con `DefaultAzureCredential` desde el día uno (nada de keys en `.env`).

> [!TIP]
> Usa **el mismo nombre de deployment** en dev/staging/prod (p. ej. `chat-model`): el código no cambia entre entornos, solo el endpoint del proyecto. Y versiona TODO: **el prompt es código** — cada cambio de instrucciones debe pasar por PR igual que un cambio de lógica.

#### Paso 2: Define el agente

```python
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

project = AIProjectClient(endpoint=PROJECT_ENDPOINT, credential=DefaultAzureCredential())
agent = project.agents.create_agent(
    model="chat-model",
    name="agente-gastos",
    instructions=INSTRUCCIONES,     # cargadas desde archivo versionado
    tools=tools,                    # funciones, OpenAPI, MCP...
)
```

![Agent playground con el selector de métricas de evaluación](https://learn.microsoft.com/es-mx/azure/foundry/media/observability/agent-playground-evaluation-metrics.png)

> [!TIP]
> Escribe las instrucciones con el checklist del tema 6 (rol → límites → formato → política de incertidumbre) y define **cuándo usar cada tool y qué está prohibido** (ejemplo del agente de gastos, tema 6). Herramientas de **mínimo privilegio**: si el agente solo consulta, no le des tools de escritura. Prueba iterativamente en el **agent playground** — ten en cuenta que sus evaluaciones automáticas vienen activadas por defecto y se facturan por consumo (desactívalas en el selector de "métricas" de la esquina superior derecha, como muestra la captura, si solo estás explorando).

#### Paso 3: Conecta conocimiento y herramientas

1. **Conocimiento (RAG):** Foundry IQ o Azure AI Search con búsqueda híbrida (tema 7).
2. **Herramientas:** funciones propias, especificaciones **OpenAPI**, servidores **MCP**, **Logic Apps** para integraciones empresariales.

> [!TIP]
> Antes de indexar documentos, pásalos por **Content Safety** (contenido nocivo) y activa **Prompt Shields para ataques indirectos** — los documentos son el vector de inyección número uno en agentes RAG (tema 8). Para las tools: valida los parámetros que genera el modelo antes de ejecutar, como validarías input de usuario.

#### Paso 4: Construye el gate de evaluación

![Los seis pasos de la evaluación previa a producción](https://learn.microsoft.com/es-mx/azure/foundry/media/evaluations/evaluation-models-diagram.png)

1. **Dataset de prueba** versionado en Git: casos reales + casos límite + pruebas de seguridad (puedes arrancar con generación sintética, tema 2).
2. **Evaluadores de agente** (además de los de calidad/seguridad): **intent resolution** (¿entendió la petición?), **tool call accuracy** (¿llamó la herramienta correcta con los parámetros correctos?) y **task adherence** (¿completó la tarea sin desviarse?).
3. Ejecuta con `azure-ai-evaluation` local o **evaluación en la nube** desde el SDK del proyecto.
4. **Red teaming automatizado:** el **AI Red Teaming Agent** (basado en PyRIT) simula ataques adversarios antes del despliegue; úsalo con revisión humana.
5. Define **umbrales numéricos** por métrica: sin pasarlos, el pipeline NO avanza.

   ![Lista de ejecuciones de evaluación con sus puntuaciones en el portal](https://learn.microsoft.com/es-mx/azure/foundry/media/observability/evaluation-runs.png)

> [!TIP]
> Congela el **modelo evaluador** (juez) aunque cambies el modelo del agente: si cambias ambos, no sabes qué movió las puntuaciones. Cuando algo falle, usa el **análisis de clústeres** del portal para agrupar fallos por causa en vez de leer 200 filas. Y compara ejecuciones con la vista estadística (tema 5): exige significancia, no "se ve mejor".

#### Paso 5: Automatiza con CI/CD

```yaml
# GitHub Actions (esqueleto)
jobs:
  test:                # pruebas unitarias del código y tools (LLM mockeado)
    ...
  evaluate:            # evaluación batch contra el agente en staging
    needs: test
    steps:
      - run: python run_eval.py --dataset eval/casos.jsonl --threshold 4.0
  deploy:              # solo si evaluate pasó umbrales
    needs: evaluate
    ...
```

- **IaC** (Bicep/Terraform) para Foundry, Search, Storage, Key Vault: entornos reproducibles.
- Entornos separados dev → staging → prod con proyectos de Foundry distintos.

> [!TIP]
> Autentica el pipeline con **OIDC/identidad federada** de GitHub/DevOps hacia Azure — cero secretos en el repo. Publica los resultados de evaluación como **artefactos del pipeline** (histórico de regresiones) y haz que el job falle con mensaje claro de qué métrica no pasó. Las pruebas unitarias van *antes* de las evaluaciones: son gratis; las evaluaciones cuestan tokens.

#### Paso 6: Despliega

| Opción | Qué es | Cuándo |
|---|---|---|
| **Foundry Agent Service (hospedado)** | Azure aloja el runtime del agente; lo invocas por API/SDK | Lo más simple; la mayoría de los casos |
| **App propia** | Contenedor en Container Apps / AKS / App Service usando el SDK | Control total del runtime y la orquestación |
| **Canales** | API REST, Teams / M365 Copilot (Agent Store), web app | Según dónde viven tus usuarios |

> [!TIP]
> Revisa **cuota y tipo de deployment** (tema del módulo 1): global standard para elasticidad, PTU si el tráfico es predecible, batch para trabajos masivos. Mantén el deployment anterior vivo durante el rollout (blue-green): el rollback debe ser cambiar una variable, no re-desplegar.

#### Paso 7: Asegura y gobierna

1. **Identidad del agente:** Microsoft **Entra Agent ID**; una identidad administrada por agente, RBAC de mínimo privilegio.
2. **Secretos:** Key Vault para lo inevitable; identidades administradas para todo lo demás.
3. **Red:** Private Link / VNet si hay datos sensibles (la evaluación también soporta red virtual).
4. **Guardrails:** filtro de contenido personalizado + Prompt Shields aplicados al deployment (tutorial del tema 8), blocklists y manejo del error `content_filter` en el cliente.

   ![Página Guardrails + controls para crear el filtro de contenido](https://learn.microsoft.com/es-mx/azure/foundry-classic/media/content-safety/content-filter/create-content-filter.png)

> [!TIP]
> El guardrail se prueba: corre las evaluaciones de seguridad **contra el deployment ya protegido** y compara la tasa de defectos antes/después. Y define límites de uso por usuario/canal — un agente sin rate limit es una factura sin techo.

#### Paso 8: Observa y mejora en producción

Las tres capacidades de observabilidad de Foundry, integradas con **Application Insights**:

1. **Tracing (OpenTelemetry):** cada run, llamada LLM, invocación de tool y tokens; soporta Agent Framework, LangChain/LangGraph y OpenAI Agents SDK. Es tu herramienta para depurar razonamientos multi-paso.
2. **Monitoreo:** dashboards en tiempo real de latencia, errores, consumo de tokens y puntuaciones de calidad; **alertas de Azure Monitor** cuando una salida cae bajo umbral o genera contenido dañino. En el portal: **Build → tu agente → pestaña Monitoring** (requiere Application Insights conectado al proyecto).

   ![Panel de supervisión del agente: puntuaciones de evaluación, tasa de éxito y uso de tokens](https://learn.microsoft.com/es-mx/azure/foundry/media/observability/how-to-monitor-agents-dashboard/foundry-metrics-dashboard.png)
3. **Evaluación continua:** evalúa una **muestra del tráfico real** (tasa de muestreo configurable), más **evaluaciones programadas** con tu dataset para detectar drift y **red teaming programado**. Todo se configura desde el engrane de la pestaña Monitoring:

   ![Panel de configuración del monitoreo: evaluación continua, programada, red teaming y alertas](https://learn.microsoft.com/es-mx/azure/foundry/media/observability/how-to-monitor-agents-dashboard/monitor-settings-panel.png)

> [!TIP]
> La tasa de muestreo es tu control de costo: empieza con 5-10% del tráfico y súbela solo donde detectes problemas. Cada fallo real de producción se convierte en **caso nuevo del dataset de evaluación** — así el gate del paso 4 aprende de producción.

#### Paso 9: Cierra el ciclo

Feedback de usuarios + fallos detectados → nuevos casos de prueba → ajuste de prompt/RAG/modelo → el pipeline re-evalúa → despliegue seguro. Ten por escrito: plan de **rollback**, plan de **respuesta a incidentes** (¿quién apaga al agente?, ¿cómo bloqueas un patrón de abuso hoy mismo? → blocklist), y calendario de **actualización de versiones de modelo** (los modelos se retiran; fija versiones y planea migraciones con el pipeline de evaluación).

### Checklist de producción

- [ ] Prompt e instrucciones versionados (PR para cambios de prompt)
- [ ] Dataset de evaluación versionado, con casos de seguridad
- [ ] Evaluaciones automatizadas con umbrales que bloquean el deploy
- [ ] Evaluadores de agente: intent resolution, tool call accuracy, task adherence
- [ ] Red teaming ejecutado antes del lanzamiento
- [ ] Autenticación sin secretos (managed identity + OIDC en CI/CD)
- [ ] Guardrail personalizado aplicado y probado (tasa de defectos)
- [ ] Tracing habilitado + dashboard + alertas
- [ ] Evaluación continua sobre tráfico muestreado
- [ ] Plan de rollback y de respuesta a incidentes

### Beneficios
Tratar al agente como software (CI/CD + evaluaciones como pruebas) es lo que separa un demo de una solución operable: puedes cambiar modelo, prompt o tools con confianza porque el pipeline detecta regresiones antes que tus usuarios.
