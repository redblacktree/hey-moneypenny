# Issue: Wake word detection — "Hey Moneypenny"

## Description
Implement always-on wake word detection that listens for "Hey Moneypenny" with low CPU usage.

## Tasks
- [ ] Evaluate Porcupine (Picovoice) for custom wake word
  - Sign up at console.picovoice.ai
  - Train "Hey Moneypenny" wake word model
  - Download `.ppn` model file for macOS arm64
- [ ] Implement `wake_word.py` module:
  - Initialize Porcupine with custom model
  - Continuously stream audio from mic
  - Emit event/callback when wake word detected
  - Handle sensitivity configuration
- [ ] Add fallback: if Porcupine unavailable, use simple energy-based VAD + periodic Whisper keyword check (higher CPU, less accurate)
- [ ] Measure idle CPU usage — target <5%
- [ ] Play confirmation chime on wake word detection
- [ ] Add unit tests with mock audio

## Technical Notes
- Porcupine Python SDK: `pip install pvporcupine`
- Requires access key from Picovoice console (free tier supports custom wake words)
- Model is platform-specific — need macOS arm64 build
- Audio format: 16kHz mono, 512-sample frames

## Acceptance Criteria
- Wake word triggers reliably within 500ms
- False positive rate < 1/hour in normal kitchen environment
- CPU usage < 5% while idle/listening

## Labels
`core`, `priority:high`
