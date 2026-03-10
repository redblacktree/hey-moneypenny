# Issue: Audio capture with voice activity detection (VAD)

## Description
After wake word triggers, capture the user's speech from the microphone and detect when they stop talking.

## Tasks
- [ ] Implement `audio_capture.py` module:
  - Start recording from system default input device (HyperX SoloCast)
  - Use Voice Activity Detection to find speech boundaries
  - Stop recording after configurable silence threshold (default 1.5s)
  - Enforce max utterance length (default 30s)
  - Return audio as 16kHz mono WAV bytes
- [ ] VAD implementation options:
  - `webrtcvad` — lightweight, proven, C-based
  - `silero-vad` — ML-based, more accurate, slightly heavier
  - Simple energy threshold — cheapest, least accurate
- [ ] Handle edge cases:
  - User says wake word but then nothing (timeout after 5s → discard)
  - Very short utterance (<0.5s) — likely false positive, discard
  - Mic disconnected mid-recording — graceful error
- [ ] Audio preprocessing:
  - Noise gate / basic noise reduction
  - Normalize audio levels
- [ ] Add tests with sample audio files

## Acceptance Criteria
- Captures clear speech from HyperX SoloCast at arm's length (~2-3 feet)
- Reliably detects end of utterance within 1.5s of silence
- Handles kitchen background noise (fridge, AC, etc.)
- Returns clean WAV suitable for Whisper

## Labels
`core`, `priority:high`
