# Tema Complementario 8: Frameworks de orquestación — Agent Framework vs Semantic Kernel vs LangChain/LangGraph

### Descripción
Puedes construir agentes "a mano" con los SDKs, pero los frameworks de orquestación aportan abstracciones (agentes, tools, memoria, grafos de flujo) ya resueltas. El mapa 2026: **Microsoft Agent Framework** (convergencia de Semantic Kernel + AutoGen) en el mundo Microsoft, y **LangChain/LangGraph** como ecosistema open source dominante fuera de él.

### Aspectos Clave

1. **Microsoft Agent Framework (MAF):**
   - Sucesor oficial: unifica **Semantic Kernel** (orientación empresarial, threads, plugins) y **AutoGen** (multi-agente experimental). Python y .NET.
   - Fortalezas: integración nativa con Foundry (Agent Service, tracing, evaluaciones), patrones de orquestación listos (sequential, concurrent, group chat, handoff, magentic), workflows tipados como grafos, soporte MCP/A2A de primera clase.
   - Elígelo si: tu stack es Azure y quieres el camino soportado por Microsoft a largo plazo.

2. **Semantic Kernel (SK):**
   - El framework empresarial original de Microsoft; sigue existiendo y muchas empresas lo tienen en producción. Conceptos: *kernel*, *plugins* (funciones), *planners* (deprecados a favor de function calling) y agentes.
   - Estado: mantenido, pero la innovación fluye hacia Agent Framework; proyectos nuevos deberían evaluar MAF primero.

3. **LangChain + LangGraph:**
   - **LangChain:** el ecosistema más grande (integraciones con todo: modelos, vector stores, loaders). Bueno para prototipos rápidos y componentes sueltos.
   - **LangGraph:** orquestación como **grafo de estados** explícito (nodos, aristas, checkpoints): control fino de flujos complejos, humanos en el loop, reintentos. Es la pieza "seria" del ecosistema para agentes en producción.
   - Elígelos si: multi-cloud/portabilidad, necesitas una integración exótica, o el equipo ya los domina. Funcionan perfectamente contra Azure OpenAI.

4. **Otros que debes ubicar:** **LlamaIndex** (excelente para pipelines RAG/datos), **CrewAI** (multi-agente declarativo simple), **PydanticAI** (agentes tipados minimalistas).

5. **Tabla comparativa:**

   | Criterio | Agent Framework | Semantic Kernel | LangChain/LangGraph |
   |---|---|---|---|
   | Respaldo | Microsoft (actual) | Microsoft (legado→MAF) | Comunidad/LangChain Inc. |
   | Integración Foundry | Nativa | Buena | Vía conectores |
   | Multi-agente | Patrones integrados | Básico | LangGraph (grafos) |
   | Ecosistema de integraciones | Creciendo | Medio | El más grande |
   | Lenguajes | Python/.NET | Python/.NET/Java | Python/JS |
   | Riesgo de breaking changes | Bajo-medio | Bajo | Históricamente alto |

6. **Criterio de decisión honesto:**
   - App simple (un agente, pocas tools) → **SDK directo de Foundry**, sin framework: menos capas = menos deuda.
   - Multi-agente/workflows en Azure → **Agent Framework**.
   - RAG-céntrico con fuentes variadas → **LlamaIndex** o LangChain.
   - Flujos complejos con control de estado y portabilidad → **LangGraph**.
   - Lo que NO hagas: mezclar dos frameworks de orquestación en la misma app.

### Ejemplo mínimo con Agent Framework (Python)

```python
# pip install agent-framework
from agent_framework.azure import AzureAIAgentClient
from azure.identity import AzureCliCredential

async def main():
    async with AzureAIAgentClient(async_credential=AzureCliCredential()).create_agent(
        name="Asistente",
        instructions="Respondes breve y en español.",
    ) as agent:
        result = await agent.run("¿Qué es MCP en una frase?")
        print(result.text)
```

### Fuentes para profundizar
- [Microsoft Agent Framework](https://learn.microsoft.com/es-es/agent-framework/) · [Repo GitHub](https://github.com/microsoft/agent-framework)
- [LangGraph](https://langchain-ai.github.io/langgraph/) · [LlamaIndex](https://docs.llamaindex.ai/)

### Beneficios
Conocer el mapa de frameworks evita casarte con el equivocado: eliges por ecosistema y necesidades de orquestación, sabiendo que los conceptos (agentes, tools, grafos, memoria) se transfieren entre todos.
