# Onboarding: qué instalar y qué debes saber antes de iniciar

### Descripción
Antes de escribir una sola línea de código para la AI-103 necesitas preparar tu entorno de desarrollo y dominar un vocabulario mínimo. Este tema resume las herramientas a instalar y los conceptos que el curso da por sentados.

> [!NOTE]
> Módulo de referencia: [Prepararse para el desarrollo de IA en Azure](https://learn.microsoft.com/es-es/training/modules/prepare-azure-ai-development/)

### Cosas a instalar

1. **Visual Studio Code** con las extensiones *Python*, *Jupyter* y *GitHub Copilot*. Es el editor usado en todos los laboratorios oficiales.
2. **Python 3.10 o superior**. Verifica con `python --version`. Se recomienda trabajar siempre con entornos virtuales:
   ```bash
   python -m venv .venv
   source .venv/bin/activate   # En Windows: .venv\Scripts\activate
   ```
3. **Azure CLI**, para autenticarte y administrar recursos desde terminal:
   ```bash
   az login
   az account show
   ```
4. **Git**, para clonar los repositorios de laboratorios (por ejemplo `mslearn-ai-studio`, `mslearn-ai-agents`).
5. **SDKs de Python** que usarás durante todo el curso:
   ```bash
   pip install azure-ai-projects azure-identity openai azure-ai-agents
   ```
6. **Una suscripción de Azure** con permisos para crear recursos y un **proyecto de Microsoft Foundry** (antes Azure AI Foundry). Todo el curso gira alrededor del portal de Foundry: https://ai.azure.com

> [!IMPORTANT]
> Si usas Python para otros proyectos en tu mismo equipo te recomiendo crear un entorno virtual para aislar dependencias. Por ejemplo, en Linux/macOS:
> ```bash
> python -m venv .venv
> source .venv/bin/activate
> ```
> En Windows:
> ```bash
> python -m venv .venv
> .venv\Scripts\activate
> ```

### Conceptos que debes saber antes de iniciar

1. **LLM (Large Language Model):** modelo entrenado con enormes cantidades de texto que predice el siguiente token. Es el motor de la IA generativa.
2. **Token:** unidad mínima de texto que procesa un modelo (~4 caracteres en inglés). El costo y los límites se miden en tokens.
3. **Prompt / Completion:** la entrada que envías al modelo y la respuesta que genera.
4. **Deployment (implementación):** en Azure no llamas "al modelo" directamente; primero lo *implementas* con un nombre propio y llamas a ese deployment.
5. **Endpoint y Key:** URL y credencial de tu recurso. Con Microsoft Entra ID puedes evitar keys usando `DefaultAzureCredential` (práctica recomendada).
6. **Hub y Proyecto de Foundry:** el *hub* agrupa recursos compartidos (almacenamiento, cómputo, conexiones); el *proyecto* es tu espacio de trabajo donde implementas modelos, creas agentes y evalúas.
7. **RAG (Retrieval Augmented Generation):** patrón que recupera información de tus datos y la inyecta en el prompt para que el modelo responda con datos actuales y propios.
8. **Agente:** LLM + instrucciones + herramientas (tools) que puede ejecutar acciones, no solo generar texto.
9. **Temperatura / top_p:** parámetros que controlan la aleatoriedad de la respuesta. Temperatura baja = respuestas deterministas; alta = más creativas.
10. **Cuota y regiones:** cada modelo tiene disponibilidad y cuota por región (TPM: tokens por minuto). Si un deployment falla, revisa la región y la cuota antes que el código.

### Primer código de conexión (patrón que verás en todo el curso)

```python
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

project_client = AIProjectClient(
    endpoint="https://<tu-recurso>.services.ai.azure.com/api/projects/<tu-proyecto>",
    credential=DefaultAzureCredential()
)

openai_client = project_client.get_openai_client()
response = openai_client.chat.completions.create(
    model="gpt-4o-mini",  # nombre de TU deployment
    messages=[{"role": "user", "content": "Hola, ¿estás listo para la AI-103?"}]
)
print(response.choices[0].message.content)
```

### Beneficios
Con el entorno listo y estos conceptos claros, los laboratorios de Microsoft Learn se vuelven mecánicos: todo ejercicio es una variación de "crear proyecto → implementar modelo → conectarse con el SDK → invocar".
