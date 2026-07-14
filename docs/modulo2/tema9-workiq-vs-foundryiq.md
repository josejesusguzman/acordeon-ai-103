# Diferencia entre Work IQ y Foundry IQ

### Descripción
Microsoft agrupa bajo el nombre "IQ" varias capas de contexto para agentes: **Work IQ**, **Foundry IQ**, **Fabric IQ** (y Web IQ). No compiten entre sí: cada una aporta un tipo de contexto distinto, y Foundry IQ actúa como el punto de integración desde el que un agente puede consumirlas.

> [!NOTE]
> Referencia: [What is Foundry IQ?](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/what-is-foundry-iq) y [Foundry IQ FAQ](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/foundry-iq-faq)

### Aspectos Clave

1. **Work IQ — contexto de CÓMO trabaja tu organización (Microsoft 365):**
   - Capa de inteligencia sobre los datos de trabajo de Microsoft 365: correos, calendario, mensajes de Teams, archivos, reuniones y patrones de colaboración.
   - Vive dentro del **límite de confianza de Microsoft 365** y mantiene un modelo semántico continuo de la organización: quién trabaja con quién, en qué proyectos, con qué documentos.
   - Es lo que alimenta a Microsoft 365 Copilot y a los agentes que necesitan contexto organizacional ("prepara el resumen para MI reunión de mañana con el equipo X").

2. **Foundry IQ — capa de CONOCIMIENTO empresarial para agentes:**
   - Plataforma administrada de knowledge bases (sobre Azure AI Search) que conecta contenido: documentos, políticas, manuales, FAQs, contratos, wikis en Blob, SharePoint, OneLake, Web e índices.
   - Su motor de **retrieval agéntico** planifica queries, selecciona fuentes, busca en paralelo, aplica permisos y devuelve respuestas fundamentadas con citas.
   - Es la pieza que usas al construir TUS agentes en Microsoft Foundry (vía MCP).

3. **La familia completa (para no confundirse en el examen):**

| Capa | Tipo de contexto | Fuente | Pregunta que responde |
|---|---|---|---|
| **Work IQ** | Organizacional / colaboración | Microsoft 365 (correo, Teams, reuniones, archivos) | "¿Cómo trabaja esta gente y en qué andan?" |
| **Foundry IQ** | Conocimiento / contenido | Blob, SharePoint, OneLake, Web, AI Search | "¿Qué dicen nuestros documentos y políticas?" |
| **Fabric IQ** | Datos de negocio (semántica analítica) | Microsoft Fabric / OneLake | "¿Qué dicen nuestras métricas y tablas?" |
| Web IQ | Señales externas en vivo | Web/Bing | "¿Qué está pasando afuera ahora?" |

4. **Cómo se relacionan:** al configurar un agente en Foundry, Foundry IQ es el punto de integración: puedes registrar como fuentes de conocimiento a Work IQ (contexto organizacional), Fabric IQ (datos de negocio) y la Web, y su motor de retrieval se encarga de consultar lo necesario con permisos y citas.

### Regla mental
- ¿El agente necesita saber de **correos, reuniones, Teams, gente**? → **Work IQ**.
- ¿El agente necesita **documentos, políticas, manuales, contenido indexado**? → **Foundry IQ**.
- ¿Datos tabulares/analíticos con semántica de negocio? → **Fabric IQ**.

### Beneficios
Distinguir las capas evita arquitecturas equivocadas: no indexes correos en una knowledge base (eso es Work IQ) ni pidas a Copilot que sirva tu manual de producto a clientes externos (eso es Foundry IQ). Cada contexto en su capa, y Foundry IQ como pegamento.

Sources: [What is Foundry IQ - Microsoft Learn](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/what-is-foundry-iq), [Making Sense of Microsoft's AI Strategy - James Serra](https://www.jamesserra.com/archive/2026/02/making-sense-of-microsofts-ai-strategy-work-iq-fabric-iq-foundry-iq/)
