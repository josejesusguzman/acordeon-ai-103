# Tema Complementario 3: LLMOps / GenAIOps — DevOps para aplicaciones de IA generativa

### Descripción
LLMOps (o GenAIOps) es aplicar disciplina DevOps a apps con LLM. La diferencia con MLOps clásico: no entrenas el modelo, pero versionas **prompts**, evalúas salidas no deterministas y monitoreas calidad además de uptime.

### Aspectos Clave

1. **El prompt es código:**
   - Versiona prompts/instrucciones en Git junto a la app (nunca hardcodeados sin control).
   - Cada cambio de prompt = un release: pasa por PR, evaluación y despliegue gradual.
   - Guarda prompts como archivos separados (YAML/Jinja) con metadatos: modelo objetivo, versión, autor, fecha.

2. **Los cuatro artefactos versionables:** prompt + modelo (nombre Y versión del deployment) + configuración (temperature, tools) + datos (índice/knowledge base). Un cambio en cualquiera puede cambiar el comportamiento: registra los cuatro en cada release.

3. **Evaluación como test suite:**
   - Dataset de evaluación (50-500 casos reales) en el repo, creciendo con cada bug.
   - En CI: correr evaluadores (groundedness, relevance, task adherence) y comparar contra el baseline; si baja del umbral, el pipeline falla.
   ```yaml
   # GitHub Actions (esqueleto)
   - name: Evaluate agent
     run: python evaluate.py --dataset eval/casos.jsonl --min-groundedness 4.0
   ```

4. **Entornos y despliegue gradual:**
   - dev → staging → prod con proyectos/recursos de Foundry separados (IaC con Bicep/Terraform).
   - **Shadow deployment** (la versión nueva responde en paralelo sin servir a usuarios) y **canary** (5% del tráfico) antes del 100%.
   - Rollback = volver al prompt/modelo anterior; tenlo automatizado.

5. **Monitoreo en producción (las 3 capas):**
   - **Sistema:** latencia, errores, tokens/costo (Application Insights + OpenTelemetry).
   - **Calidad:** evaluación continua sobre muestras de tráfico real + feedback de usuarios (👍/👎).
   - **Seguridad:** tasa de bloqueos de content filter, intentos de jailbreak, drift de temas.

6. **Gestión del cambio de modelo:** los modelos se retiran (deprecations) y las versiones nuevas cambian comportamiento. Proceso: leer calendario de retirement de Azure OpenAI → evaluar la nueva versión contra tu dataset → migrar con canary. Nunca "auto-update" a ciegas en producción.

7. **Herramientas del ecosistema:**
   - **Azure:** Foundry (evaluaciones, tracing), Application Insights, GitHub Actions/Azure DevOps.
   - **Open source:** promptfoo (evaluación de prompts en CI), MLflow (tracking de experimentos LLM), LangSmith/Langfuse (observabilidad).

### Ciclo completo

```text
Idea → prompt/código en rama → PR con evaluación automática
     → merge → deploy a staging (IaC) → evaluación + red team
     → canary en prod → monitoreo (sistema/calidad/seguridad)
     → feedback y fallos → nuevos casos al dataset → repetir
```

### Fuentes para profundizar
- [GenAIOps en Microsoft Learn](https://learn.microsoft.com/es-es/azure/ai-foundry/concepts/observability)
- [promptfoo](https://www.promptfoo.dev/) · [MLflow LLM tracking](https://mlflow.org/docs/latest/llms/)

### Beneficios
LLMOps es lo que diferencia "tenemos un bot" de "operamos un producto de IA": puedes cambiar modelo o prompt un martes por la tarde con confianza, porque el pipeline te protege de regresiones.
