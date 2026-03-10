# Issue: Wake word detection — Multi-Agent

## Description
Implement always-on wake word detection that listens for multiple wake words simultaneously and routes to the correct agent. Initial wake words: "Hey Moneypenny" → Moneypenny, "Hey Sheila" → Sheila.

## Tasks
- [ ] Evaluate Porcupine (Picovoice) for multi-keyword detection
  - Sign up at console.picovoice.ai
  - Train "Hey Moneypenny" and "Hey Sheila" wake word models
  - Download `.ppn` model files for macOS arm64
- [ ] Implement `wake_word.py` module:
  - Initialize Porcupine with **multiple keyword paths** (one per agent)
  - Continuously stream audio from mic
  - On detection, return **which keyword** triggered (Porcupine returns keyword index)
  - Map keyword index → agent name from config
  - Emit event/callback with detected agent identifier
  - Handle per-keyword sensitivity configuration
- [ ] Support adding new agents by adding entries to `agents:` config (no code changes)
- [ ] Add fallback: if Porcupine unavailable, use simple energy-based VAD + periodic Whisper keyword check (higher CPU, less accurate)
- [ ] Measure idle CPU usage — target <5%
- [ ] Play confirmation chime on wake word detection
- [ ] Add unit tests with mock audio

## Technical Notes
- Porcupine Python SDK: `pip install pvporcupine`
- Porcupine supports simultaneous multi-keyword detection via `keyword_paths` (list) and `sensitivities` (list)
- `porcupine.process(frame)` returns the **index** of the detected keyword (-1 if none)
- Requires access key from Picovoice console (free tier supports 3 custom wake words — sufficient for initial agents)
- Model is platform-specific — need macOS arm64 build
- Audio format: 16kHz mono, 512-sample frames

## Acceptance Criteria
- Each wake word triggers reliably within 500ms
- Correct agent is identified on trigger (no cross-talk between wake words)
- False positive rate < 1/hour per keyword in normal environment
- CPU usage < 5% while idle/listening
- New agents can be added via config only

## Labels
`core`, `priority:high`
