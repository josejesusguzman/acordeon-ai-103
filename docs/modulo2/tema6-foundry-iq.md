# Qué es y cómo se usa Foundry IQ

### Descripción
**Foundry IQ** es la capa de conocimiento administrada para agentes, construida sobre **Azure AI Search**. Resuelve el problema de construir RAG a mano: en lugar de montar un pipeline de indexación/embeddings/retrieval por cada agente, creas **knowledge bases** una vez y las conectas a todos los agentes que quieras — con permisos, citas y actualización automática. Algunas funciones son GA (API 2026-04-01) y otras siguen en versión preliminar (2026-05-01-preview); el portal ofrece acceso preliminar a todo.

> [!NOTE]
> Referencias: [¿Qué es Foundry IQ?](https://learn.microsoft.com/es-mx/azure/foundry/agents/concepts/what-is-foundry-iq), [módulo Build knowledge-enhanced AI agents](https://learn.microsoft.com/es-mx/training/modules/introduction-foundry-iq/) y [Creación de una knowledge base](https://learn.microsoft.com/es-mx/azure/search/agentic-retrieval-how-to-create-knowledge-base)

### Aspectos Clave

1. **El problema que resuelve:** RAG artesanal implica vector DB, pipeline de embeddings, tuning de retrieval y mantenimiento… multiplicado por cada agente. Foundry IQ lo convierte en un servicio compartido: 3 agentes = 3 sistemas RAG con el enfoque tradicional; con Foundry IQ = knowledge bases compartidas.

2. **Knowledge bases por dominio de negocio:** organizas el conocimiento como "Documentación de producto" o "Políticas de RRHH", no como "contenedor blob B". Una knowledge base mezcla **knowledge sources**: SharePoint + Blob + OneLake + índices de AI Search + web pública, y el agente la ve como una sola fuente unificada.

3. **Qué hace automáticamente al conectar una fuente:** **Discovery** (escanea documentos) → **Processing** (chunking + embeddings) → **Indexing** (buscable) → **Monitoring** (reindexación automática/programada ante cambios).

4. **Retrieval agéntico integrado:** ante una consulta, descompone preguntas complejas en **subconsultas**, elige estrategia (keyword, vectorial, híbrida), las ejecuta **en paralelo** sobre las fuentes, **re-rankea semánticamente** y devuelve respuestas con **citas**. El nivel de razonamiento es configurable (**mínimo / bajo / medio**) y puede usar un LLM de Azure OpenAI para planear consultas y sintetizar respuestas (`output_mode="answerSynthesis"`).

5. **Seguridad de nivel empresarial (clave de examen):** sincroniza **ACLs** de las fuentes compatibles, reconoce **etiquetas de confidencialidad de Microsoft Purview** y puede ejecutar las consultas con la **identidad Entra del usuario que llama** — el agente solo devuelve contenido que ese usuario tiene permiso de ver.

6. **Cómo se consume:** vía **MCP** (la knowledge base se expone como servidor MCP) o con la acción **retrieve** de la API; sirve a Foundry Agent Service, Agent Framework o cualquier app.

### Componentes

| Componente | Qué es |
|---|---|
| **Knowledge base** | Objeto de nivel superior: define qué fuentes consultar y los parámetros de recuperación (instrucciones, razonamiento, modo de salida) |
| **Knowledge sources** | Conexiones a contenido indexado o remoto (Blob, SharePoint, OneLake, índices, web) |
| **Retrieval agéntico** | Pipeline multi-consulta: descompone, paraleliza, re-rankea y unifica con citas |

### Tutorial parte A: como usuario del portal (sin código)

Flujo oficial en https://ai.azure.com — ideal para prueba de concepto (puedes usar el **nivel gratuito** de AI Search y la asignación gratuita de tokens de retrieval):

1. Inicia sesión y asegúrate de que el interruptor **New Foundry** esté **activado** (estos pasos son del portal nuevo):

   ![Interruptor New Foundry activado](https://learn.microsoft.com/es-mx/azure/foundry/media/version-banner/new-foundry.png)

2. Crea un proyecto o selecciona uno existente.
3. En el menú superior selecciona **Build** y abre la pestaña **Knowledge (Conocimientos)**:
   1. Crea o conéctate a un **servicio de Azure AI Search** que soporte retrieval agéntico.
   2. Crea la **knowledge base** agregando **un knowledge source a la vez** (Blob, SharePoint, OneLake, índice existente o web).
   3. Configura las **propiedades de recuperación** (instrucciones, nivel de razonamiento).
4. Ve a la pestaña **Agents**: crea o selecciona un agente y **conéctalo a la knowledge base**.
5. Prueba en el **playground**: haz preguntas cuyo contenido esté en tus documentos y verifica que responde **con citas**; refina las instrucciones del agente ahí mismo.

> [!NOTE]
> El playground es para prueba de concepto. En Azure Portal también puedes crear knowledge bases, pero la **conexión con agentes** se hace en el portal de Foundry o por código. Al pasar a producción, configura identidades administradas y permisos (parte B).

### Tutorial parte B: como ingeniero de IA (SDK)

**Prerrequisitos:** AI Search en región con retrieval agéntico (Basic+ si usas managed identity); rol **Search Service Contributor** para crear knowledge bases; si la KB usa LLM, la managed identity del servicio de búsqueda necesita **Cognitive Services User** sobre el recurso Foundry; `pip install azure-search-documents` (o `--pre` para las funciones preview).

**Paso 1: Crea la knowledge base por código** (control total de instrucciones de retrieval y síntesis):

```python
from azure.identity import DefaultAzureCredential
from azure.search.documents.indexes import SearchIndexClient
from azure.search.documents.indexes.models import (
    KnowledgeBase, KnowledgeSourceReference,
    KnowledgeBaseAzureOpenAIModel, AzureOpenAIVectorizerParameters,
)

index_client = SearchIndexClient(endpoint=search_url, credential=DefaultAzureCredential())

kb = KnowledgeBase(
    name="product-documentation",
    description="Documentación de producto para agentes de soporte",
    knowledge_sources=[KnowledgeSourceReference(name="docs-blob-ks"),
                       KnowledgeSourceReference(name="specs-sharepoint-ks")],
    retrieval_instructions="Usa docs-blob para preguntas técnicas; specs-sharepoint para especificaciones.",
    answer_instructions="Respuesta concisa de dos frases basada en los documentos recuperados.",
    output_mode="answerSynthesis",         # el LLM sintetiza la respuesta (preview)
    models=[KnowledgeBaseAzureOpenAIModel(azure_open_ai_parameters=AzureOpenAIVectorizerParameters(
        resource_url=aoai_endpoint, deployment_name="gpt-4o-mini", model_name="gpt-4o-mini"))],
)
index_client.create_or_update_knowledge_base(kb)
```

**Paso 2: Conecta el agente vía MCP:**

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
    definition=PromptAgentDefinition(model="gpt-4o-mini",
        instructions=retrieval_instructions, tools=[knowledge_tool])
)
```

**Paso 3: Instrucciones de retrieval efectivas.** Solo hay un comportamiento aceptable en un agente empresarial: **busca, cita y se mantiene en la base** (los otros dos — responder del entrenamiento, o responder sin citar — fallan en precisión o en responsabilidad). Las instrucciones deben definir **cuándo buscar, cómo citar y qué hacer sin respuesta**:

```python
retrieval_instructions = """Eres un asistente de soporte.
REGLAS CRÍTICAS:
- SIEMPRE busca en la knowledge base antes de responder; NUNCA respondas de tu entrenamiento.
- Toda respuesta lleva citas en el formato 【doc_id:search_id†nombre_fuente】.
- Si la base no tiene la respuesta: "No tengo esa información en la documentación actual;
  contacta a soporte@empresa.com". No adivines."""
```

**Paso 4: Prueba sistemática** con los 4 tipos de consulta: fáctica simple (retrieval directo con cita), de síntesis (varios documentos, varias citas), **fuera de la base** (debe activar el fallback, no alucinar) y ambigua (debe pedir aclaración):

```python
openai_client = project_client.get_openai_client()
conversation = openai_client.conversations.create()
response = openai_client.responses.create(
    conversation=conversation.id,
    input="¿Cuántos días de vacaciones me tocan?",
    extra_body={"agent": {"name": agent.name, "type": "agent_reference"}})
print(response.output_text)
```

**Paso 5: Producción.** Monitorea **frecuencia de citas**, **tasa de fallback** ("no sé"), tipos de consulta y precisión del retrieval; combina con los evaluadores de groundedness (módulo 1, tema 2) y ajusta instrucciones/contenido iterativamente.

### Foundry IQ vs Work IQ (y Fabric IQ): cuándo usar cada uno

Microsoft ofrece **tres cargas de trabajo "IQ"** — independientes pero combinables — que dan a los agentes acceso a distintos aspectos de la organización:

| | **Foundry IQ** | **Work IQ** | **Fabric IQ** |
|---|---|---|---|
| **Qué es** | Capa de **conocimiento administrada** para datos empresariales (Azure AI Search) | Capa de **inteligencia contextual de Microsoft 365** | Capa de **inteligencia semántica de datos** (Microsoft Fabric) |
| **Qué conecta** | Documentos y datos estructurados/no estructurados: Blob, SharePoint, OneLake, índices, web | Señales de colaboración: correos, documentos, reuniones, chats, hábitos y flujos de trabajo | Datos analíticos: ontologías, modelos semánticos, grafos y agentes de datos sobre OneLake/Power BI |
| **Para quién** | Agentes que construyes en **Foundry** (Agent Service, Agent Framework, apps propias) | **Microsoft 365 Copilot** y agentes del ecosistema M365 | Agentes y usuarios que razonan sobre **analítica de negocio** |
| **Pregunta típica** | "¿Qué dice nuestra documentación/política sobre X?" | "¿En qué está trabajando mi equipo y cómo colabora?" | "¿Por qué cayeron las ventas de la región norte este trimestre?" |

**Cuándo usar cada uno:**

- **Foundry IQ** → estás construyendo agentes propios que necesitan **conocimiento documental** con permisos y citas (RAG administrado). Es el que cubre la AI-103.
- **Work IQ** → tu escenario vive en **Microsoft 365**: quieres que Copilot o tus agentes entiendan el contexto de trabajo (quién, qué, cuándo) de la organización; se consume desde la extensibilidad de M365 Copilot, no desde Foundry.
- **Fabric IQ** → tus agentes deben razonar sobre **datos analíticos con semántica de negocio** (conceptos como "cliente", "margen", "región" definidos una vez en Fabric).
- **Juntas:** un agente completo puede usar las tres — Work IQ para el contexto de trabajo, Fabric IQ para los datos de negocio y Foundry IQ para el conocimiento documental.

### Beneficios
Foundry IQ desplaza tu esfuerzo de construir infraestructura de retrieval a diseñar experiencias de agente: conocimiento centralizado, actualizado automáticamente, con citas, permisos (ACLs + Purview + Entra) y gobernanza, reutilizable por toda la organización.
