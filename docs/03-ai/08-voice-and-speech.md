# AI/08 · Voice Recognition & Speech

> Status: Draft v1 · Decision: [ADR-0009](../01-architecture/adr/ADR-0009-voice-pipeline.md) · UX: [design/04](../04-design/04-voice-ux-and-conversation-design.md)

## 1. Modes

| Mode | Where | Behavior |
|---|---|---|
| **Push-to-talk** (default) | Desktop & mobile | Hold the Host orb (or `Space` on desktop) to talk; release to send |
| **Hands-free conversation** | Mobile, kitchen/driving-safe contexts | Tap once; VAD detects turns; barge-in interrupts TTS |
| **Voice note** | Any | Record, transcribe, send as text (no live TTS) |
| **Read-aloud** | Any | TTS of assistant replies on demand (accessibility) |

No always-listening wake word in v1 (privacy, battery).

## 2. Pipeline

```text
Mic ─▶ capture (AudioWorklet 16 kHz PCM16 / Opus) ─▶ VAD (Silero, client-side)
   ─▶ voice-gateway (WS) ─▶ streaming STT (partial + final transcripts, endpointing)
   ─▶ conversation.handleTurn(text, prosody hints) ─▶ Host (Claude, streaming tokens)
   ─▶ sentence chunker (first clause ≥ 6 words or punctuation) ─▶ streaming TTS
   ─▶ audio frames back over WS ─▶ playback queue ─▶ speaker
   barge-in: VAD speech start while playing → stop playback, send `interrupt`, keep partial context
```

### Latency budget (end of user speech → first audio)

| Step | Target |
|---|---|
| Endpointing (STT end-of-turn) | 250–400 ms |
| Host first sentence | 500–700 ms (effort `low` in voice express; prompt cached) |
| TTS first audio chunk | 150–250 ms |
| Network/jitter | 100 ms |
| **Total p50** | **≤ 1.2 s** |

Techniques: speculative Host start on high-confidence partial transcripts; short "acknowledgement" earcon (not words) if > 900 ms; proposals delivered as **cards on screen + one spoken summary sentence** (don't read three menus aloud).

## 3. Components & vendors

| Component | Phase 1 | Notes |
|---|---|---|
| Capture | Web Audio API in WebView; `nb-audio` Rust plugin (cpal) fallback | Linux WebKitGTK requires a permission handler; macOS/iOS need `NSMicrophoneUsageDescription`; Android `RECORD_AUDIO` |
| VAD | Silero VAD (onnxruntime-web) | Client-side for barge-in |
| STT | Streaming vendor (Deepgram or AssemblyAI) chosen by eval | Multilingual/code-switching EN↔ES, FR-CA; keyterm boosting for dish & restaurant names |
| On-device STT (privacy mode, desktop) | whisper.cpp via Rust | Non-streaming chunks; slower; opt-in |
| TTS | Streaming vendor (ElevenLabs or Cartesia) chosen by eval | Two voices (warm host, crisp express) per language |
| Transport | WebSocket (Phase 1–2) → LiveKit WebRTC + Agents (Phase 2+) | Better on mobile networks; echo cancellation |

All behind interfaces:

```ts
interface SpeechToText {
  start(opts: { language: "auto" | "en" | "es" | "fr"; keyterms: string[]; sampleRate: 16000 }): SttStream;
}
interface SttStream {
  write(frame: Int16Array): void;
  on(event: "partial" | "final" | "endOfTurn" | "error", cb: (e: SttEvent) => void): void;
  close(): Promise<void>;
}
interface TextToSpeech {
  stream(text: AsyncIterable<string>, opts: { voice: VoiceId; rate: number; language: string }): AsyncIterable<AudioChunk>;
}
```

## 4. Vocabulary & understanding

- **Keyterms per session:** names of open merchants nearby, dish names in current proposals, avatar names ("Mia", "Rosa") → boosted in STT.
- **Normalization:** "the second one", "the Thai one", "same as Leo" → resolved against on-screen proposals by the Host with UI state in the live context.
- **Numbers & money:** "under twenty bucks" → budget 2000 minor units USD.
- **Code-switching:** "Quiero algo light, nothing too heavy" handled; reply in the dominant language of the turn or the user's preference.

## 5. Voice-specific dial signals

- Speech rate vs. user baseline, pause length, interruptions (barge-ins) → urgency/directness adjustments.
- Never infer emotion labels for storage; prosody affects only the current session's dials.

## 6. Privacy & safety

- Audio streamed, not stored by default; transcripts stored as messages (user-deletable).
- No speaker identification or voiceprints (biometric laws); multiple speakers treated as one user.
- On-screen mic indicator; mic stops after 8 s of silence in hands-free mode and after each push-to-talk.
- Voice confirmations for anything sensitive are repeated on screen ("Remember: Mia — sesame (severe)?") and require a tap or explicit "yes".
- Payment is never confirmed by voice alone in v1; voice can prepare checkout, a tap/biometric completes it.

## 7. Testing

- Eval set: 300 recorded utterances (accents, noise, kitchen background, kids, EN/ES/FR-CA, dish names) → WER and slot accuracy (dishes, names, numbers).
- Latency harness: synthetic audio through the full pipeline; p50/p95 per vendor combination.
- Barge-in tests; network degradation tests (3G profile, packet loss).
