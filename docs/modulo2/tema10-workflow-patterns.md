# Workflow patterns en Microsoft Foundry: diferencias y cuándo usar cada uno

### Descripción
Los **workflows** de Microsoft Foundry orquestan agentes y otros componentes como un grafo de nodos (agentes, condiciones, loops, entrada humana) con variables compartidas. Foundry ofrece **patrones predefinidos**; el examen espera que sepas distinguirlos y elegir.

> [!NOTE]
> Módulo de referencia: [Build agent-driven workflows using Microsoft Foundry](https://learn.microsoft.com/es-es/training/modules/build-agent-workflows-microsoft-foundry/) (unidad *Identify Workflow Patterns*)

### Los 3 patrones predefinidos

1. **Sequential (secuencial):**
   - Camino fijo, paso a paso: cada nodo se ejecuta en orden y pasa su salida al siguiente.
   - Ideal para pipelines y procesos multi-etapa: validar entrada → enriquecer datos → generar respuesta final.
   - **Fortaleza:** predecible y fácil de razonar; el mejor punto de partida.

2. **Human-in-the-loop:**
   - El workflow **pausa** en puntos definidos para pedir aprobación o input humano; hace la pregunta, espera respuesta y reanuda según lo respondido.
   - Ideal para: aprobaciones, confirmaciones, contexto que solo una persona puede aportar, y escalamiento de casos de baja confianza.
   - **Fortaleza:** equilibra automatización con supervisión.

3. **Group chat:**
   - Orquestación dinámica entre múltiples agentes: el control cambia de agente según contexto, reglas o resultados intermedios; los agentes construyen sobre las salidas de los demás.
   - Ideal para: soporte multi-dominio, colaboración de especialistas, requests complejos sin camino fijo.
   - **Fortaleza:** flexibilidad; **costo:** menos predecible.

### Piezas con las que se construyen (conceptos de la unidad *Understand Workflows*)
- **Nodos:** agentes, If/Else (condiciones), For-Each (loops), entrada de usuario, fin.
- **Variables:** *system* (contexto de la conversación) y *local* (datos capturados durante la ejecución).
- **Structured outputs:** los agentes devuelven JSON tipado que las condiciones evalúan para enrutar.
- **Power Fx:** el lenguaje de expresiones que evalúa condiciones y manipula datos (ver tema siguiente).

### Cuándo usar cada patrón

| Escenario | Patrón |
|---|---|
| Proceso repetible con etapas claras (ETL conversacional, redacción→revisión→publicación) | Sequential |
| Aprobación de gastos, publicación de contenido, acciones irreversibles | Human-in-the-loop |
| Ticket que puede ser de facturación, técnico o ventas y requiere especialistas colaborando | Group chat |
| Lista de elementos a procesar uno por uno (tickets, facturas) | Sequential + For-Each |
| Ítems de baja confianza deben revisarse por un humano | Cualquiera + patrón de escalamiento (If confianza < umbral → humano) |

### Ejemplo típico de examen
"Un workflow procesa solicitudes de reembolso; si el monto > $500 o la confianza del agente < 0.8 debe aprobarlo un gerente; el resto se aprueba automático."
→ **Sequential + If/Else (Power Fx) + Human-in-the-loop** para la rama de aprobación.

### Beneficios
Los patrones dan un vocabulario de diseño: empiezas identificando si tu proceso es fijo (sequential), supervisado (human-in-the-loop) o colaborativo (group chat), y el lienzo de Foundry te da los nodos exactos para materializarlo sin código complejo.
