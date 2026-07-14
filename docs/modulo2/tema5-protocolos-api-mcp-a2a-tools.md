# Protocolos: API, MCP, A2A y tools — qué es cada uno y cuándo usarlo

### Descripción
Alrededor de los agentes hay una sopa de siglas que en realidad describe **capas distintas de integración**. Entenderlas por separado aclara todo el diseño de una solución agéntica.

> [!NOTE]
> Módulos de referencia: [Connect an agent to MCP tools](https://learn.microsoft.com/es-es/training/modules/connect-agent-to-mcp-tools/) y [Discover agents with A2A](https://learn.microsoft.com/es-es/training/modules/discover-agents-with-a2a/)

### Las capas, de adentro hacia afuera

1. **Tools (function calling):**
   - Qué es: el mecanismo nativo por el que un LLM invoca funciones que tú registras (nombre + descripción + esquema de parámetros).
   - Cuándo: SIEMPRE que el agente deba actuar. Es la base; MCP y OpenAPI son formas de *empaquetar* tools.

2. **API (REST/OpenAPI):**
   - Qué es: la integración clásica servicio-a-servicio. En Foundry puedes dar a un agente una spec **OpenAPI** y el agente aprende todas sus operaciones como tools automáticamente.
   - Cuándo: ya tienes APIs corporativas documentadas y quieres exponerlas al agente sin escribir wrappers; o tu app llama al agente (el agente MISMO se expone como API).

3. **MCP (Model Context Protocol):**
   - Qué es: protocolo abierto (Anthropic, adoptado por Microsoft) que estandariza cómo un agente descubre y usa **tools y datos** de un servidor externo. Piensa "USB-C de las tools": un servidor MCP publica sus herramientas y cualquier agente compatible las consume sin código a medida.
   - En Foundry: los agentes se conectan a servidores MCP (`MCPTool` con `server_url`); Foundry IQ expone las knowledge bases como servidor MCP; Azure AI Language y Speech también publican servidores MCP.
   - Cuándo: quieres compartir el mismo conjunto de tools entre muchos agentes/plataformas, o consumir tools de terceros. Relación **agente ↔ herramientas/datos**.

4. **A2A (Agent-to-Agent):**
   - Qué es: protocolo abierto (Google, adoptado por Microsoft) para que **agentes hablen con agentes**, incluso de distintas organizaciones/plataformas. Cada agente publica una **Agent Card** (JSON con identidad, capacidades, endpoint) que permite descubrirlo e invocarlo mediante tareas.
   - Cuándo: sistemas multi-agente distribuidos: tu agente de compras delega en el agente del proveedor. Relación **agente ↔ agente**.

### Tabla comparativa (la pregunta de examen)

| Protocolo | Conecta | Analogía | Ejemplo |
|---|---|---|---|
| Tool / function calling | LLM → tu función | Manos del agente | `consultar_pedido()` |
| OpenAPI | Agente → API REST existente | Manual de una API | Exponer tu ERP al agente |
| MCP | Agente → servidor de tools/datos | USB-C de herramientas | Foundry IQ, GitHub MCP server |
| A2A | Agente → otro agente | Teléfono entre agentes | Delegar tarea a agente externo |

### Conexión a un servidor MCP en Foundry (Python)

```python
from azure.ai.projects.models import PromptAgentDefinition, MCPTool

mcp_tool = MCPTool(
    server_label="docs-empresa",
    server_url="https://<search>.search.windows.net/knowledgebases/docs/mcp"
)
agent = project_client.agents.create_version(
    agent_name="asistente-docs",
    definition=PromptAgentDefinition(
        model="gpt-4o-mini",
        instructions="Responde usando la base de conocimiento y cita fuentes.",
        tools=[mcp_tool]
    )
)
```

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
