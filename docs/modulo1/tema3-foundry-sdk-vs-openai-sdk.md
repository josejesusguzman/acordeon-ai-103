# Cuándo elegir Foundry SDK y cuándo elegir OpenAI SDK

### Descripción
En Azure puedes hablar con los modelos de dos maneras principales: con el **SDK de Microsoft Foundry** (`azure-ai-projects`) o con el **SDK de OpenAI** (`openai`) apuntado a Azure. No son excluyentes: el Foundry SDK de hecho te entrega un cliente OpenAI por debajo, y Microsoft recomienda combinarlos en la misma aplicación cuando conviene. La decisión depende de tres cosas: **qué punto de conexión usas**, **cómo te autenticas** y **qué tan "dentro" del ecosistema Foundry** vive tu solución (agentes, evaluaciones, tracing, conexiones).

> [!NOTE]
> Módulo de referencia: [Desarrollo de una app de chat con Microsoft Foundry — Elección de un punto de conexión y un SDK](https://learn.microsoft.com/es-mx/training/modules/foundry-sdk/03-microsoft-foundry-sdk)

### Los dos puntos de conexión de un proyecto Foundry

Cada proyecto de Foundry expone **dos endpoints**; el SDK que elijas suele derivarse del endpoint que necesitas:

| Endpoint | Formato | SDK natural |
|---|---|---|
| **Project endpoint** | `https://{recurso}.services.ai.azure.com/api/projects/{proyecto}` | Foundry SDK (`AIProjectClient`) |
| **Azure OpenAI endpoint** | `https://{recurso}.openai.azure.com/openai/v1` | OpenAI SDK (`OpenAI` / `AzureOpenAI`) |

Ambos se encuentran en la página **Overview** del proyecto en https://ai.azure.com.

### Aspectos Clave

1. **OpenAI SDK (`openai`):**
   - Interfaz estándar de facto: `chat.completions` / `responses` (la API **Responses** es la recomendada para proyectos nuevos; **ChatCompletions** es la más establecida y compatible entre plataformas).
   - Ideal si: solo consumes inferencia (GPT-4o, GPT-5, etc.), quieres portabilidad entre OpenAI.com y Azure, o migras código existente con cambios mínimos.
   - Limitación: no conoce el resto de Foundry (agentes, conexiones, evaluaciones en la nube, tracing, datasets, índices).

2. **Foundry SDK (`azure-ai-projects` + `azure-ai-agents`):**
   - Punto de entrada único al **proyecto**: modelos, agentes (Foundry Agent Service), invocación/aprobación de herramientas, conexiones (Search, Storage), evaluaciones en la nube, seguimiento y observabilidad, gobernanza y metadatos.
   - Ideal si: creas **agentes**, usas **varios modelos** del catálogo (Phi, Llama, DeepSeek, Mistral…), necesitas herramientas conectadas o gobernanza a nivel proyecto.
   - Nota: al usarlo para chat igual instalas `openai`, porque el cliente de chat del Foundry SDK se deriva del SDK de OpenAI.

3. **Autenticación:** para producción se recomienda **Microsoft Entra ID** (`DefaultAzureCredential`, sin claves). Alternativas: clave de API (guárdala en **Azure Key Vault**, nunca en el código) o variables de entorno (`OPENAI_BASE_URL`, `OPENAI_API_KEY`). Con el OpenAI SDK puedes usar Entra ID vía `get_bearer_token_provider`.

4. **Lo mejor de ambos mundos:** el patrón oficial del curso es conectarte al proyecto con Foundry SDK y pedirle el cliente OpenAI:

```python
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

project = AIProjectClient(endpoint=PROJECT_ENDPOINT, credential=DefaultAzureCredential())

# Cliente OpenAI "gratis" a partir del proyecto (sin keys)
openai_client = project.get_openai_client()
r = openai_client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Explica RAG en una frase"}]
)
```

5. **Modelos no-OpenAI:** los modelos del catálogo (Phi, DeepSeek, Llama…) implementados como *Foundry Models* se consumen igual vía el cliente de chat del proyecto o el endpoint compatible; el OpenAI SDK también funciona contra ese endpoint, pero pierdes las funciones de proyecto.

### Casos de estudio

#### Caso 1: Migración de OpenAI.com a Azure → **OpenAI SDK**

**Contexto:** una startup tiene un chatbot de soporte en producción construido con el SDK de `openai` contra api.openai.com. Un cliente corporativo exige que los datos se procesen en Azure (facturación centralizada, compliance, SLA empresarial).

**Decisión:** OpenAI SDK contra el endpoint Azure OpenAI v1. La migración es cambiar `base_url` y la autenticación; el resto del código (prompts, streaming, function calling) queda intacto:

```python
from openai import OpenAI
from azure.identity import DefaultAzureCredential, get_bearer_token_provider

token_provider = get_bearer_token_provider(
    DefaultAzureCredential(), "https://ai.azure.com/.default"
)
client = OpenAI(
    base_url="https://mi-recurso.openai.azure.com/openai/v1/",
    api_key=token_provider,   # Entra ID, sin claves
)
```

**Por qué no Foundry SDK:** no usan agentes ni evaluaciones; agregar `azure-ai-projects` sería una dependencia extra sin beneficio. La **portabilidad** (poder volver a OpenAI.com o servir a clientes en ambas nubes) es el requisito número uno.

#### Caso 2: Asistente interno de RR. HH. con agente y herramientas → **Foundry SDK**

**Contexto:** una empresa quiere un asistente que responda sobre políticas internas (RAG sobre Azure AI Search), consulte días de vacaciones vía API (function calling con aprobación) y mantenga conversaciones con historial (threads). Seguridad: cero claves en código, solo identidades administradas.

**Decisión:** Foundry SDK. El `AIProjectClient` da acceso al **Agent Service** (agente + tools + threads), recupera la **conexión** a AI Search sin hardcodear endpoints, habilita **tracing** para auditar cada llamada a herramienta y usa `DefaultAzureCredential` con RBAC.

**Por qué no OpenAI SDK solo:** tendrías que reimplementar a mano la orquestación del agente, el manejo de threads y la integración con Search, que Foundry ya administra a nivel proyecto.

#### Caso 3: App multi-modelo con enrutamiento por costo → **Foundry SDK**

**Contexto:** una plataforma de análisis de documentos clasifica cada petición: las triviales (extraer campos) van a un SLM barato (**Phi-4**), las complejas (razonamiento legal) a **GPT-5** o **DeepSeek-R1**. Todos los modelos están implementados en el mismo proyecto Foundry.

**Decisión:** Foundry SDK. Un solo `AIProjectClient` enruta a cualquier deployment del catálogo (OpenAI y no-OpenAI) con la misma autenticación, y la gobernanza del proyecto (cuotas, monitoreo, costos) queda centralizada. Cambiar el modelo es cambiar el nombre del deployment.

**Por qué no OpenAI SDK solo:** funcionaría contra el endpoint compatible, pero perderías conexiones, gobernanza y las funciones de proyecto que hacen manejable operar 4+ modelos.

#### Caso 4: Producto SaaS multi-proveedor → **OpenAI SDK**

**Contexto:** un ISV vende un producto que el cliente despliega donde quiera: unos usan OpenAI.com, otros Azure OpenAI, otros un gateway compatible. El código debe ser idéntico en todos los despliegues.

**Decisión:** OpenAI SDK con configuración por **variables de entorno** (`OPENAI_BASE_URL`, `OPENAI_API_KEY`): el mismo binario corre contra cualquier backend compatible con la API de OpenAI, con **dependencia mínima** de Azure.

**Por qué no Foundry SDK:** ataría el producto a proyectos de Foundry, rompiendo la promesa de portabilidad.

#### Caso 5: Plataforma empresarial con calidad continua → **Foundry SDK (+ OpenAI SDK para inferencia)**

**Contexto:** un banco opera varios copilotos y exige: evaluaciones automáticas en cada release (groundedness, safety), observabilidad de cada interacción y despliegue de datasets de prueba versionados (ver tema 2).

**Decisión:** el patrón híbrido oficial. Foundry SDK para lo que solo Foundry hace: **evaluaciones en la nube**, **tracing**, **datasets e índices**, conexiones y metadatos; y de ese mismo cliente se extrae `get_openai_client()` para la inferencia. Un solo endpoint (el del proyecto), una sola identidad Entra ID, dos SDKs colaborando.

### Tabla de decisión rápida

| Escenario | SDK recomendado |
|---|---|
| Chatbot simple con GPT-4o en Azure | OpenAI SDK |
| Código que debe correr igual en OpenAI.com y Azure | OpenAI SDK |
| Migrar código OpenAI existente con cambios mínimos | OpenAI SDK |
| Agentes con tools, threads y conexiones | Foundry SDK |
| Mezclar GPT + Phi + DeepSeek en una app | Foundry SDK |
| Evaluaciones, tracing y monitoreo integrados | Foundry SDK |
| Autenticación sin claves (Entra ID / RBAC) | Ambos la soportan; nativa en Foundry SDK |
| Proyecto nuevo que vivirá en Azure | Híbrido: Foundry SDK + `get_openai_client()` |

### Regla mental para el examen

- ¿La pregunta menciona **agentes, evaluaciones, tracing, conexiones o datasets**? → Foundry SDK (`AIProjectClient`).
- ¿Menciona **portabilidad, código existente o compatibilidad máxima con la API de OpenAI**? → OpenAI SDK.
- ¿Menciona **API nueva recomendada**? → Responses API; ¿**compatibilidad amplia entre plataformas**? → ChatCompletions.
- ¿Menciona **producción y seguridad**? → Entra ID (`DefaultAzureCredential`), claves solo en Key Vault.


