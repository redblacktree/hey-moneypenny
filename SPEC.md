# hey-moneypenny — Voice Assistant for OpenClaw

## Overview
A macOS voice assistant that runs on a Mac mini, listens for multiple wake words and routes speech to the corresponding OpenClaw agent. For example, "Hey Moneypenny" routes to Moneypenny and "Hey Sheila" routes to Sheila. Each agent gets its own voice channel/session.

## Hardware Requirements
- Mac mini (Apple Silicon) — always-on
- USB microphone (e.g., HyperX SoloCast)
- Built-in speaker (or external speaker)

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                  hey-moneypenny                      │
│                                                      │
│  ┌──────────┐   ┌──────────┐   ┌────────────────┐  │
│  │  Wake     │──▶│  Audio   │──▶│  Whisper STT   │  │
│  │  Word     │   │  Capture │   │  (local/API)   │  │
│  │  Detect   │   │          │   │                │  │
│  └──────────┘   └──────────┘   └───────┬────────┘  │
│       ▲                                 │           │
│       │                                 ▼           │
│  ┌──────────┐                  ┌────────────────┐  │
│  │  Audio   │◀─────────────────│  OpenClaw      │  │
│  │  Playback│   TTS response   │  Gateway API   │  │
│  │  (speak) │                  │  (WebSocket)   │  │
│  └──────────┘                  └────────────────┘  │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │  Status LED / Menu Bar UI (optional)          │   │
│  │  - Listening indicator                        │   │
│  │  - Processing indicator                       │   │
│  │  - Muted/unmuted toggle                       │   │
│  └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

## Components

