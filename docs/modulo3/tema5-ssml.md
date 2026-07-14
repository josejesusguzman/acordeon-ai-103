# SSML: Speech Synthesis Markup Language a fondo

### Descripción
El Speech SDK acepta texto plano, pero también **SSML**, una sintaxis XML que describe CÓMO debe sonar la voz sintetizada: estilo, pausas, pronunciación, prosodia y hasta varias voces en un mismo audio. Es la diferencia entre un lector monótono y un locutor.

> [!NOTE]
> Unidad de referencia: [Use Speech Synthesis Markup Language](https://learn.microsoft.com/en-us/training/modules/create-speech-enabled-apps/6-speech-synthesis-markup?pivots=text) y [documentación de SSML](https://learn.microsoft.com/es-es/azure/ai-services/speech-service/speech-synthesis-markup)

### Qué puedes controlar con SSML

1. **Estilo de habla** (voces neuronales): "excited", "cheerful", "sad", "newscast", "customerservice"…
2. **Pausas y silencios:** `<break>` con fuerza (`weak`/`strong`) o tiempo exacto (`time="500ms"`).
3. **Fonemas:** pronunciación fonética exacta, p. ej. que "SQL" se diga "sequel".
4. **Prosodia:** tono (pitch), timbre y velocidad (rate) con `<prosody>`.
5. **Reglas "say-as":** interpretar un string como fecha, hora, teléfono, moneda, deletreo…
6. **Audio grabado:** insertar audios pregrabados (`<audio>`) o simular ambiente.
7. **Múltiples voces:** un diálogo con varias `<voice>` en el mismo documento.

### Ejemplo oficial (diálogo entre dos voces)

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis"
                     xmlns:mstts="https://www.w3.org/2001/mstts" xml:lang="en-US">
    <voice name="en-US-AriaNeural">
        <mstts:express-as style="cheerful">
          I say tomato
        </mstts:express-as>
    </voice>
    <voice name="en-US-GuyNeural">
        I say <phoneme alphabet="sapi" ph="t ao m ae t ow"> tomato </phoneme>.
        <break strength="weak"/>Lets call the whole thing off!
    </voice>
</speak>
```
Resultado: Aria dice su línea *alegre*; Guy pronuncia "tomato" como *tom-ah-toe*, pausa breve, y remata la frase.

### Cómo enviarlo desde Python

```python
ssml = """
<speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis'
       xmlns:mstts='https://www.w3.org/2001/mstts' xml:lang='es-MX'>
  <voice name='es-MX-DaliaNeural'>
    <mstts:express-as style='cheerful'>
      ¡Hola! Tu pedido <say-as interpret-as='characters'>PED-0042</say-as>
      llegará el <say-as interpret-as='date' format='dmy'>15-08-2026</say-as>.
    </mstts:express-as>
    <break time='400ms'/>
    <prosody rate='-10%' pitch='+5%'>¿Puedo ayudarte con algo más?</prosody>
  </voice>
</speak>
"""
result = speech_synthesizer.speak_ssml_async(ssml).get()
```
> Nota: se usa `speak_ssml_async()` en lugar de `speak_text_async()`.

### Chuleta de etiquetas

| Etiqueta | Para qué | Ejemplo |
|---|---|---|
| `<voice>` | Elegir voz (puede haber varias) | `<voice name="es-ES-ElviraNeural">` |
| `<mstts:express-as>` | Estilo emocional | `style="excited"` |
| `<break>` | Pausa | `<break time="500ms"/>` |
| `<prosody>` | Tono/velocidad/volumen | `rate="+20%" pitch="-2st"` |
| `<phoneme>` | Pronunciación fonética | `ph="t ao m ae t ow"` |
| `<say-as>` | Interpretar formato | `interpret-as="telephone"` |
| `<audio>` | Insertar audio grabado | `src="jingle.wav"` |
| `<lang>` | Cambiar idioma puntual | `xml:lang="en-US"` |

### Casos de uso típicos
- **IVR/call center:** números de teléfono, folios y fechas dichos correctamente (`say-as`).
- **Marca sonora:** misma voz, estilo y ritmo en todos los mensajes de la empresa.
- **Contenido educativo/noticias:** estilos `newscast`/`narration` y pausas didácticas.
- **Términos técnicos y nombres propios:** `phoneme` para no destrozar "Azure" o apellidos.

### Beneficios
SSML te da control de dirección de voz sin re-grabar nada: el mismo texto puede sonar alegre, pausado, bilingüe o corporativo cambiando etiquetas — esencial para experiencias de voz profesionales.
