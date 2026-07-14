# Live voice: agentes de voz en tiempo real (Voice Live API)

### Descripción
El patrón clásico STT → LLM → TTS funciona, pero se siente "robótico": hay pausas entre cada etapa y no puedes interrumpir al asistente. **Voice Live** (Azure AI Speech / Foundry) resuelve esto con una única API de **conversación hablada en tiempo real**: audio entra y sale en streaming por WebSocket, con detección de turnos e interrupciones.

> [!NOTE]
> Módulo de referencia: [Develop a voice live agent](https://learn.microsoft.com/es-es/training/modules/develop-voice-live-agent/)

### Por qué el patrón clásico se queda corto

```text
Clásico:  audio ──STT──> texto ──LLM──> texto ──TTS──> audio   (latencia sumada, sin interrupciones)
Voice Live: audio ⇄ [API única en streaming: escucha + entiende + responde] ⇄ audio
```

### Aspectos Clave

1. **Conversación full-duplex:** el usuario puede hablar MIENTRAS el asistente responde; el sistema detecta la interrupción (*barge-in*) y corta la respuesta. Esto es imposible con el pipeline por etapas.
2. **Detección de turnos (turn detection / VAD):** la API decide cuándo el usuario terminó de hablar (semantic VAD / server VAD), eliminando el clásico "presiona para hablar".
3. **Baja latencia:** al no serializar STT→LLM→TTS, el tiempo de respuesta baja a niveles de conversación natural (sub-segundo percibido).
4. **Extras integrados:** cancelación de ruido, supresión de eco, voces neuronales (incluidas voces personalizadas) y transcripción simultánea de la conversación.
5. **Modelos:** puedes usar modelos nativos de voz (tipo gpt-realtime) o combinar el motor de voz con tu agente/modelo de Foundry; también se integra con **agentes existentes** (Foundry Agent) para darles "boca y oídos".

### Esqueleto en Python (WebSocket asíncrono)

```python
# pip install azure-ai-voicelive aiohttp
import asyncio
from azure.ai.voicelive.aio import connect
from azure.ai.voicelive.models import (
    RequestSession, ServerVad, AzureStandardVoice, Modality
)

async def main():
    async with connect(
        endpoint="wss://<tu-recurso>.services.ai.azure.com/voice-live/realtime",
        credential=credential,
        model="gpt-realtime"
    ) as conn:
        # Configura la sesión: voz, instrucciones y detección de turnos
        session = RequestSession(
            modalities=[Modality.TEXT, Modality.AUDIO],
            instructions="Eres un asistente amable; responde breve y en español.",
            voice=AzureStandardVoice(name="es-MX-DaliaNeural"),
            turn_detection=ServerVad(
                threshold=0.5, silence_duration_ms=500)
        )
        await conn.session.update(session=session)

        # Loop de eventos: audio del micrófono -> conn / audio de conn -> bocinas
        async for event in conn:
            ...  # manejar eventos: session.updated, response.audio.delta,
                 # input_audio_buffer.speech_started (interrupción), etc.

asyncio.run(main())
```

### Manejo de eventos clave
- `SESSION_UPDATED`: la sesión está lista → empezar a capturar micrófono.
- `RESPONSE_AUDIO_DELTA`: chunks de audio del asistente → reproducir.
- `INPUT_AUDIO_BUFFER_SPEECH_STARTED`: el usuario interrumpió → detener reproducción y limpiar buffer.
- `ERROR`: manejar reconexión.

### Cuándo usar Voice Live vs Speech SDK clásico

| Escenario | Usa |
|---|---|
| Asistente conversacional hablado, call center, kiosko | **Voice Live** |
| Leer notificaciones/documentos en voz alta | Speech SDK (TTS) |
| Transcribir archivos/reuniones (batch) | Speech SDK / Fast transcription |
| Necesitas interrupciones y turnos naturales | **Voice Live** |
| Control total del pipeline y post-procesamiento del texto | Clásico STT→LLM→TTS |

### Beneficios
Voice Live convierte a tus agentes en interlocutores de verdad: hablan, escuchan y se dejan interrumpir con latencia natural — la pieza que hace viables asistentes telefónicos y de voz en producción.
