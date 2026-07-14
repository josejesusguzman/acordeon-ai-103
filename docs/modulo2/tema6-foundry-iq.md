# Qué es y cómo se usa Foundry IQ

### Descripción
**Foundry IQ** es la plataforma administrada de conocimiento para agentes, construida sobre **Azure AI Search**. Resuelve el problema de construir RAG a mano: en lugar de montar un pipeline de indexación/embeddings/retrieval por cada agente, creas **knowledge bases** una vez y las conectas a todos los agentes que quieras.

> [!NOTE]
> Módulo de referencia: [Build knowledge-enhanced AI agents with Foundry IQ](https://learn.microsoft.com/es-es/training/modules/introduction-foundry-iq/)

### Aspectos Clave

1. **El problema que resuelve:** RAG artesanal implica vector DB, pipeline de embeddings, tuning de retrieval y mantenimiento… multiplicado por cada agente. Foundry IQ lo convierte en un servicio compartido.

2. **Knowledge bases por dominio de negocio:** organizas el conocimiento como "Documentación de producto" o "Políticas de RRHH", no como "contenedor blob B". Una knowledge base puede mezclar fuentes: SharePoint + Blob + OneLake + índices existentes, y el agente la ve como una sola fuente unificada.

3. **Qué hace automáticamente al conectar una fuente:**
   1. **Discovery:** escanea los documentos de la ubicación.
   2. **Processing:** hace chunking y genera embeddings.
   3. **Indexing:** el contenido queda buscable.
   4. **Monitoring:** cambios en los documentos disparan reindexación automática.

4. **Retrieval agéntico integrado:** al recibir una consulta, Foundry IQ analiza la pregunta, elige estrategia (keyword, semántica, expansión de query), ejecuta búsquedas (en paralelo si hay varias fuentes), **rankea** resultados y devuelve **citas** de los documentos fuente. Todo sin código custom.

5. **Se consume vía MCP:** la knowledge base se expone como servidor MCP; conectar un agente es agregar una tool:

```python
from azure.ai.projects import AIProjectClient
from azure.ai.projects.models import PromptAgentDefinition, MCPTool

project_client = AIProjectClient(endpoint=project_endpoint, credential=credential)

knowledge_tool = MCPTool(
    server_label="product-docs",
    server_url=f"{search_endpoint}/knowledgebases/product-documentation/mcp"
)

agent = project_client.agents.create_version(
    agent_name="product-support-agent",
    definition=PromptAgentDefinition(
        model="gpt-4o-mini",
        instructions="Answer product questions using the knowledge base. Always cite your sources.",
        tools=[knowledge_tool]
    )
)
```

6. **La ventaja del conocimiento compartido:** tres agentes (soporte, RRHH, developers) pueden compartir knowledge bases: mejorar una base beneficia inmediatamente a todos los agentes conectados, con permisos controlando quién accede a qué.

### Cómo usarlo, paso a paso
1. En el portal de Foundry, crea una **knowledge base**.
2. Conecta **data sources** (Blob, SharePoint, Web, OneLake, índice de AI Search — ver tema siguiente).
3. Espera la indexación y prueba consultas en el portal.
4. Conecta el agente con `MCPTool` e **instrucciones de retrieval** explícitas (siempre buscar, siempre citar, fallback si no encuentra).
5. Evalúa y monitorea la calidad de respuestas (groundedness, citas).

### Beneficios
Foundry IQ desplaza tu esfuerzo de construir infraestructura de retrieval a diseñar experiencias de agente: conocimiento centralizado, actualizado automáticamente, con citas y gobernanza, reutilizable por toda la organización.
