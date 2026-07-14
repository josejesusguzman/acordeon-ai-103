# Tutorial: cómo crear un custom tool para tu agente

### Descripción
Las *tools* son lo que convierte a un LLM en agente: le permiten ejecutar acciones (consultar una base de datos, enviar un correo, llamar una API). Un **custom tool** es una función tuya que el agente decide invocar cuando la necesita (function calling).

> [!NOTE]
> Módulo de referencia: [Build an agent with custom tools](https://learn.microsoft.com/es-es/training/modules/build-agent-with-custom-tools/)

### Cómo funciona por debajo
1. Registras la función con su **descripción y parámetros** (esquema JSON).
2. El modelo, al leer la pregunta del usuario, decide si necesita la tool y genera los **argumentos**.
3. El runtime ejecuta TU código Python con esos argumentos.
4. El resultado vuelve al modelo, que redacta la respuesta final.

> La descripción de la función es un prompt: de su claridad depende que el agente la use bien.

### Tutorial paso a paso (Foundry Agent Service, Python)

```python
import json
from azure.ai.projects import AIProjectClient
from azure.ai.agents.models import FunctionTool, ToolSet
from azure.identity import DefaultAzureCredential

# 1. Define tu función normal de Python (type hints + docstring = esquema)
def consultar_pedido(numero_pedido: str) -> str:
    """
    Consulta el estado de un pedido en el sistema de la empresa.

    :param numero_pedido: Número de pedido, formato PED-XXXX.
    :return: Estado del pedido en JSON.
    """
    # Aquí iría tu lógica real (BD, API interna...)
    estados = {"PED-0001": "enviado", "PED-0002": "en preparación"}
    return json.dumps({"pedido": numero_pedido,
                       "estado": estados.get(numero_pedido, "no encontrado")})

# 2. Empaqueta la(s) función(es) como tools del agente
toolset = ToolSet()
toolset.add(FunctionTool(functions={consultar_pedido}))

# 3. Crea el agente con las tools
project = AIProjectClient(endpoint=PROJECT_ENDPOINT, credential=DefaultAzureCredential())
agents_client = project.agents
agents_client.enable_auto_function_calls(toolset)   # ejecución automática

agent = agents_client.create_agent(
    model="gpt-4o",
    name="agente-pedidos",
    instructions=(
        "Eres un asistente de pedidos. Usa la herramienta consultar_pedido "
        "cuando el usuario pregunte por el estado de un pedido. "
        "Si el pedido no existe, pídele que verifique el número."
    ),
    toolset=toolset
)

# 4. Conversa: el agente decide cuándo llamar tu función
thread = agents_client.threads.create()
agents_client.messages.create(thread_id=thread.id, role="user",
                              content="¿Cómo va mi pedido PED-0001?")
run = agents_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)

for m in agents_client.messages.list(thread_id=thread.id):
    print(m.role, ":", m.content)
```

### Buenas prácticas para custom tools

1. **Descripciones precisas:** di qué hace, cuándo usarla y el formato de los parámetros.
2. **Devuelve JSON estructurado**, no prosa: el modelo lo interpreta mejor.
3. **Maneja errores dentro de la tool** y devuelve mensajes útiles ("pedido no encontrado") en vez de lanzar excepciones crudas.
4. **Valida entradas:** el modelo puede generar argumentos inválidos; tu función es la última línea de defensa.
5. **Principio de mínimo privilegio:** la tool solo debe poder hacer lo que el agente necesita (cuidado con tools que borran/escriben).
6. **Idempotencia cuando sea posible:** el agente puede reintentar llamadas.

### Otras formas de dar tools a un agente
- **OpenAPI tool:** apunta el agente a una spec OpenAPI y expone la API completa.
- **MCP tools:** conecta servidores Model Context Protocol (ver tema de protocolos).
- **Tools integradas:** Code Interpreter, File Search, Bing Grounding, Azure Functions, Logic Apps.

### Beneficios
Con custom tools el agente deja de "hablar" sobre tu negocio y empieza a **operar** en él, manteniendo tu lógica en código normal, probado y versionado.
