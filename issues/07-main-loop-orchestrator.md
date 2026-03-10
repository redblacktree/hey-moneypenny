# Issue: Main loop / orchestrator

## Description
Wire all components together into the main event loop: wake → capture → transcribe → process → speak → repeat.

## Tasks
- [ ] Implement `main.py` orchestrator:
  - Load config from YAML
  - Initialize all components (wake word, audio, STT, TTS, OpenClaw client)
  - Main loop:
    1. Listen for wake word (blocking, low CPU)
    2. Play wake confirmation chime
    3. Capture audio until silence
    4. Transcribe audio
    5. Send to OpenClaw, await response
    6. Speak response via TTS
    7. Return to step 1
  - Handle component failures gracefully
- [ ] State machine:
  - `IDLE` — listening for wake word
  - `LISTENING` — capturing speech
  - `PROCESSING` — transcribing + waiting for OpenClaw
  - `SPEAKING` — playing TTS response
  - `ERROR` — something broke, attempting recovery
- [ ] Signal handling:
  - SIGTERM/SIGINT → graceful shutdown
  - SIGHUP → reload config
- [ ] Logging:
  - Structured logging to `/tmp/hey-moneypenny/`
  - Log state transitions, latencies, errors
  - Rotate logs daily
- [ ] CLI arguments:
  - `--config <path>` — config file
  - `--verbose` — debug logging
  - `--test-tts` — speak a test phrase and exit
  - `--test-stt` — record 5s and transcribe, then exit

## Acceptance Criteria
- Full voice interaction loop works end-to-end
- Recovers from transient errors without manual restart
- Clean shutdown on SIGTERM
- Total round-trip latency (wake word → response starts playing) <8s typical

## Labels
`core`, `priority:high`
