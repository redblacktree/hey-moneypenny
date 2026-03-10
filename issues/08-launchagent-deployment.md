# Issue: LaunchAgent deployment and auto-start

## Description
Package hey-moneypenny as a macOS LaunchAgent so it starts automatically on login and restarts on crash.

## Tasks
- [ ] Create LaunchAgent plist:
  - `~/Library/LaunchAgents/ai.openclaw.hey-moneypenny.plist`
  - RunAtLoad: true
  - KeepAlive: true (restart on crash)
  - StandardOutPath / StandardErrorPath → `/tmp/hey-moneypenny/`
  - WorkingDirectory → project dir
  - EnvironmentVariables: OPENAI_API_KEY, PICOVOICE_ACCESS_KEY
- [ ] Install script: `make install` or `./install.sh`
  - Copies plist to LaunchAgents
  - Sets up Python venv if needed
  - Validates mic is available
  - Loads the agent
- [ ] Uninstall script: `make uninstall`
- [ ] Health check:
  - Expose a simple health endpoint or write heartbeat to a file
  - OpenClaw can monitor this in HEARTBEAT.md
- [ ] Log rotation:
  - Daily log files in `/tmp/hey-moneypenny/`
  - Clean up logs older than 7 days

## Acceptance Criteria
- Starts automatically on Mac mini login
- Restarts within 10s if process crashes
- Logs are accessible and rotated
- Easy install/uninstall

## Labels
`deployment`, `priority:medium`
