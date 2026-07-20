# Manual del vibecoder: cómo programar efectivamente con IA (GitHub Copilot)

### Descripción
"Vibecoding" es programar delegando gran parte de la escritura de código a la IA. Hacerlo bien no es aceptar todo lo que sugiere el autocompletado: es dirigir a la IA como a un pair programmer junior con memoria infinita. Esta guía cubre **GitHub Copilot en VS Code al 100%**: autocompletado, los modos de chat (Ask, Edit, Agent, Plan), manejo de contexto, personalización del contexto del proyecto (instructions), skills, agentes personalizados, artefactos y el ecosistema alrededor (coding agent en la nube, CLI, MCP).

> [!NOTE]
> Referencias: [Documentación de agentes de VS Code](https://code.visualstudio.com/docs/agents/overview), [Custom instructions](https://code.visualstudio.com/docs/agent-customization/custom-instructions), [Agent Skills](https://code.visualstudio.com/docs/agent-customization/agent-skills), [Custom agents](https://code.visualstudio.com/docs/agent-customization/custom-agents) y ejemplos comunitarios en [github/awesome-copilot](https://github.com/github/awesome-copilot)

### 1. Autocompletado (code completions)

- **Ghost text:** sugerencias en línea mientras escribes. `Tab` acepta todo, `Ctrl+→` acepta palabra por palabra, `Alt+]` / `Alt+[` cambia entre sugerencias alternativas, `Esc` descarta.
- **Next Edit Suggestions (NES):** Copilot no solo completa donde está el cursor: predice la **siguiente edición** en otra parte del archivo (p. ej., renombraste un parámetro y te sugiere actualizar sus usos). Salta a la sugerencia con `Tab`.
- **De dónde saca contexto el autocompletado:** del archivo activo y de las **pestañas abiertas** relacionadas. Por eso conviene abrir los archivos relevantes (modelo + servicio + test) antes de escribir.
- **Guíalo con el propio código:** nombres descriptivos, firmas tipadas y comentarios-instrucción producen completions radicalmente mejores:

  ```python
  # Función que llama a Azure AI Language para extraer entidades PII
  # de un texto en español y devuelve una lista de dicts {tipo, texto}
  def extraer_pii(texto: str) -> list[dict]:
  ```

> [!IMPORTANT]
> Las *custom instructions* (sección 4) **NO aplican al autocompletado en línea**, solo al chat. Para el autocompletado, tu contexto es el código mismo: nombres, tipos, comentarios y archivos abiertos.

### 2. Los modos del chat: Ask, Edit, Agent y Plan

Se eligen en el desplegable de agentes del Chat (`Ctrl+Alt+I`); el **model picker** te deja además elegir el modelo (GPT, Claude, Gemini…) por conversación.

| Modo | Qué hace | Cuándo usarlo |
|---|---|---|
| **Ask** | Responde preguntas y explica; **no edita** archivos | Entender código, explorar opciones, stack traces, "¿qué hace este regex?" |
| **Edit** | Aplica ediciones multi-archivo que tú defines; revisas cada diff | Cambios acotados y dirigidos: "agrega manejo de errores a todas las llamadas HTTP" |
| **Agent** | Ciclo autónomo: planifica, edita, ejecuta terminal, corre pruebas, itera hasta terminar; pide **aprobaciones** para acciones sensibles | Tareas completas: "implementa el endpoint con sus pruebas" |
| **Plan** | Investiga el código con herramientas de **solo lectura** y produce un plan de implementación revisable; con un **handoff** pasas al modo Agent para ejecutarlo | Features o refactors grandes: planear primero, ejecutar después |

Complementos: **Inline chat** (`Ctrl+I`) para pedir cambios sin salir del editor; comandos rápidos `/explain`, `/fix`, `/tests`, `/doc`; **checkpoints** para volver a un punto anterior de la sesión si el agente se descarriló; y la vista **Review edits** para aceptar/rechazar cambios archivo por archivo.

### 3. Contexto: qué ve Copilot y cómo controlarlo por mensaje

Copilot incluye **implícitamente** el archivo activo y tu selección. Para todo lo demás, tú mandas:

- **`#` menciones (Add Context):** `#file` (archivo específico), `#selection`, `#codebase` (busca en TODO el workspace lo relevante a tu pregunta — la mención más poderosa), `#fetch` (contenido de una URL), `#terminalSelection`, `#changes` (tus cambios de git), `#testFailure`.
- **`@` participantes:** `@workspace` (preguntas sobre el proyecto completo), `@terminal` (comandos de shell), `@vscode` (configuración del editor), `@github` (issues, PRs, repos).
- **Arrastrar y soltar:** archivos, carpetas o **imágenes** (mockups, capturas de errores) directamente al chat.
- **Verifica qué usó:** cada respuesta incluye una sección **References** con los archivos e instructions que realmente consultó. Si no está tu archivo clave, méncionalo explícito.

Regla de oro: contexto **preciso** gana a contexto masivo. `#codebase` + 2 archivos clave supera a pegar medio proyecto.

### 4. Modificar el contexto permanente que Copilot tiene del proyecto

Aquí es donde dejas de repetir lo mismo en cada prompt. VS Code lee automáticamente estos archivos y los agrega a **todas** las peticiones de chat del workspace:

| Archivo | Alcance | Para qué |
|---|---|---|
| `.github/copilot-instructions.md` | Todo el workspace, siempre | Estándares del proyecto: stack, convenciones de nombres, arquitectura, manejo de errores |
| `AGENTS.md` (raíz o subcarpetas) | Todo el workspace; estándar multi-agente | Un solo archivo que reconocen Copilot, Claude Code y otros agentes |
| `.github/instructions/*.instructions.md` | Solo archivos que coincidan con su patrón `applyTo` | Reglas por lenguaje/framework/carpeta (frontend vs backend, tests, docs) |
| Instrucciones de organización (GitHub org) | Todos los repos de la org | Estándares corporativos compartidos |

- **Genera las instrucciones con IA:** escribe **`/init`** en el chat y Copilot analiza tu proyecto y genera el `copilot-instructions.md` a la medida. Con **`/create-instruction`** generas una regla puntual ("siempre tabs y comillas simples"), y si corriges al agente en una conversación puedes pedirle "extrae una instrucción de esto" para volver la corrección permanente.
- **Formato de `.instructions.md`:**

  ```markdown
  ---
  name: 'Python Standards'
  description: 'Convenciones para archivos Python'
  applyTo: '**/*.py'
  ---
  - Sigue PEP 8 y usa type hints en todas las firmas.
  - Docstrings en funciones públicas. Indentación de 4 espacios.
  ```

- **Prioridad cuando hay conflicto:** personales (usuario) > repositorio > organización.
- **Cómo escribirlas bien (guía oficial):** frases cortas y autocontenidas; incluye el **porqué** ("usa date-fns porque moment.js está deprecado"); muestra ejemplos de código preferido/evitado; omite lo que ya fuerza el linter.
- **Depuración:** clic derecho en el Chat → **Diagnostics** muestra qué instructions/skills/agents se cargaron y sus errores; la sección References de cada respuesta confirma cuáles se aplicaron.
- **Excluir contenido:** para que Copilot no lea archivos sensibles (llaves, datos), configura *content exclusions* en la configuración del repositorio/organización en GitHub.

### 5. Skills (Agent Skills)

Una **skill** es una carpeta con un `SKILL.md` (+ scripts, plantillas y recursos) que enseña a Copilot una **capacidad reutilizable**: "cómo probamos con Playwright", "cómo depuramos GitHub Actions", "nuestro proceso de deploy". Es un [estándar abierto](https://agentskills.io) portable entre VS Code, Copilot CLI y el coding agent en la nube.

- **Ubicación:** `.github/skills/<nombre>/SKILL.md` (proyecto) o `~/.copilot/skills/` (personal).
- **Formato mínimo:**

  ```markdown
  ---
  name: webapp-testing
  description: Guía para probar web apps con Playwright. Úsala al crear o correr pruebas de navegador.
  ---
  # Pasos
  1. Revisa la [plantilla de test](./test-template.js)
  2. Crea el test en `tests/` con localizadores por rol
  3. Corre `npx playwright test`
  ```

- **Carga progresiva (3 niveles):** Copilot solo lee nombre+descripción de todas las skills; carga el cuerpo del `SKILL.md` cuando la tarea coincide; y abre los recursos (scripts, ejemplos) solo si las instrucciones los referencian. Puedes instalar muchas skills sin quemar contexto.
- **Invocación:** automática (por relevancia de la descripción) o manual como slash command: `/webapp-testing para la página de login`. Controla ambas con `user-invocable` y `disable-model-invocation`.
- **`context: fork` (experimental):** ejecuta la skill en un subagente aparte y solo devuelve el resultado — ideal para skills que leen muchos archivos.
- **Créalas con IA:** `/create-skill` en el chat, o "crea una skill de cómo acabamos de depurar esto" para capturar un procedimiento que te funcionó.
- **Skill vs. instruction:** instruction = *reglas siempre activas* (estándares); skill = *capacidad bajo demanda* con scripts y recursos.

### 6. Agentes personalizados y subagentes

Un **custom agent** (`.agent.md` en `.github/agents/`) define una **persona** con sus propias instrucciones, herramientas permitidas y modelo: un *Planner* de solo lectura, un *Security Reviewer*, un *Implementer*.

```markdown
---
description: Genera un plan de implementación sin editar código
name: Planner
tools: ['search/codebase', 'search/usages', 'web/fetch']
handoffs:
  - label: Implementar plan
    agent: agent
    prompt: Implementa el plan anterior.
---
Estás en modo planeación. Genera un plan Markdown con: Overview,
Requirements, Implementation Steps y Testing. No edites código.
```

- **Handoffs:** botones al final de la respuesta que pasan al siguiente agente con prompt precargado (Plan → Implementar → Revisar). Flujos guiados con aprobación humana en cada paso.
- **Subagentes:** un agente coordinador puede delegar en otros (propiedad `agents`): un *Feature Builder* que usa un *Researcher* (solo lectura) y un *Implementer* (edición). Cada subagente trabaja en su propio contexto.
- **Herramientas mínimas:** dale a cada agente solo las tools que necesita (un revisor no necesita `edit`) — es el principio de menor privilegio aplicado a la IA.
- **Créalos con `/create-agent`** o compártelos a nivel organización de GitHub.

### 7. Artefactos, sesiones y control

- **Artifacts panel:** reúne los productos no-código de tus sesiones (planes, reportes, documentos generados) en un panel dedicado para revisarlos sin bucear en el historial del chat.
- **Checkpoints:** cada respuesta del agente crea un punto de restauración; si la iteración 4 arruinó lo de la 2, regresas.
- **Sesiones y Agents window:** administra varias sesiones de agente en paralelo (locales, CLI o en la nube) desde una sola ventana.
- **Aprobaciones:** el modo Agent pide permiso para comandos de terminal y acciones sensibles; puedes configurar listas de auto-aprobación — hazlo con cuidado.

### 8. El ecosistema completo (mapa rápido)

| Pieza | Qué aporta |
|---|---|
| **Prompt files** (`.prompt.md`) | Prompts reutilizables como slash commands: `/scaffold-component`, `/prep-pr` |
| **MCP servers** | Conectan al agente con tus herramientas y datos reales (BD, Jira, APIs) |
| **Hooks** | Comandos automáticos en puntos del ciclo del agente (formatear tras cada edit, bloquear comandos riesgosos) — guardrails deterministas |
| **Agent plugins** | Paquetes instalables de instructions + skills + agents + MCP desde un marketplace |
| **Copilot coding agent (nube)** | Le asignas un issue en GitHub.com y trabaja en segundo plano hasta abrir un PR que tú revisas |
| **Copilot CLI** | El agente en tu terminal, con las mismas skills e instructions |
| **Code review / commit messages** | Copilot revisa PRs y genera mensajes de commit (instrucciones configurables vía settings) |

Regla para elegir personalización: ¿regla siempre activa? → **instruction**. ¿Capacidad con pasos/scripts? → **skill**. ¿Rol con herramientas propias? → **custom agent**. ¿Prompt que repites? → **prompt file**. ¿Datos/herramientas externas? → **MCP**. ¿Garantía que no dependa del modelo? → **hook**.

### Reglas del buen vibecoder

1. **Piensa antes de pedir:** describe qué quieres con entradas, salidas y restricciones: lenguaje, framework, ejemplo de entrada/salida y qué NO hacer.
2. **Divide en pasos pequeños:** "hazme la app completa" produce lodo; "crea el modelo", luego "el endpoint", luego "las pruebas" produce software.
3. **Invierte en contexto permanente:** 30 minutos escribiendo `copilot-instructions.md` y 2-3 skills ahorran repetir lo mismo en cada prompt para siempre.
4. **NUNCA aceptes sin leer:** eres responsable del código. Revisa lógica, seguridad (secretos hardcodeados, inyección SQL) y licencias.
5. **Exige pruebas:** pide las pruebas unitarias (`/tests`) y córrelas. El test que falla es tu mejor detector de alucinación de código.
6. **Usa la IA para entender, no solo para producir:** `/explain`, "explícame este stack trace".
7. **Itera con errores reales:** pega el error completo (o usa `#terminalSelection`); es el prompt más efectivo que existe.
8. **Cuida los secretos:** nunca pegues keys en prompts; usa variables de entorno y configura content exclusions.
9. **Corrige una vez, institucionaliza siempre:** cada corrección que le hagas al agente conviértela en instruction o skill.

### Flujo de trabajo recomendado (tarea nueva)

```text
1. Escribe el QUÉ (criterios de aceptación) en un issue/comentario
2. Modo Plan: genera el plan de implementación y revísalo/edítalo
3. Handoff a modo Agent: ejecuta el plan paso a paso (con aprobaciones)
4. Revisa los diffs con Review edits; usa checkpoints si algo se descarrila
5. /tests + correr pruebas; pega fallos de vuelta al chat
6. Pide refactor + docs; commit con mensaje generado
7. Code review humano final (o Copilot code review + humano)
```

### Anti-patrones

- Aceptar 200 líneas generadas sin correrlas: deuda técnica instantánea.
- Pelear 10 iteraciones con la IA cuando leer la documentación tomaría 2 minutos.
- Dejar que la IA elija arquitectura sin darle restricciones del proyecto (para eso están las instructions).
- Copiar/pegar entre ChatGPT y el IDE cuando Copilot ya tiene el contexto del workspace (`#codebase`).
- Usar modo Agent con auto-aprobación total para "ir más rápido": es regalarle la terminal.
- Tener 40 skills e instructions contradictorias: audítalas (la extensión *Chat Customizations Evaluations* detecta contradicciones y ambigüedades).

### Beneficios
Bien usado, Copilot elimina el trabajo mecánico (boilerplate, pruebas, docs) y te deja el trabajo de ingeniería real: diseñar, decidir y verificar. La diferencia entre un vibecoder mediocre y uno bueno no es el prompt del momento: es el **contexto permanente** (instructions, skills, agentes) que construyó alrededor de su proyecto.
