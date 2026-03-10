# Issue: Menu bar status UI (optional)

## Description
A minimal macOS menu bar icon showing the current state of hey-moneypenny.

## Tasks
- [ ] Implement `ui.py` using `rumps` (Python macOS menu bar library):
  - Icon states:
    - 🟢 Idle — listening for wake word
    - 🟡 Active — capturing/processing speech
    - 🔴 Error — mic disconnected, gateway down, etc.
    - ⚫ Muted — wake word detection paused
  - Menu items:
    - "Mute / Unmute" — toggle wake word listening
    - "Test Voice" — speak a test phrase
    - "Open Logs" — open log directory in Finder
    - "Quit"
- [ ] Optional: Show last interaction summary in menu
- [ ] Keyboard shortcut to mute/unmute (e.g., global hotkey)
- [ ] Integration with main loop — UI runs on main thread, audio on background threads

## Acceptance Criteria
- Menu bar icon visible and reflects current state
- Mute toggle works reliably
- Doesn't interfere with audio processing

## Labels
`ui`, `priority:low`, `nice-to-have`
