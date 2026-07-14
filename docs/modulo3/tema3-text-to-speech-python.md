# Ejercicio avanzado: uso de audio en Python con la API Text to Speech

### Descripción
Azure Speech (Foundry Tools) ofrece la API **Text to speech** para síntesis de voz. En la práctica casi todo se hace con el **Speech SDK** (`azure-cognitiveservices-speech`), cuyo patrón espejea al de reconocimiento de voz. Este tema desarrolla el flujo completo y variantes avanzadas de manejo de audio en Python.

> [!NOTE]
> Unidad de referencia: [Use the Text to Speech API](https://learn.microsoft.com/en-us/training/modules/create-speech-enabled-apps/4-text-to-speech?pivots=text)

### El patrón oficial (5 pasos)

1. **`SpeechConfig`:** encapsula la conexión al recurso (endpoint/región + key).
2. **`AudioConfig` (opcional):** define el destino del audio: altavoz por defecto, archivo, o `None` para procesar el stream directamente.
3. **`SpeechSynthesizer`:** cliente proxy de la API, creado con los dos configs.
4. **Llamar métodos:** `speak_text_async()` convierte texto a voz.
5. **Procesar el resultado (`SpeechSynthesisResult`):** propiedades `audio_data`, `reason`, `result_id`. Si `reason == SynthesizingAudioCompleted`, la síntesis fue exitosa.

### Código base (del módulo oficial)

```python
import azure.cognitiveservices.speech as speechsdk

speech_config = speechsdk.SpeechConfig(subscription=KEY, endpoint=ENDPOINT)
audio_config = speechsdk.audio.AudioOutputConfig(use_default_speaker=True)

speech_synthesizer = speechsdk.SpeechSynthesizer(
    speech_config=speech_config, audio_config=audio_config)

text = "My voice is my password!"
result = speech_synthesizer.speak_text_async(text).get()

if result.reason == speechsdk.ResultReason.SynthesizingAudioCompleted:
    print(f"Speech synthesized for text [{text}]")
elif result.reason == speechsdk.ResultReason.Canceled:
    details = result.cancellation_details
    print(f"Canceled: {details.reason}")
    if details.reason == speechsdk.CancellationReason.Error:
        print(f"Error: {details.error_details}")
```

### Variantes avanzadas de audio

1. **Elegir voz neural** (antes de crear el synthesizer):
   ```python
   speech_config.speech_synthesis_voice_name = "es-MX-DaliaNeural"
   ```
2. **Guardar a archivo WAV:**
   ```python
   audio_config = speechsdk.audio.AudioOutputConfig(filename="salida.wav")
   ```
3. **Obtener los bytes en memoria (sin altavoz ni archivo):** pasa `audio_config=None` y usa `result.audio_data`:
   ```python
   synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config, audio_config=None)
   result = synthesizer.speak_text_async("Hola mundo").get()
   with open("hola.mp3", "wb") as f:
       f.write(result.audio_data)          # bytes crudos, útiles para APIs web
   ```
4. **Formato de salida (MP3 comprimido en vez de WAV):**
   ```python
   speech_config.set_speech_synthesis_output_format(
       speechsdk.SpeechSynthesisOutputFormat.Audio16Khz32KBitRateMonoMp3)
   ```
5. **Streaming con eventos** (audio conforme se genera, para apps en tiempo real):
   ```python
   def on_audio(evt):
       # evt.result.audio_data llega en chunks
       pass
   synthesizer.synthesizing.connect(on_audio)
   ```
6. **El ciclo completo voz↔voz:** combina `SpeechRecognizer` (speech-to-text: `recognize_once_async()`) + tu lógica/LLM + `SpeechSynthesizer` (text-to-speech) para un asistente hablado.

### Alternativa generativa
Los modelos de audio generativo (gpt-4o-audio / gpt-realtime, módulo [Develop generative AI audio apps](https://learn.microsoft.com/es-es/training/modules/develop-generative-ai-audio-apps/)) también generan voz, pero el Speech SDK sigue siendo la opción para TTS controlado, barato y con voces corporativas consistentes (más SSML, ver tema 5).

### Beneficios
Dominar `SpeechConfig → AudioConfig → SpeechSynthesizer → Result` te habilita cualquier escenario de audio en Python: altavoz, archivo, bytes en memoria o streaming — el mismo patrón, distinto `AudioConfig`.
