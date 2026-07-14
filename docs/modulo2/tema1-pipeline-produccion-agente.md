# Pipeline hasta producción de un agente de IA con Azure

### Descripción
Llevar un agente del playground a producción es un pipeline con etapas bien definidas: desarrollo → evaluación → seguridad → despliegue → observabilidad. Aquí está el mapa completo con las piezas de Azure para cada etapa.

> [!NOTE]
> Rutas de referencia: [Desarrollo de agentes de IA en Azure](https://learn.microsoft.com/es-es/training/paths/develop-ai-agents-azure/) y el módulo [Deploy and govern agentic AI solutions](https://learn.microsoft.com/es-es/training/paths/deploy-govern-agentic-ai-solutions-azure/)

### Etapas del pipeline

1. **Desarrollo local:**
   - Proyecto de Foundry + VS Code. Define el agente (modelo, instrucciones, tools) con `azure-ai-projects`/`azure-ai-agents` o Microsoft Agent Framework.
   - Versiona TODO en Git: instrucciones (prompts) incluidas — el prompt es código.

2. **Datos y conocimiento:**
   - Conecta conocimiento con Foundry IQ / Azure AI Search (RAG) y herramientas (custom tools, MCP, OpenAPI, Logic Apps).

3. **Evaluación (gate de calidad):**
   - Datasets de prueba + evaluadores (`azure-ai-evaluation`): relevance, groundedness, task adherence, tool call accuracy.
   - Evaluaciones de seguridad y red teaming automatizado.
   - Criterio: sin pasar umbrales definidos, el pipeline NO avanza.

4. **CI/CD (GitHub Actions / Azure DevOps):**
   ```yaml
   # esqueleto conceptual de GitHub Actions
   jobs:
     test:      # pruebas unitarias del código y tools
     evaluate:  # corre evaluaciones batch contra el agente en un entorno de prueba
     deploy:    # publica el agente/app si evaluate pasa umbrales
   ```
   - Infraestructura como código (Bicep/Terraform) para recursos de Foundry, Search, Storage.
   - Entornos separados: dev → staging → prod, con proyectos de Foundry distintos.

5. **Despliegue del agente:**
   - **Agente hospedado en Foundry (Agent Service):** Azure aloja el runtime; lo invocas por API/SDK. Lo más simple.
   - **App propia:** contenedor en Azure Container Apps / AKS / App Service que usa el SDK. Máximo control.
   - **Canales:** API REST, Microsoft 365 Copilot / Teams (vía Agent Store), web app.

6. **Seguridad y gobernanza:**
   - Identidad del agente con **Microsoft Entra Agent ID**, RBAC de mínimo privilegio, secretos en Key Vault, redes privadas (Private Link).
   - Guardrails de contenido en el modelo (Content Safety) + límites de uso.

7. **Observabilidad y mejora continua:**
   - **Tracing** con Application Insights / OpenTelemetry: cada run, tool call y tokens.
   - Monitoreo de métricas operativas (latencia, errores, costo) y de calidad (evaluaciones continuas sobre tráfico real).
   - Feedback de usuarios → nuevos casos de prueba → repetir ciclo.

### Diagrama mental

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

### Checklist de producción
- [ ] Prompt e instrucciones versionados
- [ ] Evaluaciones automatizadas con umbrales
- [ ] Autenticación sin secretos (managed identity)
- [ ] Filtros de contenido configurados
- [ ] Tracing habilitado
- [ ] Plan de rollback y de respuesta a incidentes

### Beneficios
Tratar al agente como software (CI/CD + evaluaciones como pruebas) es lo que separa un demo de una solución operable: puedes cambiar modelo, prompt o tools con confianza porque el pipeline detecta regresiones antes que tus usuarios.
