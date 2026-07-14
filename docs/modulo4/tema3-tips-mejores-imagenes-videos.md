# Tips para generar mejores imágenes y/o videos

### Descripción
La calidad de una imagen o video generado depende ~80% del prompt y ~20% de los parámetros. Estas técnicas aplican a DALL·E/GPT-image y a Sora (video) en Microsoft Foundry.

> [!NOTE]
> Módulos de referencia: [Generate images](https://learn.microsoft.com/es-es/training/modules/generate-images-azure-openai/) y [Generate video with Sora in Foundry](https://learn.microsoft.com/es-es/training/modules/generate-video-with-foundry/)

### Anatomía de un buen prompt visual

```text
[SUJETO] + [ACCIÓN/POSE] + [ENTORNO] + [ESTILO] + [ILUMINACIÓN] + [CÁMARA/COMPOSICIÓN]
```
Ejemplo pobre: "un perro".
Ejemplo bueno: "Un golden retriever corriendo por una playa al atardecer, estilo fotografía profesional, luz cálida dorada, lente 85mm f/1.8, poca profundidad de campo".

### Tips para imágenes

1. **Sé específico con el estilo:** "acuarela", "render 3D estilo Pixar", "fotografía editorial", "pixel art". El estilo es la instrucción que más cambia el resultado.
2. **Describe lo que SÍ quieres**, no lo que no: las negaciones ("sin gente") tienden a introducir el concepto. Reformula en positivo ("calle vacía").
3. **Ilumina la escena con palabras:** "golden hour", "luz de neón", "iluminación de estudio suave" — la luz define el mood.
4. **Composición y cámara:** "primer plano", "vista aérea", "gran angular", "simetría central" dan control fotográfico.
5. **Itera en serie, no en paralelo:** cambia UNA cosa por iteración para saber qué funcionó; con GPT-image aprovecha la edición conversacional ("misma imagen pero de noche").
6. **Parámetros:** `quality="hd"` para detalle fino, `style="natural"` para realismo sobrio vs `"vivid"` para dramatismo; tamaño horizontal (1792x1024) para paisajes/banners.
7. **Texto dentro de la imagen:** ponlo entre comillas y corto ("un letrero que dice 'ABIERTO'"); los modelos recientes lo manejan mejor, pero sigue siendo el punto débil.

### Tips para video (Sora)

1. **Piensa como director:** describe la TOMA: "plano medio, cámara lenta, dolly hacia adelante, día lluvioso". El movimiento de cámara es parte del prompt.
2. **Una escena por clip:** los cortes de escena dentro de un mismo prompt confunden al modelo; genera clips separados y edítalos después.
3. **Describe el movimiento del sujeto:** "el dron asciende revelando la ciudad" da coherencia temporal; los videos sin acción descrita tienden a "respirar" estáticamente.
4. **Duración y resolución mínimas viables:** clips cortos (5-10 s) tienen mejor consistencia; genera corto, selecciona y luego extiende/re-genera.
5. **Consistencia de personajes:** reutiliza descripciones EXACTAS del personaje entre clips ("mujer de chamarra roja y casco blanco") para minimizar variaciones.
6. **Física y manos:** evita interacciones físicas complejas (malabares, multitudes); son los puntos débiles actuales de la generación de video.

### Flujo de trabajo recomendado
```text
1. Borrador con prompt base (calidad estándar, barato)
2. Elegir el mejor resultado
3. Refinar prompt (estilo, luz, cámara) sobre ese ganador
4. Generar versión final en alta calidad
5. (Video) Ensamblar clips en un editor tradicional
```

### Costo y responsabilidad
- Cada iteración cuesta: refina en calidad baja, finaliza en alta.
- Los filtros de contenido aplican a prompt Y resultado; evita personas reales identificables, marcas y estilos de artistas vivos nombrados — además de ética, te ahorra bloqueos.

### Beneficios
Prompts estructurados (sujeto-acción-entorno-estilo-luz-cámara) + iteración disciplinada convierten la generación visual de lotería en proceso repetible con calidad profesional.
