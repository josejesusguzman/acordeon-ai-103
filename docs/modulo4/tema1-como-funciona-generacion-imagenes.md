# Cómo funciona la creación de imágenes con IA

### Descripción
Los generadores de imágenes (DALL·E, GPT-image, FLUX, Stable Diffusion) transforman texto en imágenes. Entender el mecanismo (difusión + comprensión de lenguaje) explica tanto sus capacidades como sus fallas típicas (manos raras, texto ilegible) y cómo usarlos por API en Azure.

> [!NOTE]
> Módulo de referencia: [Generate images with AI](https://learn.microsoft.com/es-es/training/modules/generate-images-azure-openai/)

### El mecanismo, en 4 ideas

1. **Entrenamiento con pares imagen-texto:** el modelo aprende de millones de imágenes con descripciones, construyendo un espacio donde texto e imágenes comparten representación (embeddings): "gato con sombrero" queda cerca de las imágenes de gatos con sombrero.
2. **Difusión (diffusion):** la técnica dominante. En entrenamiento, se agrega ruido progresivo a las imágenes hasta volverlas estática pura; el modelo aprende a **revertir** ese proceso. Al generar: parte de ruido aleatorio y lo "des-ruida" paso a paso guiado por tu prompt, hasta que emerge la imagen.
3. **Guía por texto (conditioning):** en cada paso de des-ruido, el modelo consulta la representación de tu prompt para decidir qué detalles reforzar. Por eso prompts más específicos → resultados más controlados.
4. **Autoregresivo/multimodal (GPT-image):** los modelos más recientes integran la generación en el LLM multimodal: entienden instrucciones complejas, corrigen texto dentro de la imagen y permiten edición conversacional ("ahora hazlo de noche").

```text
Prompt ──> encoder de texto ──┐
                              ▼
Ruido aleatorio ──> [des-ruido iterativo guiado] ──> imagen final
```

### Uso por API en Azure (Python)

```python
import requests

response = openai_client.images.generate(
    model="dall-e-3",                 # tu deployment (o gpt-image-1)
    prompt="Un ajolote astronauta flotando sobre la CDMX, estilo acuarela",
    n=1,
    size="1024x1024",
    quality="hd",
    style="vivid"
)
image_url = response.data[0].url
img = requests.get(image_url).content
open("ajolote.png", "wb").write(img)
```

Parámetros clave: `size` (1024x1024, 1792x1024…), `quality` (`standard`/`hd`), `style` (`vivid`/`natural`), `n` (número de imágenes).

### Por qué falla en lo que falla
- **Manos/dedos y texto:** la difusión optimiza plausibilidad visual global, no consistencia simbólica; contar dedos o dibujar letras exactas es "detalle fino" difícil (los modelos multimodales recientes lo mejoran mucho).
- **Negaciones:** "sin sombrero" a menudo pone sombrero: el prompt introduce el concepto. Mejor describe lo que SÍ quieres.
- **Composiciones exactas** ("el tercer libro a la izquierda dice…"): el control espacial fino es limitado; itera o edita.

### Seguridad integrada
En Azure, la generación pasa por filtros de contenido (Content Safety) en el prompt y en la imagen resultante, y las políticas bloquean contenido dañino, celebridades, marcas — parte del argumento empresarial de usar Azure (ver tema 2).

### Beneficios
Entender la difusión te hace mejor usuario: sabes que el modelo "esculpe desde el ruido guiado por tu texto", así que prompts concretos, estilos explícitos e iteración son la palanca — no la suerte.
