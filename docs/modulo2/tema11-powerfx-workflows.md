# Ejemplos de uso de Power Fx en workflows

### Descripción
**Power Fx** es el lenguaje low-code estilo Excel que actúa como "pegamento" de los workflows de Foundry: manipula datos, evalúa condiciones y controla el flujo sin código complejo. Aparece donde haya decisiones, asignación de variables o loops.

> [!NOTE]
> Unidad de referencia: [Apply Power Fx in Workflows](https://learn.microsoft.com/en-us/training/modules/build-agent-workflows-microsoft-foundry/6-apply-power-fx?pivots=text)

### Cómo funcionan las fórmulas
Una fórmula Power Fx es una expresión que evalúa a un valor y puede referenciar:
- **Variables de sistema (`System.`):** contexto del workflow/conversación — actividad actual, último mensaje, información del usuario.
- **Variables locales (`Local.`):** datos capturados o creados durante la ejecución, usables en nodos posteriores.

### Tabla de fórmulas de ejemplo (directo del módulo oficial)

| Propósito | Fórmula | Nota |
|---|---|---|
| Texto a mayúsculas | `Upper(Local.Input)` | Transforma un string |
| Texto a minúsculas | `Lower(Local.Input)` | |
| Longitud de un string | `Len(Local.Input)` | Número de caracteres |
| Chequeo condicional | `Local.Confidence > 0.8` | true/false, para nodos If/Else |
| Lógica If/Else | `If(Local.Confidence > 0.8, "Proceed", "Escalate")` | Devuelve uno de dos valores |
| Sumar lista | `Sum([10, 20, 30])` | |
| Sumar columna de tabla | `Sum(Local.ItemList, Amount)` | Suma la propiedad `Amount` de cada registro |
| Contar elementos | `Count(Local.ItemList)` | |
| ¿Está vacío? | `IsBlank(Local.Input)` | true si la variable está vacía |
| ¿Tabla vacía? | `IsEmpty(Local.ItemList)` | true si no hay registros |
| Iterar elementos | `ForAll(Local.ItemList, Upper(Name))` | Aplica fórmula a cada ítem |
| Concatenar texto | `Concatenate(Local.FirstName, " ", Local.LastName)` | Une strings |

### Dónde se usan en el workflow

1. **Nodos If/Else (puntos de decisión):** evalúan condiciones sobre variables o **structured outputs** de agentes. Ejemplo clásico — escalamiento por confianza:
   ```text
   If(Local.Confidence > 0.8, "Proceed", "Escalate")
   ```
   Si el agente clasificador devolvió `{"categoria": "reembolso", "confidence": 0.65}`, la rama "Escalate" lleva a un nodo human-in-the-loop.

2. **Nodos For-Each (loops):** iteran colecciones aplicando las mismas acciones a cada ítem — por ejemplo, procesar una lista de tickets sin duplicar nodos:
   ```text
   ForAll(Local.Tickets, ...)   # dentro del loop se usa el ítem actual
   ```

3. **Asignación de variables:** capturar y transformar datos entre nodos:
   ```text
   Set: Local.NombreCompleto = Concatenate(Local.Nombre, " ", Local.Apellido)
   Set: Local.TotalFacturas = Sum(Local.Facturas, Monto)
   ```

### Mini-escenarios prácticos

- **Validar entrada del usuario antes de llamar al agente:**
  `If(IsBlank(Local.Input), "PedirDatos", "ContinuarFlujo")`
- **Enrutar por categoría del structured output:**
  `If(Local.Categoria = "facturacion", "AgenteFacturas", Local.Categoria = "tecnico", "AgenteTecnico", "AgenteGeneral")`
- **Resumir montos para el reporte final:**
  `Concatenate("Procesadas ", Text(Count(Local.Items)), " solicitudes por $", Text(Sum(Local.Items, Amount)))`

### Beneficios
Power Fx permite que el workflow reaccione dinámicamente a inputs, salidas de agentes y datos almacenados con fórmulas legibles tipo Excel: lógica compleja, mantenible y sin desplegar código.
