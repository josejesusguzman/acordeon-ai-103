# Casos exitosos reales de agentes de IA implementados en empresas

### Descripción
Los agentes de IA ya no son demos: hay despliegues productivos documentados públicamente por Microsoft y sus clientes. Estos casos sirven como referencia de arquitectura y como argumento de negocio. (Cifras según casos publicados por Microsoft; verifica las más recientes en [customers.microsoft.com](https://www.microsoft.com/en-us/customers/search?filters=product%3Aazure-ai-foundry).)

### Casos representativos

1. **Soporte al cliente — telecomunicaciones y aerolíneas:**
   - Patrón: agente conversacional con RAG sobre base de conocimiento + tools que consultan sistemas (estado de cuenta, vuelos) + escalamiento humano.
   - Ejemplos públicos: **Vodafone** (asistente TOBi evolucionado con Azure OpenAI, millones de interacciones), **Air India** (asistente AI.g sobre Azure OpenAI que absorbe la gran mayoría de consultas de clientes sin intervención humana).
   - Resultado típico: 60-90% de consultas resueltas sin agente humano y reducción de tiempos de espera.

2. **Productividad interna — banca y seguros:**
   - Patrón: copilotos internos con RAG sobre políticas/documentación, con permisos por usuario.
   - Ejemplos: bancos como **ABN AMRO** (asistente para empleados y clientes) y aseguradoras que usan agentes para resumir siniestros y pólizas.
   - Resultado: minutos ahorrados por consulta × miles de empleados al día.

3. **Ingeniería de software:**
   - Patrón: GitHub Copilot + agentes de revisión/refactor en el pipeline.
   - Microsoft reporta que una parte sustancial del código nuevo en proyectos que lo adoptan es asistido por IA; empresas como **Accenture** documentaron aumentos de velocidad de desarrollo en sus estudios con Copilot.

4. **Operaciones y back-office:**
   - Patrón: **multi-agente**: un agente clasifica documentos entrantes (facturas, contratos) con Document Intelligence/Content Understanding, otro valida contra ERP mediante tools, otro redacta la respuesta; orquestados con workflows.
   - Ejemplo: procesamiento de facturas y onboarding de proveedores en manufactura y retail.

5. **Salud:**
   - Patrón: agentes de documentación clínica (transcripción con Speech + resumen con LLM) tipo **DAX Copilot de Nuance/Microsoft**, usados por decenas de miles de médicos para reducir la carga de notas clínicas.

### Qué tienen en común los casos que funcionan

1. **Alcance acotado:** resuelven UN proceso medible (no "un asistente para todo").
2. **Datos propios bien conectados** (RAG con permisos) — el valor está en el conocimiento, no en el modelo.
3. **Humano en el loop** para casos de baja confianza o alto riesgo.
4. **Métricas de negocio definidas desde el día 1:** tasa de resolución, tiempo ahorrado, CSAT, costo por interacción.
5. **Despliegue iterativo:** piloto con un grupo → medición → expansión.

### Beneficios
Estudiar estos patrones te permite proponer arquitecturas ya validadas: casi cualquier caso empresarial es una combinación de RAG + tools + orquestación + human-in-the-loop, que es exactamente lo que la AI-103 enseña a construir.
