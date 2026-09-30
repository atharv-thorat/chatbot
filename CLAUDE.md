# VoiceBot — Project Spec for Claude Code

## What this is
A full-stack, LLM-powered chatbot with a voice assistant. Users can type or talk;
the bot replies in streamed text and speaks the reply aloud. Not rule-based:
all understanding and responses come from an LLM behind a provider abstraction.

## Stack (do not change without asking)
- **Frontend:** Next.js 14 (App Router), TypeScript, Tailwind CSS — in `/frontend`
- **Backend:** FastAPI, Python 3.11, Pydantic v2, SQLModel + SQLite — in `/backend`
- **LLM:** provider interface with two implementations:
  - `GroqProvider` (default, cloud) — model name from env `LLM_MODEL`
  - `OllamaProvider` (local/offline) — base URL from env `OLLAMA_URL`
- **STT:** provider interface: `GroqWhisperSTT` (default), `FasterWhisperSTT` (local)
- **TTS:** provider interface: `EdgeTTS` (default, uses `edge-tts` package), `PiperTTS` (local)
- Provider selection via env vars: `LLM_PROVIDER`, `STT_PROVIDER`, `TTS_PROVIDER`

## Architecture
- `POST /api/chat` — body `{conversation_id?, message}` → **SSE stream** of tokens,
  final event includes `conversation_id`
- `POST /api/stt` — multipart audio (webm/opus from MediaRecorder) → `{text}`
- `POST /api/tts` — `{text, voice?}` → `audio/mpeg`
- `GET /api/conversations`, `GET /api/conversations/{id}`, `DELETE /api/conversations/{id}`
- `GET /api/health`
- System prompt lives in `backend/app/prompts/system.md` and is loaded at startup.

### Voice flow (frontend)
1. Push-to-talk mic button → MediaRecorder → `/api/stt` → transcript shown as user message
2. Transcript sent to `/api/chat`; tokens stream into the UI
3. As each full sentence completes, call `/api/tts` and enqueue audio in a playback queue
4. **Barge-in:** pressing the mic or Stop while speaking clears the audio queue and
   aborts the in-flight chat request (AbortController)

## Folder layout
```
backend/app/{main.py, config.py, api/, providers/{llm,stt,tts}/, db/, prompts/}
backend/tests/
frontend/src/{app/, components/, hooks/, lib/}
```

## Conventions
- Python: type hints everywhere, `ruff` for lint + format, `pytest` for tests.
  Tests mock all providers — no real API calls in tests.
- TypeScript: strict mode, ESLint, no `any`.
- Config only through env vars; keep `.env.example` in sync. **Never commit `.env`.**
- Small focused commits using Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`).
- After finishing any task: run lint + tests, fix failures, then summarize what changed.

## Commands
- Backend: `cd backend && uvicorn app.main:app --reload`
- Backend tests: `cd backend && pytest`
- Frontend: `cd frontend && npm run dev`
- Full stack: `docker compose up --build`