### 1. Wake Word Detection (Multi-Agent)
- **Engine**: [openWakeWord](https://github.com/dscripka/openWakeWord) — fully open-source, no vendor dependency, no API key required
  - Train custom wake words locally using synthetic speech (TTS) + noise augmentation
  - No limit on number of wake words — add new agents anytime
  - Supports simultaneous multi-model detection — load multiple `.tflite`/`.onnx` models and predict on each frame
  - Built-in Silero VAD integration for reducing false positives
  - Runs efficiently on Apple Silicon (~15-20 models simultaneously on a Raspberry Pi 3 single core)
  - Python library: `pip install openwakeword`
  - Training: use the included Colab notebook or local training script to generate models from synthetic speech samples
- **Training workflow for new wake words**:
  1. Generate synthetic speech samples of the phrase using TTS (e.g., OpenAI TTS, various voices/speeds)
  2. Augment with background noise, room impulse responses
  3. Train a small neural net model (~1 hour on Colab or local)
  4. Export `.onnx` model, add to config
- **Requirements**: <5% CPU idle, <500ms detection latency
- **Routing**: Each wake word model maps to a specific OpenClaw agent/session (see config below)

### 2. Audio Capture
- Capture audio from USB mic via system default input device
- Use Voice Activity Detection (VAD) after wake word to detect when user stops speaking
- **Silence threshold**: ~1.5 seconds of silence = end of utterance
- **Max utterance**: 30 seconds (prevent runaway recordings)
- **Audio format**: 16kHz mono WAV (Whisper's preferred format)
- Library: `sounddevice` (Python) or AVFoundation (Swift)

### 3. Speech-to-Text (STT)
- **Primary**: OpenAI Whisper API (`/v1/audio/transcriptions`)
  - Fast, accurate, handles accents well
  - Cost: ~$0.006/min
  - Requires network
- **Fallback**: Local Whisper via `whisper.cpp` or `/opt/homebrew/bin/whisper`
  - Already installed on the Mac mini
  - Slower but works offline
- Config toggle to choose local vs API

### 4. OpenClaw Integration
- Send transcribed text to OpenClaw gateway as a message
- **Option A**: WebSocket to gateway at `ws://127.0.0.1:18789` (direct, real-time)
- **Option B**: HTTP API if available
- **Option C**: Write to a named pipe / file that OpenClaw watches
- Need to receive the response text back for TTS
- Should create a dedicated "voice" channel/session in OpenClaw

### 5. Text-to-Speech (TTS)
- **Primary**: OpenAI TTS API (alloy/nova/shimmer voices)
  - Natural sounding, fast
  - Cost: ~$0.015/1K chars
- **Fallback**: macOS `say` command (free, instant, robotic)
- **Stretch**: ElevenLabs for premium voice quality
- Play through system default output device (Mac mini speaker)
- Library: `afplay` (macOS native) or `sounddevice`

### 6. Status / UI (optional, low priority)
- macOS menu bar icon showing state:
  - 🟢 Listening for wake word
  - 🟡 Processing speech
  - 🔴 Muted / error
- Click to mute/unmute
- Could be a simple SwiftUI menu bar app or Python with rumps

## Tech Stack Decision

### Recommended: Python
- **Why**: Fastest to build, best library ecosystem for audio/ML
  - `openwakeword` — wake word detection (open-source, locally trained)
  - `sounddevice` — audio capture
  - `openai` — Whisper + TTS APIs
  - `websocket-client` or `httpx` — OpenClaw gateway
  - `numpy` — audio processing
  - `rumps` — menu bar (optional)
- **Tradeoff**: Slightly higher resource usage than native Swift
- **Runtime**: Python 3.11+ via Homebrew, managed with `uv` or `pip`

### Alternative: Swift
- **Why**: Native macOS, lower resource usage, better AVFoundation integration
- **Tradeoff**: Slower to develop, fewer audio ML libraries
- **When**: If Python prototype proves too resource-heavy

## Configuration

```yaml
# hey-moneypenny.yaml

# Multi-agent wake word configuration
agents:
  moneypenny:
    wake_word:
      model_path: ./models/hey_moneypenny.onnx
      sensitivity: 0.5
    openclaw:
      gateway_url: ws://127.0.0.1:18789
      session: voice-moneypenny  # dedicated session for this agent
    tts:
      engine: openai
      voice: nova
  sheila:
    wake_word:
      model_path: ./models/hey_sheila.onnx
      sensitivity: 0.5
    openclaw:
      gateway_url: ws://127.0.0.1:18789
      session: voice-sheila
    tts:
      engine: openai
      voice: shimmer  # different voice per agent

wake_word:
  engine: openwakeword
  vad_threshold: 0.5  # Silero VAD filter (0-1, reduces false positives)

audio:
  input_device: null  # null = system default
  sample_rate: 16000
  silence_threshold_sec: 1.5
  max_utterance_sec: 30

stt:
  engine: openai  # or "local"
  model: whisper-1
  language: en

tts:
  # defaults (can be overridden per agent above)
  engine: openai
  voice: nova
  speed: 1.0

openclaw:
  # defaults (can be overridden per agent above)
  gateway_url: ws://127.0.0.1:18789

ui:
  menu_bar: true
  sounds:
    wake: ./sounds/wake.wav      # confirmation chime when wake word detected
    error: ./sounds/error.wav
```

## User Flow

1. Mac mini is always on, `hey-moneypenny` runs as a LaunchAgent
2. User says a wake word — **"Hey Moneypenny"** or **"Hey Sheila"**
3. openWakeWord identifies **which** wake word triggered
4. 🔔 Short chime plays (confirms wake word detected)
5. System captures speech until 1.5s silence
6. Audio sent to Whisper for transcription
7. Transcribed text routed to the **correct agent's** OpenClaw session
8. OpenClaw processes, returns response text
9. Response converted to speech via the **agent's configured TTS voice**
10. Audio plays through Mac mini speaker
11. Return to step 1 (listening for all wake words)

## Error Handling
- Network down → fall back to local Whisper + macOS `say`
- Mic disconnected → alert via Telegram, retry every 30s
- OpenClaw gateway down → alert via Telegram, queue messages
- Wake word false positive → short utterance (<1s silence) gets discarded

## Security
- All processing is local-first (wake word detection never leaves device)
- Audio is never stored permanently — transcribe and discard
- OpenClaw gateway is localhost only
- No cloud wake word processing (openWakeWord runs entirely on-device)

## Deployment
- Install as macOS LaunchAgent (`~/Library/LaunchAgents/ai.openclaw.hey-moneypenny.plist`)
- Auto-start on login
- Auto-restart on crash
- Logs to `/tmp/hey-moneypenny/`

## Open Questions
- [ ] Does OpenClaw have a voice/audio channel API, or do we need to use an existing channel (e.g., a virtual "voice" session)?
- [ ] openWakeWord model training — validate quality of locally-trained models for "Hey Moneypenny" and "Hey Sheila"
- [ ] Should responses also be sent as iMessage/Telegram in addition to spoken? (probably yes, for history)
- [ ] Barge-in support — can user interrupt a long spoken response? (V2)
