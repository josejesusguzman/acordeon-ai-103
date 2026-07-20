# Protocolos: API, MCP, A2A y tools — qué es cada uno y cuándo usarlo

### Descripción
Alrededor de los agentes hay una sopa de siglas que en realidad describe **capas distintas de integración**: tools (el mecanismo), OpenAPI (reutilizar APIs), MCP (estandarizar herramientas/datos) y A2A (federar agentes). Entenderlas por separado — y saber **cuándo y por qué** usar cada una — aclara todo el diseño de una solución agéntica. Al final hay un tutorial corto con código para implementar cada una.

> [!NOTE]
> Módulos de referencia: [Connect an agent to MCP tools](https://learn.microsoft.com/es-mx/training/modules/connect-agent-to-mcp-tools/) y [Discover agents with A2A](https://learn.microsoft.com/es-mx/training/modules/discover-agents-with-a2a/)

### Las capas, de adentro hacia afuera: cuándo y por qué

#### 1. Tools (function calling) — el mecanismo base

- **Qué es:** el mecanismo nativo por el que un LLM invoca funciones que tú registras (nombre + descripción + esquema de parámetros). El modelo no ejecuta nada: **pide** la llamada y tu código (o el Agent Service) la ejecuta y le devuelve el resultado.
- **Cuándo:** SIEMPRE que el agente deba actuar u obtener datos frescos. Es la base; OpenAPI y MCP son formas de *empaquetar* tools.
- **Por qué:** control total — tú escribes la función, validas parámetros y decides qué puede hacer. Cero dependencias externas.
- **Señal de que te quedas aquí:** pocas funciones (2-10), propias de esta app, que nadie más va a reutilizar.

#### 2. API (REST/OpenAPI) — reutilizar lo que ya existe

- **Qué es:** en Foundry le das al agente una **especificación OpenAPI** y aprende TODAS sus operaciones como tools automáticamente, sin escribir un wrapper por endpoint.
- **Cuándo:** ya tienes APIs corporativas documentadas (ERP, CRM, servicios internos) y quieres exponerlas al agente; o al revés, tu app llama al agente (el agente mismo se expone como API).
- **Por qué:** cero código de integración — la spec ES la integración; los equipos dueños de la API no tienen que aprender nada de agentes; la autenticación se declara en la tool (anónima, API key o managed identity).
- **Señal:** "esa capacidad ya existe como API REST" → OpenAPI tool, no reescribas funciones.

#### 3. MCP (Model Context Protocol) — estandarizar herramientas y datos

- **Qué es:** protocolo abierto (Anthropic, adoptado por Microsoft) que estandariza cómo un agente **descubre e invoca** tools y datos de un servidor externo. Piensa "USB-C de las tools": el servidor publica sus herramientas y cualquier agente compatible las consume **sin código a medida**, incluso si el servidor agrega tools nuevas mañana (descubrimiento dinámico en runtime).
- **En Foundry:** el Agent Service trae soporte nativo de servidores MCP remotos (`MCPTool`); Foundry IQ expone knowledge bases como servidor MCP; Azure AI Language/Speech y GitHub publican servidores MCP.
- **Cuándo:** quieres compartir el mismo conjunto de tools entre muchos agentes/plataformas (VS Code, Foundry, Claude, Copilot Studio…); consumir ecosistema de terceros; o que las tools evolucionen sin redesplegar agentes. Relación **agente ↔ herramientas/datos**.
- **Por qué:** una sola integración sirve para N agentes; el descubrimiento es dinámico (vs. OpenAPI, que es una spec estática que tú registras); y trae control de aprobación humana integrado (`require_approval`).
- **Señal:** "estas herramientas las van a usar varios agentes/equipos" o "quiero enchufar el servidor MCP de X".

#### 4. A2A (Agent-to-Agent) — federar agentes

- **Qué es:** protocolo abierto (Google, adoptado por Microsoft) para que **agentes hablen con agentes**, incluso de distintas organizaciones/plataformas. Cada agente publica **aptitudes (skills)** — id, nombre, descripción, tags, ejemplos, modos de entrada/salida — y una **Agent Card** (su "tarjeta de presentación digital": identidad, endpoint, capacidades como streaming/push, skills y requisitos de autenticación) que permite descubrirlo e invocarlo mediante tareas.
- **Cuándo:** sistemas multi-agente **distribuidos**: tu agente de compras delega en el agente del proveedor; agentes de distintos equipos/plataformas que deben colaborar sin fusionarse. Relación **agente ↔ agente**.
- **Por qué (ventajas oficiales):** colaboración entre plataformas tradicionalmente desconectadas; **cada agente elige su propio LLM** (a diferencia de MCP, donde las tools cuelgan del modelo del agente que las llama); y **autenticación integrada** en el protocolo.
- **Señal:** "el otro agente no es mío" o "cada agente vive en su propia plataforma/organización".

### Tabla comparativa (la pregunta de examen)

| Protocolo | Conecta | Analogía | Descubrimiento | Ejemplo |
|---|---|---|---|---|
| Tool / function calling | LLM → tu función | Manos del agente | Tú lo registras | `consultar_pedido()` |
| OpenAPI | Agente → API REST existente | Manual de una API | Spec estática | Exponer tu ERP al agente |
| MCP | Agente → servidor de tools/datos | USB-C de herramientas | Dinámico en runtime | Foundry IQ, GitHub MCP server |
| A2A | Agente → otro agente | Teléfono entre agentes | Agent Card | Delegar tarea a agente externo |

Regla mental: **MCP conecta agentes con herramientas; A2A conecta agentes con agentes.** Si la pregunta menciona "Agent Card" o "skills descubribles" → A2A; si menciona "server_url de tools" o "USB-C" → MCP; si menciona "spec ya documentada" → OpenAPI.

### Tutoriales cortos: implementar cada protocolo

#### Tutorial 1: Function calling (tool propia)

```python
from azure.ai.agents.models import FunctionTool, ToolSet

def consultar_pedido(order_id: str) -> str:
    """Devuelve el estado del pedido indicado."""     # la docstring ES la descripción que lee el LLM
    return json.dumps({"order_id": order_id, "estado": "en tránsito"})

toolset = ToolSet()
toolset.add(FunctionTool(functions={consultar_pedido}))
agents_client.enable_auto_function_calls(toolset)     # ejecuta la función automáticamente

agent = agents_client.create_agent(
    model="gpt-4o-mini",
    name="agente-pedidos",
    instructions="Usa consultar_pedido cuando el usuario pregunte por su pedido.",
    toolset=toolset,
)
```

Claves: nombre y docstring descriptivos (el modelo decide con eso), type hints en los parámetros (generan el esquema) y validar dentro de la función como si el input viniera de un usuario — porque en la práctica así es.

#### Tutorial 2: OpenAPI tool (API existente)

```python
import json
from azure.ai.agents.models import OpenApiTool, OpenApiAnonymousAuthDetails

with open("api_clima.json") as f:          # tu especificación OpenAPI 3.0
    spec = json.load(f)

clima_tool = OpenApiTool(
    name="api_clima",
    spec=spec,
    description="Consulta el pronóstico del clima por ciudad",
    auth=OpenApiAnonymousAuthDetails(),    # o autenticación con managed identity / API key
)

agent = agents_client.create_agent(
    model="gpt-4o-mini",
    name="agente-clima",
    instructions="Responde preguntas del clima usando la API.",
    tools=clima_tool.definitions,          # TODAS las operaciones de la spec, como tools
)
```

Claves: cuida que la spec tenga `description` útiles por operación (es lo que lee el modelo para elegir); en el portal puedes hacer lo mismo desde el editor del agente, sección **Tools → Add → OpenAPI**, pegando la spec.

#### Tutorial 3: MCP (servidor de herramientas estándar)

```python
from azure.ai.projects.models import PromptAgentDefinition, MCPTool

mcp_tool = MCPTool(
    server_label="github",
    server_url="https://api.githubcopilot.com/mcp/",
    allowed_tools=["search_repositories"],   # opcional: limita qué tools puede usar
    require_approval="always",               # "always" (default) o "never"
)
mcp_tool.update_headers("Authorization", f"Bearer {token}")  # auth del servidor

agent = project_client.agents.create_version(
    agent_name="asistente-github",
    definition=PromptAgentDefinition(
        model="gpt-4o-mini",
        instructions="Usa las herramientas del servidor MCP para responder.",
        tools=[mcp_tool],
    ),
)
```

Claves: no necesitas crear sesión de cliente MCP ni envolver funciones — el Agent Service descubre e invoca las tools automáticamente. Con `require_approval="always"`, la respuesta del agente incluirá un `mcp_approval_request` (qué tool quiere invocar y con qué argumentos); apruebas enviando un `mcp_approval_response` con el `approval_request_id` y `approve=True`. Puedes conectar **varios servidores MCP** al mismo agente como tools independientes. Para conocimiento propio, apunta el `server_url` a la knowledge base de **Foundry IQ**.

#### Tutorial 4: A2A (exponer y consumir un agente)

**Exponer tu agente** (esbozo con el SDK de A2A): defines skills y Agent Card, implementas un *executor* que atiende las tareas y lo hospedas como servidor:

```python
from a2a.types import AgentCard, AgentSkill, AgentCapabilities

skill = AgentSkill(
    id="generar_titulo",
    name="Generar título",
    description="Genera un título atractivo para un artículo técnico",
    tags=["escritura", "titulos"],
    examples=["Dame un título para un artículo sobre RAG"],
)

card = AgentCard(
    name="Agente de Títulos",
    description="Especialista en títulos de artículos técnicos",
    url="https://titulos.miempresa.com/",       # endpoint del servicio A2A
    version="1.0",
    default_input_modes=["text"], default_output_modes=["text"],
    capabilities=AgentCapabilities(streaming=True),
    skills=[skill],
)
# + AgentExecutor que invoca a tu agente de Foundry, hospedado como servidor HTTP
```

**Consumir un agente remoto:** el cliente (u orquestador) recupera la Agent Card del endpoint conocido, descubre sus skills y le envía la tarea:

```python
card = await A2ACardResolver(base_url="https://titulos.miempresa.com").get_agent_card()
client = A2AClient(agent_card=card)
respuesta = await client.send_message("Título para un artículo sobre evaluaciones en Foundry")
```

Claves: la Agent Card debe representar fielmente skills y endpoint — es lo único que el otro agente ve para decidir delegarte; el flujo del módulo oficial es **definir skills/card → implementar el executor → hospedar el servidor → conectar el cliente**. Ejemplo canónico: un agente de enrutamiento descubre las cards de un "agente de títulos" y un "agente de esquemas" y encadena sus resultados.

### Cómo se combinan en una solución real
```text
Usuario ──> Agente principal (Foundry)
              ├─ tools locales (function calling)
              ├─ MCP ──> Foundry IQ (conocimiento) / servidores de tools
              ├─ OpenAPI ──> APIs corporativas
              └─ A2A ──> agentes especializados de otros equipos/empresas
```

### Beneficios
Usar el protocolo correcto por capa evita reinventar integraciones: tools para actuar, OpenAPI para reutilizar APIs, MCP para estandarizar herramientas/conocimiento y A2A para federar agentes.
