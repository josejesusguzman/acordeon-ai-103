# Tema Complementario 10: Contenerización y despliegue de apps de IA (Docker, Container Apps, AKS)

### Descripción
Tu agente no vive en el playground: vive en un contenedor detrás de una API. Este tema cubre el camino DevOps para empaquetar y desplegar aplicaciones LLM en Azure, y cómo elegir entre App Service, Container Apps y AKS.

### Aspectos Clave

1. **Dockerfile de referencia para una API de agente (FastAPI):**
   ```dockerfile
   FROM python:3.12-slim
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install --no-cache-dir -r requirements.txt
   COPY . .
   EXPOSE 8000
   # usuario no-root: buena práctica de seguridad
   RUN useradd -m appuser && chown -R appuser /app
   USER appuser
   CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
   ```
   Claves: imagen slim, dependencias cacheadas antes del código, sin secretos en la imagen (van por variables de entorno/managed identity).

2. **Particularidades de las apps LLM en contenedores:**
   - **Streaming:** usa respuestas SSE/chunked y verifica que el ingress no las bufferice.
   - **Timeouts largos:** una corrida de agente puede tardar 30-120 s; ajusta timeouts de ingress y health probes separadas (liveness rápida, readiness real).
   - **Estado fuera del contenedor:** historial/threads en Cosmos DB o Redis, nunca en memoria del pod (los pods mueren).
   - **Async:** usa clientes async (`AsyncAzureOpenAI`) para no bloquear workers esperando al modelo.

3. **¿Dónde desplegar? — la decisión:**

   | Servicio | Cuándo |
   |---|---|
   | **Azure App Service** | App web/API simple, equipo sin experiencia en contenedores |
   | **Azure Container Apps (ACA)** | El default moderno: serverless, escala a cero, KEDA, Dapr, sin administrar clúster |
   | **AKS (Kubernetes)** | Microservicios complejos, control total, GPUs propias, equipos con expertise K8s |
   | **Azure Functions** | Tools/webhooks event-driven de corta duración |

   Regla práctica: **empieza en Container Apps**; migra a AKS solo cuando tengas una razón concreta (no antes).

4. **Identidad y secretos en el despliegue:**
   - **Managed Identity** del Container App con rol RBAC hacia Foundry/OpenAI → `DefaultAzureCredential()` funciona sin ninguna key.
   - Secretos inevitables → Key Vault referenciado, no variables en texto plano.
   - Redes: Private Endpoints hacia los servicios de IA para tráfico fuera de internet.

5. **Escalado para cargas LLM:** el cuello de botella no es tu CPU, es la **cuota TPM del modelo**. Escalar pods sin subir cuota solo multiplica errores 429. Diseña: rate limiting propio, colas (Service Bus) para cargas batch, y auto-scaling por requests concurrentes (KEDA), no por CPU.

6. **CI/CD completo (GitHub Actions):**
   ```yaml
   # esqueleto: build -> push ACR -> deploy ACA
   - uses: azure/login@v2
     with: { client-id: ..., tenant-id: ..., subscription-id: ... }  # OIDC, sin passwords
   - run: az acr build -t agente:${{ github.sha }} -r miacr .
   - run: az containerapp update -n mi-agente -g mi-rg \
            --image miacr.azurecr.io/agente:${{ github.sha }}
   ```
   Con revisiones de Container Apps obtienes **blue/green y canary** nativos (dividir tráfico entre revisiones).

### Checklist de producción
- [ ] Imagen slim, non-root, escaneada (Defender for Cloud/Trivy)
- [ ] Managed identity, cero keys en el contenedor
- [ ] Streaming y timeouts verificados end-to-end
- [ ] Estado en Cosmos/Redis, pods sin estado
- [ ] Manejo de 429 con retry/backoff y rate limiting propio
- [ ] OTel exportando a Application Insights
- [ ] Revisiones canary + rollback probado

### Fuentes para profundizar
- [Azure Container Apps](https://learn.microsoft.com/es-es/azure/container-apps/overview)
- [Elegir servicio de cómputo en Azure](https://learn.microsoft.com/es-es/azure/architecture/guide/technology-choices/compute-decision-tree)

### Beneficios
Este es el puente entre "AI Engineer" y "DevOps": el mismo agente que funciona en tu laptop queda empaquetado, seguro, observable y escalable — y desplegable un viernes sin miedo gracias a canary + rollback.
