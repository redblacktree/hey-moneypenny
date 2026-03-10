# Issue: OpenClaw gateway integration

## Description
Send transcribed text to OpenClaw and receive the assistant's response.

## Tasks
- [ ] Implement `openclaw_client.py` module:
  - Connect to OpenClaw gateway at `ws://127.0.0.1:18789`
  - Research OpenClaw's WebSocket protocol / API for programmatic message injection
  - Send transcribed text as a user message
  - Receive assistant response text
  - Handle connection lifecycle (connect, reconnect, heartbeat)
- [ ] Determine the best integration approach:
  - **Option A**: WebSocket API (if OpenClaw exposes one for programmatic use)
  - **Option B**: Create a custom OpenClaw channel plugin for "voice"
  - **Option C**: Use the existing BlueBubbles/Telegram channel as a relay
  - **Option D**: HTTP REST API (check if gateway exposes one)
  - Consult OpenClaw docs at `/Users/moneypenny/.openclaw/workspace/docs`
- [ ] Session management:
  - Should voice interactions go to the main session or a dedicated voice session?
  - Probably main session — so Moneypenny has full context
- [ ] Handle streaming responses:
  - If OpenClaw streams tokens, buffer until sentence boundary for TTS
  - If not streaming, wait for full response
- [ ] Error handling:
  - Gateway down → queue message, retry, alert via other channel
  - Timeout → "I'm having trouble connecting, try again"

## Research Needed
- Read OpenClaw gateway API docs
- Check if there's a REST or WebSocket endpoint for message injection
- Look at how existing channels (Telegram, Slack) integrate — may need to create a voice channel plugin

## Acceptance Criteria
- Sends transcribed text to OpenClaw reliably
- Receives full response text
- Reconnects automatically if gateway restarts
- Response latency from send to receive <10s (depends on model)

## Labels
`core`, `priority:high`, `research`
