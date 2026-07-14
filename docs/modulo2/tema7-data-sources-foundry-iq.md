# Cuándo y cómo elegir data sources con Foundry IQ

### Descripción
Tu knowledge base vale lo que valen sus datos. Foundry IQ soporta **seis tipos de data source**; elegir bien depende de dónde vive tu información y cómo necesitas accederla (indexada, directa o en tiempo real).

> [!NOTE]
> Unidad de referencia: [Configure data sources for knowledge bases](https://learn.microsoft.com/en-us/training/modules/introduction-foundry-iq/4-data-requirements?pivots=text)

### Los 6 data sources

| Data source | Tipo de acceso | Ideal para |
|---|---|---|
| **Azure AI Search Index** | Indexado | Búsqueda empresarial con pipelines personalizados ya existentes |
| **Azure Blob Storage** | Directo | Archivos/documentos en Azure Storage (PDF, DOCX, TXT, MD, HTML) |
| **Web** | Tiempo real | Información pública y actual vía Bing |
| **SharePoint (Remote)** | Tiempo real | Contenido vivo de SharePoint con gobernanza Microsoft 365 |
| **SharePoint (Indexed)** | Indexado | Búsqueda avanzada sobre SharePoint con pipelines custom |
| **OneLake** | Directo | Datos no estructurados en Microsoft Fabric |

### Detalles que caen en examen

1. **Azure AI Search Index:** úsalo si ya invertiste en AI Search: aprovechas ranking semántico, scoring profiles, facetas y multi-idioma. Es la opción con capacidades de búsqueda más sofisticadas.
2. **Blob Storage:** camino más directo de archivo→conocimiento, sin mantener un índice propio; organiza contenedores por tema/nivel de acceso.
3. **Web:** grounding con Bing para eventos recientes, precios, disponibilidad. **Advertencia:** menos control sobre las fuentes; si la exactitud verificable es crítica, prefiere fuentes indexadas propias. Se puede combinar como fuente suplementaria.
4. **SharePoint Remote vs Indexed:**

   | Característica | Remote | Indexed |
   |---|---|---|
   | Acceso | Consulta en tiempo real | Índice preprocesado |
   | Velocidad | Depende de SharePoint | Más rápida |
   | Mantenimiento | Sin índice que mantener | Requiere actualizar índice |
   | Búsqueda avanzada | Limitada | Todo el poder de AI Search |
   | Frescura | Siempre actual | Según programación de indexado |
   | Permisos | Respeta permisos de SharePoint automáticamente | Se configuran al indexar |

   - **Remote** = setup simple, contenido siempre vigente, permisos M365 automáticos.
   - **Indexed** = respuestas más rápidas, analizadores custom, enriquecimiento con AI, combinar con otras fuentes.
5. **OneLake:** si tu organización usa Fabric: reportes BI, documentación de datasets, hallazgos analíticos accesibles conversacionalmente.

### Guía de decisión rápida

| Si tus datos están… | Y necesitas… | Elige |
|---|---|---|
| En SharePoint | Setup simple, siempre actual | SharePoint Remote |
| En SharePoint | Búsqueda avanzada, pipelines custom | SharePoint Indexed |
| Archivos en Azure | Acceso directo a archivos | Blob Storage |
| En Microsoft Fabric | Contenido del lakehouse | OneLake |
| Ya indexados | Aprovechar inversión en AI Search | AI Search Index |
| Públicos y cambiantes | Contenido web en tiempo real | Web |

> [!TIP]
> Puedes **combinar múltiples fuentes** en una sola knowledge base: por ejemplo, SharePoint interno como fuente principal + Web como complemento para información actual.

### Beneficios
Elegir el data source correcto equilibra frescura, velocidad, capacidades de búsqueda y gobernanza — y como Foundry IQ unifica todo detrás de la knowledge base, puedes cambiar o sumar fuentes sin tocar a los agentes.
