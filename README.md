# JARVIS Assistant — Phase 0/1 Scaffold

A working starting point: FastAPI agent backend (memory + tool-calling +
swappable LLMs + voice) and a native Android client that connects to it.
See `docs/ARCHITECTURE.md` for the full design rationale.

## What actually runs right now

- Text chat with memory + tool-calling: `POST /api/chat`
- Real-time voice over WebSocket: `/ws/voice` (needs Deepgram + ElevenLabs keys)
- Android app that connects to the voice endpoint, streams mic audio, plays
  back the response

## Run the backend

```bash
cd backend
cp .env.example .env
# edit .env: at minimum set ANTHROPIC_API_KEY (or OPENAI_API_KEY / GOOGLE_API_KEY)
docker compose up --build
```

Test the brain without any voice/mobile setup:

```bash
curl -X POST http://localhost:8000/api/chat \
  -H "Content-Type: application/json" \
  -d '{"user_id": "dev-user-1", "message": "What is 42 * 17, and remember I prefer metric units"}'
```

That single call exercises: Postgres short-term memory, ChromaDB long-term
memory, the tool registry (calculator tool fires), and the LLM router.

## Run the Android app

1. Open `android/` in Android Studio (Iguana or later).
2. If using an emulator, the default `BACKEND_WS_URL` in
   `JarvisApplication.kt` (`ws://10.0.2.2:8000/...`) already points at your
   host machine's localhost — no changes needed.
3. If using a physical device, run `adb reverse tcp:8000 tcp:8000`, or change
   `BACKEND_WS_URL` to your machine's LAN IP.
4. Run the app, tap "Talk to Jarvis", grant mic permission, speak.

Known Phase 0 limitation (flagged, not hidden): the client writes ElevenLabs'
MP3 stream directly to a PCM `AudioTrack`, which will produce noise, not
speech. Phase 1 wiring adds an `MediaCodec` MP3->PCM decode step — call it
out explicitly if you want that finished before testing playback; text chat
and transcript flow work fully today.

## Roadmap (see docs/ARCHITECTURE.md for detail)

- [x] Phase 0: architecture, repo skeleton, agent core, 2 tools, voice wiring
- [ ] Phase 1: fix MP3 playback decode, add wake word (Porcupine), harden the
      tool-calling loop (partial-message resume, streaming LLM responses)
- [ ] Phase 2: more tools — file/PDF analysis, image generation, code execution
- [ ] Phase 3: vision (screenshots, OCR, camera)
- [ ] Phase 4: computer control (desktop client, screen read, automation)
- [ ] Phase 5: Android system integrations (SMS, calls, contacts, Bluetooth...)
- [ ] Phase 6: smart home (Home Assistant bridge)
- [ ] Phase 7: productivity integrations (calendar, email, reminders)
- [ ] Phase 8: security (face/voice auth, permission system, encryption)
- [ ] Phase 9: full UI (orb, waveform, glassmorphism), other platforms

## Next step

Tell me which of these you want built out next — I'd suggest fixing the
MP3 decode + wake word (finishes Phase 1, makes the voice loop actually
usable end-to-end) before moving on to new tools.
