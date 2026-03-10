# Issue: Speech-to-text via Whisper

## Description
Transcribe captured audio to text using OpenAI Whisper API with local fallback.

## Tasks
- [ ] Implement `stt.py` module:
  - Primary: OpenAI Whisper API (`/v1/audio/transcriptions`)
    - Model: `whisper-1`
    - Language hint: `en`
    - Send audio as WAV bytes
    - Return transcription text
  - Fallback: Local Whisper via subprocess (`/opt/homebrew/bin/whisper` or `whisper.cpp`)
    - Use `base` or `small` model for speed
    - Parse stdout for transcription
  - Auto-fallback: if API call fails (network error, timeout), retry once then fall back to local
- [ ] Configuration:
  - `stt.engine`: "openai" or "local"
  - `stt.model`: API model name or local model size
  - `stt.language`: language hint
- [ ] Measure latency:
  - API: target <2s for typical utterance
  - Local: target <5s for typical utterance
- [ ] Handle empty/garbage transcriptions (whisper sometimes hallucinates on silence)

## Acceptance Criteria
- Accurate transcription of conversational English
- API latency <2s for utterances up to 15s
- Graceful fallback to local when API unavailable
- No hallucinated text from silence/noise

## Labels
`core`, `priority:high`
