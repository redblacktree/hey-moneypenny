# Issue: Wake word detection — Multi-Agent (openWakeWord)

## Description
Implement always-on wake word detection using openWakeWord that listens for multiple wake words simultaneously and routes to the correct agent. Initial wake words: "Hey Moneypenny" → Moneypenny, "Hey Sheila" → Sheila.

## Tasks
- [ ] Train custom wake word models locally:
  - Generate synthetic speech samples using OpenAI TTS (multiple voices, speeds, inflections)
  - Augment with background noise and room impulse responses
  - Train using openWakeWord's training notebook/script
  - Export `.onnx` models for each wake word
- [ ] Implement `wake_word.py` module:
  - Initialize openWakeWord `Model` with multiple `wakeword_models` (one per agent)
  - Enable Silero VAD via `vad_threshold` to reduce false positives
  - Continuously stream 16kHz mono audio frames from mic
  - On detection, identify **which model** triggered and map to agent name from config
  - Emit event/callback with detected agent identifier
  - Handle per-model threshold configuration
- [ ] Support adding new agents by adding entries to `agents:` config + training a new model (no code changes)
- [ ] Measure idle CPU usage — target <5%
- [ ] Play confirmation chime on wake word detection
- [ ] Add unit tests with mock audio

## Technical Notes
- openWakeWord Python library: `pip install openwakeword`
- `Model(wakeword_models=["path/to/model1.onnx", "path/to/model2.onnx"])`
- `model.predict(frame)` returns dict of `{model_name: score}` — check each against threshold
- Audio format: 16kHz mono, frames should be multiples of 80ms for best efficiency
- Supports both `.tflite` and `.onnx` model formats
- Built-in Silero VAD integration: `vad_threshold=0.5` filters out non-speech noise
- No API key or vendor account required — fully self-contained
- Training generates models from synthetic speech — no need to manually record samples

## Training Notes
- Use openWakeWord's `automatic_model_training.ipynb` notebook
- Generate ~3000 synthetic samples per wake word using varied TTS voices
- Augment with noise datasets (room tone, HVAC, TV, etc.)
- Training takes ~1 hour locally or on Colab
- Test with `model.predict_clip("test.wav")` before deploying

## Acceptance Criteria
- Each wake word triggers reliably within 500ms
- Correct agent is identified on trigger (no cross-talk between wake words)
- False positive rate < 1/hour per keyword in normal environment
- CPU usage < 5% while idle/listening
- New agents can be added via config + new model only (no code changes)

## Labels
`core`, `priority:high`
