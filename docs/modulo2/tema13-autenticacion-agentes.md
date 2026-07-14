# Métodos de autenticación de agentes

### Descripción
Un agente tiene DOS problemas de identidad: (1) cómo tu código/usuarios se autentican **hacia** el agente y los servicios de Azure, y (2) cómo el agente se autentica **hacia** las herramientas y APIs que usa. Resolver ambos sin regar claves por el código es parte del diseño de producción.

> [!NOTE]
> Contexto en Learn: módulos de agentes ([Foundry Agent Service](https://learn.microsoft.com/es-es/training/modules/develop-ai-agents-azure-vs-code/)) y la documentación de [Microsoft Entra Agent ID](https://learn.microsoft.com/en-us/entra/agent-id/)

### 1. Autenticación hacia Azure/Foundry (tu app → agente)

1. **API Keys:**
   - Simple: header con la key del recurso.
   - Riesgos: rotación manual, fuga en repos, sin identidad por usuario. Solo para prototipos.
2. **Microsoft Entra ID (recomendado):**
   - `DefaultAzureCredential` intenta en orden: variables de entorno → managed identity → Azure CLI → interactivo.
   ```python
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient

   project = AIProjectClient(
       endpoint=PROJECT_ENDPOINT,
       credential=DefaultAzureCredential()   # sin keys en el código
   )
   ```
   - En local usa tu sesión `az login`; en producción usa **Managed Identity** del App Service/Container App: cero secretos.
   - Acceso controlado por **RBAC** (roles como *Azure AI User* / *Azure AI Project Manager*).
3. **Managed Identity (system o user-assigned):**
   - Identidad administrada por Azure para tus recursos de cómputo; sin credenciales que guardar ni rotar. Es el estándar de producción.

### 2. Autenticación del agente hacia sus tools

1. **Conexiones del proyecto de Foundry:** guardas las credenciales (keys, connection strings) de servicios como AI Search o Storage como *connections* del proyecto; el agente las usa sin que aparezcan en el código.
2. **OpenAPI tools:** soportan esquemas de autenticación: anónimo, **API key** y **managed identity**, declarados en la definición de la tool.
3. **Servidores MCP:** pasan headers de autenticación (por ejemplo `Authorization: Bearer …`) configurados en la conexión; con Foundry IQ el acceso puede ser **permission-aware** (respeta permisos del usuario final sobre los documentos).
4. **On-Behalf-Of (OBO) / identidad del usuario:** para que el agente actúe con los permisos DEL USUARIO (por ejemplo, leer solo el SharePoint que ese usuario puede ver), el token del usuario fluye hacia la tool. Crítico para no convertir al agente en un "super-usuario" que filtra datos.

### 3. Identidad propia del agente: Microsoft Entra Agent ID
- Los agentes creados en Foundry reciben una **identidad propia en Microsoft Entra** (Agent ID): aparecen en el directorio como "no human identities", se les aplican políticas, se auditan y se les puede revocar acceso como a cualquier cuenta.
- Beneficios: gobernanza centralizada (¿cuántos agentes hay y qué pueden hacer?), mínimo privilegio por agente y trazabilidad de sus acciones.

### Buenas prácticas (checklist)
- [ ] Nada de keys hardcodeadas: `DefaultAzureCredential` + Managed Identity.
- [ ] Secretos inevitables → **Azure Key Vault**.
- [ ] RBAC de mínimo privilegio para usuarios Y agentes.
- [ ] Tools sensibles con identidad del usuario (OBO), no con identidad global.
- [ ] Rotación y auditoría: logs de quién/qué agente accedió a qué.
- [ ] Redes: Private Link para que endpoints no queden expuestos a internet.

### Regla mental para el examen
- "¿Sin gestionar credenciales en producción?" → **Managed Identity**.
- "¿Mismo código local y en la nube?" → **DefaultAzureCredential**.
- "¿El agente debe respetar los permisos del usuario sobre los datos?" → **identidad del usuario / permission-aware retrieval**.
- "¿Gobernar la identidad de los agentes como cuentas?" → **Entra Agent ID**.

### Beneficios
Una estrategia de identidad correcta hace que el agente sea auditable y contenible: sabes quién es, qué puede tocar y puedes apagarlo — exactamente lo que un auditor (y el examen) quieren escuchar.
