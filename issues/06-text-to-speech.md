# Issue: Text-to-speech and audio playback

## Description
Convert OpenClaw's text response to speech and play it through the Mac mini speaker.

## Tasks
- [ ] Implement `tts.py` module:
  - Primary: OpenAI TTS API (`/v1/audio/speech`)
    - Voice options: alloy, echo, fable, onyx, nova, shimmer
    - Default: `nova` (warm, natural)
    - Model: `tts-1` (faster) or `tts-1-hd` (higher quality)
    - Output format: mp3 or wav
    - Speed: configurable (0.25 to 4.0)
  - Fallback: macOS `say` command
    - Voice: system default or configurable
    - Instant, no network required, but robotic
  - Stretch: ElevenLabs API for premium voice quality
- [ ] Audio playback:
  - Use `afplay` (macOS native) for simplicity
  - Or `sounddevice` for more control (non-blocking playback)
  - Play through system default output device
- [ ] Streaming TTS (stretch goal):
  - OpenAI TTS supports streaming response
  - Start playing audio before full response is generated
  - Reduces perceived latency significantly
- [ ] Handle long responses:
  - Split at sentence boundaries for streaming
  - Consider summarizing very long responses for voice (add instruction to OpenClaw)
- [ ] Configuration:
  - `tts.engine`: "openai", "say", "elevenlabs"
  - `tts.voice`: voice name
  - `tts.speed`: playback speed

## Acceptance Criteria
- Natural-sounding speech output through Mac mini speaker
- Latency from text to first audio <2s (API) or <500ms (say)
- Graceful fallback to `say` when API unavailable
- Audible and clear in a kitchen environment

## Labels
`core`, `priority:high`
