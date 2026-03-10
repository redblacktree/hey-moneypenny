# Issue: Project scaffold and dev environment

## Description
Set up the `hey-moneypenny` Python project with proper structure, dependency management, and configuration.

## Tasks
- [ ] Initialize Python project with `pyproject.toml` (use `uv` for dependency management)
- [ ] Create project structure:
  ```
  hey-moneypenny/
  ├── src/
  │   ├── __init__.py
  │   ├── main.py           # Entry point, orchestrator
  │   ├── config.py          # YAML config loader
  │   ├── wake_word.py       # Wake word detection
  │   ├── audio_capture.py   # Mic recording + VAD
  │   ├── stt.py             # Speech-to-text
  │   ├── tts.py             # Text-to-speech + playback
  │   ├── openclaw_client.py # Gateway integration
  │   └── ui.py              # Menu bar (optional)
  ├── config/
  │   └── default.yaml
  ├── sounds/
  │   └── wake.wav           # Wake word confirmation chime
  ├── models/                # Wake word model files
  ├── tests/
  ├── pyproject.toml
  └── README.md
  ```
- [ ] Add dependencies: `sounddevice`, `numpy`, `openai`, `pyyaml`, `websocket-client`
- [ ] Create default config YAML
- [ ] Add README with setup instructions
- [ ] Create LaunchAgent plist template

## Labels
`setup`, `priority:high`
