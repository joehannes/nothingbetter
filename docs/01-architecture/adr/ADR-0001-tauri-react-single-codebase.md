# ADR-0001 · Tauri 2 + React as the single client codebase

- Status: Accepted (2026-10-15)
- Deciders: Founder, Engineering

## Context
The product must run as a desktop app, mobile app and website. The founder asked for Tauri + React. Voice, GPS and push are needed on every platform.

## Decision
One React 19 + TypeScript SPA (`apps/app`) runs on the web and inside a Tauri 2 shell (`apps/tauri`) for macOS, Windows, Linux, iOS and Android. Platform differences live behind `src/platform/*` adapters (geo, audio, push, storage, deep links).

## Consequences
- + One UI, one design system, shared contracts with the backend.
- + Small binaries; Rust available for native audio and permission handling.
- − WebView differences (WebKitGTK on Linux needs explicit `getUserMedia` permission handling; iOS WKWebView audio quirks) → native `nb-audio` fallback.
- − Background location for a courier app is limited in Tauri mobile → courier mode uses the dispatch vendor's driver app until a native plugin exists.

## Alternatives
React Native (second UI stack), Electron (no mobile, heavier), Flutter (no React reuse).
