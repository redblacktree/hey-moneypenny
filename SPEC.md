# hey-moneypenny — Voice Assistant for OpenClaw

## Overview
A macOS voice assistant that runs on a Mac mini, listens for the wake word "Hey Moneypenny," transcribes speech, routes it through OpenClaw, and speaks the response back through the speaker.

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

### 1. Wake Word Detection
- **Primary option**: [Porcupine by Picovoice](https://picovoice.ai/platform/porcupine/) — supports custom wake words, low CPU, runs locally
  - Free tier: 3 custom wake words, unlimited on-device
  - Train "Hey Moneypenny" via their console
  - Python SDK or Swift SDK available
- **Fallback option**: Whisper-based VAD + keyword spotting (higher CPU, more flexible)
- **Requirements**: <5% CPU idle, <500ms detection latency

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
  - `pvporcupine` — wake word
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
wake_word:
  engine: porcupine  # or "whisper"
  sensitivity: 0.5   # 0.0 (fewer false positives) to 1.0 (more sensitive)
  model_path: ./models/hey-moneypenny.ppn

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
  engine: openai  # or "say" or "elevenlabs"
  voice: nova     # openai: alloy/echo/fable/onyx/nova/shimmer
  speed: 1.0

openclaw:
  gateway_url: ws://127.0.0.1:18789
  channel: voice  # dedicated channel for voice interactions

ui:
  menu_bar: true
  sounds:
    wake: ./sounds/wake.wav      # confirmation chime when wake word detected
    error: ./sounds/error.wav
```

## User Flow

1. Mac mini is always on, `hey-moneypenny` runs as a LaunchAgent
2. User says **"Hey Moneypenny"**
3. 🔔 Short chime plays (confirms wake word detected)
4. System captures speech until 1.5s silence
5. Audio sent to Whisper for transcription
6. Transcribed text sent to OpenClaw gateway
7. OpenClaw processes, returns response text
8. Response converted to speech via TTS
9. Audio plays through Mac mini speaker
10. Return to step 1 (listening for wake word)

## Error Handling
- Network down → fall back to local Whisper + macOS `say`
- Mic disconnected → alert via Telegram, retry every 30s
- OpenClaw gateway down → alert via Telegram, queue messages
- Wake word false positive → short utterance (<1s silence) gets discarded

## Security
- All processing is local-first (wake word detection never leaves device)
- Audio is never stored permanently — transcribe and discard
- OpenClaw gateway is localhost only
- No cloud wake word processing (Porcupine runs on-device)

## Deployment
- Install as macOS LaunchAgent (`~/Library/LaunchAgents/ai.openclaw.hey-moneypenny.plist`)
- Auto-start on login
- Auto-restart on crash
- Logs to `/tmp/hey-moneypenny/`

## Open Questions
- [ ] Does OpenClaw have a voice/audio channel API, or do we need to use an existing channel (e.g., a virtual "voice" session)?
- [ ] Porcupine free tier limits — do we need a paid plan for custom wake words?
- [ ] Should responses also be sent as iMessage/Telegram in addition to spoken? (probably yes, for history)
- [ ] Barge-in support — can user interrupt a long spoken response? (V2)
