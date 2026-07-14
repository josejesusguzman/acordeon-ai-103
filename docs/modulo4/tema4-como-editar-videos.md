# Cómo editar videos generados con IA (Sora en Foundry)

### Descripción
Generar el clip es la mitad del trabajo; la otra mitad es **editarlo**: extender, recortar, mezclar, mantener consistencia entre tomas y ensamblar el resultado final. Sora en Microsoft Foundry ofrece operaciones de edición sobre video generado, y el resto se completa con flujo de edición tradicional.

> [!NOTE]
> Módulo de referencia: [Generate video with Sora in Microsoft Foundry](https://learn.microsoft.com/es-es/training/modules/generate-video-with-foundry/)

### Operaciones de edición con Sora

1. **Text-to-video:** generación base a partir del prompt (duración, resolución y aspect ratio configurables).
2. **Image-to-video:** anima una imagen fija de referencia — útil para partir de un keyframe aprobado (producto, personaje) y garantizar la primera toma.
3. **Video-to-video / remix:** transforma un video existente con un nuevo prompt (cambiar estilo, clima, hora del día) conservando el movimiento base.
4. **Extensión (continuación):** alarga un clip generándole continuación coherente; encadenando extensiones logras secuencias más largas que el límite por generación.
5. **Recomposición (re-cut):** genera variaciones de un segmento que no funcionó sin regenerar todo el clip.

### Flujo de trabajo API (Python, patrón de video jobs)

```python
# Patrón: crear job -> poll de estado -> descargar resultado
import requests, time

# 1. Crear el job de generación/edición
job = requests.post(
    f"{endpoint}/openai/v1/video/generations/jobs?api-version=preview",
    headers={"api-key": KEY},
    json={
        "model": "sora",
        "prompt": "Dron ascendiendo sobre una ciudad futurista al amanecer, cámara lenta",
        "n_seconds": 10,
        "height": 1080,
        "width": 1920
    }
).json()

# 2. Poll hasta que termine (succeeded/failed)
while True:
    status = requests.get(
        f"{endpoint}/openai/v1/video/generations/jobs/{job['id']}?api-version=preview",
        headers={"api-key": KEY}).json()
    if status["status"] in ("succeeded", "failed"):
        break
    time.sleep(5)

# 3. Descargar el video final
gen_id = status["generations"][0]["id"]
video = requests.get(
    f"{endpoint}/openai/v1/video/generations/{gen_id}/content/video?api-version=preview",
    headers={"api-key": KEY}).content
open("clip.mp4", "wb").write(video)
```
> La generación de video es **asíncrona** (jobs): crear → consultar estado → descargar. Esto cae bien en preguntas de examen.

### Ensamblaje final (fuera de la IA)
Para el corte final usa un editor tradicional o ffmpeg:
```bash
# unir clips generados
ffmpeg -f concat -safe 0 -i lista.txt -c copy final.mp4
# recortar segundos 2 a 8 de un clip
ffmpeg -i clip.mp4 -ss 00:00:02 -to 00:00:08 -c copy recorte.mp4
```
La IA genera tomas; el ritmo, música, transiciones y color siguen siendo edición clásica.

### Buenas prácticas
1. **Guion de tomas primero:** lista de clips con prompt por toma; genera y aprueba toma por toma.
2. **Keyframes con image-to-video** cuando la identidad visual importa (producto/marca).
3. **Descripciones idénticas de personajes/objetos** entre clips para consistencia.
4. **Versiona prompts y seeds** junto al proyecto: son tu "material de rodaje".
5. **Deja margen de edición:** genera 1-2 s extra por clip para poder cortar en el ensamblaje.

### Beneficios
Tratar a Sora como "cámara + segunda unidad" y no como editor final produce resultados profesionales: la IA fabrica tomas infinitas baratas y tú mantienes el control narrativo en la edición.
