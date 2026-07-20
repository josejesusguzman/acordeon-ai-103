# Casos exitosos reales de agentes de IA implementados en empresas

### Descripción
Los agentes de IA ya no son demos: hay despliegues productivos documentados públicamente por Microsoft y sus clientes. Estos casos sirven como referencia de arquitectura y como argumento de negocio. Cada caso incluye **el problema que enfrentaron, cómo lo resolvieron y qué resultados obtuvieron**. (Cifras según los casos publicados; verifica las más recientes en [customers.microsoft.com](https://www.microsoft.com/en-us/customers/search?filters=product%3Aazure-ai-foundry).)

### Casos específicos documentados

#### 1. Air India — AI.g: soporte al cliente a escala aerolínea

- **Problema:** tras la fusión con Vistara, el tráfico de pasajeros se duplicó y con él las consultas de soporte (~40,000 diarias). Escalar con call centers tradicionales era inviable en costo, y los tiempos de espera dañaban la experiencia en plena reconstrucción de la marca.
- **Solución:** construyeron **AI.g**, el primer agente virtual generativo de una aerolínea, sobre **Azure OpenAI** con **RAG sobre Azure AI Search** (políticas de equipaje, reservas, reembolsos), más Azure AI Speech y Vision para una plataforma multimodal. Casos complejos o de baja confianza escalan a agentes humanos. Salió a producción en **6 meses**.
- **Resultados:** **97% de los millones de interacciones se resuelven sin humano**; el volumen de llamadas se mantuvo plano pese a duplicarse los pasajeros; ahorros de millones de dólares anuales.
- **Lección:** el patrón RAG + escalamiento humano aguanta escala real; el valor estuvo en conectar el conocimiento propio, no en el modelo per se.

#### 2. Fujitsu — agente de propuestas de venta (multi-agente)

- **Problema:** los equipos comerciales gastaban enormes cantidades de tiempo armando propuestas: el conocimiento estaba disperso en repositorios, productos y especialistas, y las propuestas quedaban desactualizadas o inconsistentes.
- **Solución:** un agente sobre **Azure AI Agent Service** que interpreta la petición del vendedor, integra datos de múltiples fuentes y genera propuestas actualizadas. El núcleo es **Fujitsu Kozuchi Composite AI con Semantic Kernel orquestando múltiples agentes especializados**. Destacaron la facilidad de integración entre Agent Service y **Azure AI Search** desde el portal.
- **Resultados:** **+67% de productividad** en creación de propuestas; los vendedores reinvierten el tiempo en el cliente.
- **Lección:** para tareas compuestas (buscar + redactar + validar) funciona mejor un **orquestador de agentes especializados** que un solo agente gigante.

#### 3. NTT DATA — agentes de datos empresariales y sales coach

- **Problema:** consultas rutinarias de servicio al cliente consumían al equipo, y el acceso a insights de datos empresariales requería especialistas — cada respuesta a un cliente tardaba demasiado.
- **Solución:** agentes conversacionales con **Azure AI Foundry Agent Service + Microsoft Fabric** que consultan datos empresariales en lenguaje natural y responden **según el rol y función del usuario** (permisos por usuario). Además, un **sales coach multi-agente** que prepara reuniones, redacta propuestas y descubre insights por cliente.
- **Resultados:** automatización de consultas rutinarias, mejores tiempos de respuesta y **50% más velocidad de salida al mercado**.
- **Lección:** el control de acceso por rol no es un extra: es lo que hace viable poner datos empresariales detrás de un agente.

#### 4. Dow — agentes de facturación de fletes (autónomo + conversacional)

- **Problema:** Dow recibe **más de 100,000 facturas de transporte en PDF al año** (20% de sus facturas de envío). La verificación manual no alcanzaba para detectar errores de cobro: sobrecargos incorrectos pasaban inadvertidos entre miles de documentos.
- **Solución:** **dos agentes complementarios**: uno **autónomo** (Copilot Studio) que monitorea el correo entrante, extrae y estructura los datos de los PDFs y marca anomalías de facturación en un dashboard; y el **Freight Agent**, conversacional, con el que los empleados "dialogan con los datos" para investigar cada anomalía.
- **Resultados:** detectaron sobrecargos de $30,000 donde la tarifa típica era $5,000 y cargos multiplicados por 10 en varios envíos; proyectan **millones de dólares de ahorro** al escalarlo globalmente.
- **Lección:** el patrón ganador de back-office es **agente autónomo que detecta + humano que decide + agente conversacional que ayuda a investigar** — human-in-the-loop bien diseñado.

#### 5. Salud — Dragon Copilot (documentación clínica)

- **Problema:** el burnout médico por carga documental: horas de notas clínicas después de cada turno, tiempo robado a los pacientes.
- **Solución:** **Dragon Copilot** (Nuance/Microsoft): escucha ambiental de la consulta + voz a texto + resúmenes generativos integrados al flujo clínico, con sugerencias de códigos ICD-10, cartas de referencia y resúmenes post-visita — sin cambiar de sistema.
- **Resultados:** **más de 100,000 médicos y enfermeras** lo usan a diario atendiendo a millones de pacientes al mes; una enfermera reporta ~**2 horas de documentación ahorradas por turno de 12 horas**.
- **Lección:** en dominios regulados el agente gana cuando se **integra al flujo de trabajo existente** (el expediente clínico) en vez de ser "otro chat".

#### 6. Vodafone — TOBi/SuperTOBi: evolucionar un bot legacy

- **Problema:** su chatbot clásico basado en intents entendía mal las preguntas reales de los clientes: demasiados "no te entendí" y escalamientos innecesarios.
- **Solución:** evolucionaron TOBi a **SuperTOBi con Azure OpenAI**: comprensión de lenguaje natural real + RAG sobre su base de conocimiento, desplegado por país/idioma.
- **Resultados (según lo publicado por Microsoft):** resolución sin humano en la mayoría de las consultas y mejora de tiempos de respuesta y satisfacción en los mercados donde se desplegó.
- **Lección:** no partías de cero: un bot legacy con tráfico real es el mejor dataset de evaluación para su reemplazo generativo.

### Problemas comunes y cómo los resolvieron

| Problema enfrentado | Solución aplicada en los casos |
|---|---|
| Alucinaciones ante clientes | RAG con datos propios + escalamiento humano en baja confianza (Air India, Vodafone) |
| Conocimiento disperso en silos | Orquestación multi-agente sobre múltiples fuentes (Fujitsu, NTT DATA) |
| Datos sensibles con permisos distintos por usuario | Respuestas condicionadas al rol vía control de acceso (NTT DATA) |
| Volumen imposible de revisar manualmente | Agente autónomo que filtra + dashboard + humano que decide (Dow) |
| Adopción por usuarios expertos escépticos | Integrarse al flujo existente en lugar de imponer una app nueva (Dragon Copilot) |
| Escalar sin explotar costos | Automatizar el % masivo simple y reservar humanos/modelos caros para lo complejo (todos) |

### Qué tienen en común los casos que funcionan

1. **Alcance acotado:** resuelven UN proceso medible (no "un asistente para todo").
2. **Datos propios bien conectados** (RAG con permisos) — el valor está en el conocimiento, no en el modelo.
3. **Humano en el loop** para casos de baja confianza o alto riesgo.
4. **Métricas de negocio definidas desde el día 1:** tasa de resolución, tiempo ahorrado, CSAT, costo por interacción.
5. **Despliegue iterativo:** piloto con un grupo → medición → expansión.

> [!NOTE]
> Fuentes: [Air India](https://www.microsoft.com/en/customers/story/26047-air-india-azure-openai-in-foundry-models), [Fujitsu](https://www.microsoft.com/en/customers/story/21885-fujitsu-azure-ai-foundry) y su [caso técnico con Semantic Kernel](https://devblogs.microsoft.com/semantic-kernel/customer-case-study-fujitsu-kozuchi-ai-agent-powered-by-semantic-kernel/), [NTT DATA](https://www.microsoft.com/en/customers/story/23654-ntt-data-azure-ai-agent-service), [Dow](https://www.microsoft.com/en-us/worklab/ai-impact-at-dow-copilot-identifies-millions-in-cost-savings) y [Dragon Copilot](https://www.microsoft.com/en-us/microsoft-cloud/blog/healthcare/2025/03/03/meet-microsoft-dragon-copilot-your-new-ai-assistant-for-clinical-workflow/).

### Beneficios
Estudiar estos patrones te permite proponer arquitecturas ya validadas: casi cualquier caso empresarial es una combinación de RAG + tools + orquestación + human-in-the-loop, que es exactamente lo que la AI-103 enseña a construir.
