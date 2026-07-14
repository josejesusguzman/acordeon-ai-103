# Proceso básico de Azure AI Language: language → entities → PII

### Descripción
**Azure AI Language** es el servicio de análisis de texto "clásico" (pre-LLM, pero vigente y barato). Su flujo típico encadena tres análisis: detectar el **idioma**, extraer **entidades** y detectar/enmascarar **PII** (información personal identificable). Todo con la misma librería `azure-ai-textanalytics`.

> [!NOTE]
> Módulo de referencia: [Analyze text with Azure AI Language](https://learn.microsoft.com/es-es/training/modules/analyze-text-ai-language/)

### El pipeline básico

```text
Texto crudo ──> 1. Language detection (¿en qué idioma está?)
                     └─> 2. Entity recognition (¿qué personas/lugares/fechas menciona?)
                              └─> 3. PII detection (¿qué datos personales hay que proteger?)
```

1. **Language detection:** devuelve idioma (ISO 639-1) y score de confianza. Importante porque los demás análisis funcionan mejor si les pasas el idioma correcto.
2. **Named Entity Recognition (NER):** extrae entidades categorizadas: `Person`, `Location`, `Organization`, `DateTime`, `Quantity`, `Email`, `URL`… También existe **entity linking** (vincula "Venus" al artículo correcto de Wikipedia para desambiguar).
3. **PII detection:** caso especial de NER enfocado en datos sensibles: nombres, teléfonos, correos, números de tarjeta, cuentas, identificaciones. Devuelve las entidades **y** el texto con los datos enmascarados (`redactedText`), listo para almacenar logs sin exponer datos.

### Código completo del pipeline en Python

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential

client = TextAnalyticsClient(
    endpoint="https://<tu-recurso>.cognitiveservices.azure.com/",
    credential=AzureKeyCredential("<tu-key>")
)

docs = ["Hola, soy Ana García, mi correo es ana.garcia@mail.com y vivo en Monterrey."]

# 1) Idioma
idioma = client.detect_language(docs)[0]
print("Idioma:", idioma.primary_language.iso6391_name)   # es

# 2) Entidades
entidades = client.recognize_entities(docs, language="es")[0]
for e in entidades.entities:
    print(f"  {e.text} -> {e.category} ({e.confidence_score:.2f})")
# Ana García -> Person, Monterrey -> Location, ana.garcia@mail.com -> Email

# 3) PII
pii = client.recognize_pii_entities(docs, language="es")[0]
print("Texto enmascarado:", pii.redacted_text)
# "Hola, soy **********, mi correo es ******************* y vivo en Monterrey."
```

### Otras capacidades del mismo servicio (con la misma factura)
- **Sentiment analysis / opinion mining:** positivo/negativo/neutro por documento y por oración.
- **Key phrase extraction:** frases clave para indexar o etiquetar.
- **Summarization:** resumen extractivo/abstractivo de documentos y conversaciones.
- **CLU (Conversational Language Understanding):** intents y entities personalizados para bots.
- **Custom NER / custom text classification:** entrena categorías propias con tus etiquetas.
- **Question answering:** base de conocimiento de preguntas-respuestas.

### Por qué encadenar language → entities → PII (y no al revés)
- El idioma condiciona la calidad de NER y PII (pásalo como parámetro).
- NER te dice **qué hay** en el texto (valor analítico); PII te dice **qué debes proteger** (valor de compliance). Son análisis distintos aunque se parezcan.
- Enmascarar PII al inicio del pipeline te permite usar el texto en logs, analytics o prompts de LLM sin fugas de datos.

### Beneficios
Este pipeline es rápido, barato, determinista y auditable: la opción correcta cuando necesitas análisis masivo de texto con requisitos de privacidad, reservando los LLM para lo que realmente los necesita (ver siguiente tema).
