# ADR-0009 · Cascaded, vendor-neutral voice pipeline

- Status: Accepted (2026-10-15)

## Context
Voice conversations on laptop and mobile need low latency, barge-in, multilingual support (EN/ES/FR, code-switching) and privacy.

## Decision
Cascade: capture → VAD → streaming STT → Claude (streaming) → sentence chunker → streaming TTS → playback, behind `SpeechToText` and `TextToSpeech` interfaces. Transport: WebSocket to `voice-gateway` in Phase 1–2, LiveKit (WebRTC + Agents) in Phase 2+ for mobile networks. Optional on-device STT (whisper.cpp) in the Tauri desktop app for a privacy mode.

## Consequences
- + Swap vendors by eval (accuracy on food names, latency, price).
- + Same orchestrator for text and voice.
- − More moving parts than a single speech-to-speech model; latency must be engineered (streaming everywhere, early TTS).
