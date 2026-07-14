# Manual del vibecoder: cómo programar efectivamente con IA (GitHub Copilot)

### Descripción
"Vibecoding" es programar delegando gran parte de la escritura de código a la IA. Hacerlo bien no es aceptar todo lo que sugiere el autocompletado: es dirigir a la IA como a un pair programmer junior con memoria infinita. Esta guía usa **GitHub Copilot** como herramienta base.

### Herramientas y modos de Copilot

1. **Code completions:** sugerencias en línea mientras escribes. Acepta con `Tab`, siguiente sugerencia con `Alt+]`.
2. **Copilot Chat:** conversación en el IDE. Comandos útiles: `/explain`, `/fix`, `/tests`, `/doc`.
3. **Modo Edit:** cambios multi-archivo dirigidos ("agrega manejo de errores a todas las llamadas HTTP de este proyecto").
4. **Modo Agent / coding agent:** le das una tarea completa (issue) y la ejecuta: crea archivos, corre pruebas, itera. Tú revisas el resultado como si fuera un PR.
5. **Contexto con `@workspace`, `#file`:** referencia explícita a archivos o al proyecto para respuestas fundamentadas en TU código.

### Reglas del buen vibecoder

1. **Piensa antes de pedir:** describe qué quieres, con entradas, salidas y restricciones. Un buen prompt de código incluye: lenguaje, framework, ejemplo de entrada/salida y qué NO hacer.
2. **Divide en pasos pequeños:** "hazme la app completa" produce lodo; "crea el modelo Pydantic para X", luego "el endpoint", luego "las pruebas" produce software.
3. **Da contexto en el código:** nombres descriptivos, docstrings y comentarios-instrucción; Copilot completa mucho mejor con buen contexto:
   ```python
   # Función que llama a Azure AI Language para extraer entidades PII
   # de un texto en español y devuelve una lista de dicts {tipo, texto}
   def extraer_pii(texto: str) -> list[dict]:
   ```
4. **NUNCA aceptes sin leer:** eres responsable del código. Revisa lógica, seguridad (secretos hardcodeados, inyección SQL) y licencias.
5. **Exige pruebas:** pide a la IA las pruebas unitarias (`/tests`) y córrelas. El test que falla es tu mejor detector de alucinación de código.
6. **Usa la IA para entender, no solo para producir:** `/explain` sobre código ajeno, "explícame este stack trace", "¿qué hace este regex?".
7. **Itera con errores reales:** pega el error completo en el chat; es el prompt más efectivo que existe.
8. **Documenta decisiones:** pide que genere README/docstrings al final de cada bloque de trabajo.
9. **Cuida los secretos:** nunca pegues keys/connection strings en prompts; usa variables de entorno y `DefaultAzureCredential`.

### Flujo de trabajo recomendado (tarea nueva)

```text
1. Escribe en un comentario/issue QUÉ quieres (criterios de aceptación)
2. Pide a Copilot un plan (sin código) y revísalo
3. Genera código paso a paso, corriendo cada paso
4. Genera pruebas y ejecútalas
5. Pide refactor + docs
6. Code review humano final
```

### Anti-patrones
- Aceptar 200 líneas generadas sin correrlas: deuda técnica instantánea.
- Pelear 10 iteraciones con la IA cuando leer la documentación tomaría 2 minutos.
- Dejar que la IA elija arquitectura sin darle restricciones del proyecto.
- Copiar/pegar entre ChatGPT y el IDE cuando Copilot ya tiene el contexto del workspace.

### Beneficios
Bien usado, Copilot elimina el trabajo mecánico (boilerplate, pruebas, docs) y te deja el trabajo de ingeniería real: diseñar, decidir y verificar. Mal usado, produce código que nadie entiende — incluida la IA que lo escribió.
