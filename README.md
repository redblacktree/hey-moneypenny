# hey-moneypenny 🎙️

Voice assistant for [OpenClaw](https://openclaw.ai). Runs on a Mac mini, listens for "Hey Moneypenny," and lets you have a conversation with your AI assistant through the speaker.

## How It Works

1. **Always listening** — low-power wake word detection waits for "Hey Moneypenny"
2. **You speak** — captures your voice until you stop talking
3. **Transcribes** — sends audio to Whisper for speech-to-text
4. **Thinks** — routes your message through OpenClaw
5. **Responds** — speaks the response through your speaker

## Requirements

- macOS (Apple Silicon)
- Python 3.11+
- USB microphone (tested with HyperX SoloCast)
- OpenClaw gateway running locally
- OpenAI API key (for Whisper + TTS)
- Picovoice access key (for wake word detection)

## Quick Start

```bash
# Install dependencies
uv sync

# Configure
cp config/default.yaml config/local.yaml
# Edit config/local.yaml with your API keys and preferences

# Run
uv run python -m hey_moneypenny --config config/local.yaml

# Or install as LaunchAgent (auto-start on login)
make install
```

## Configuration

See `config/default.yaml` for all options. Key settings:

- **Wake word sensitivity** — tune for your environment
- **STT engine** — OpenAI API (fast) or local Whisper (offline)
- **TTS voice** — choose from OpenAI voices or macOS `say`
- **OpenClaw gateway** — defaults to `ws://127.0.0.1:18789`

## Architecture

See [SPEC.md](./SPEC.md) for the full technical specification.

## License

MIT
