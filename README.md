# Singcheck

[![Made with pollinations.ai](https://raw.githubusercontent.com/pollinations/pollinations/main/packages/ui/src/brand/badge-made-with.svg)](https://pollinations.ai/?ref=badge) [![Made with pollinations.ai](docs/badge-made-with.svg)](https://pollinations.ai/?ref=badge)

Made with [pollinations.ai](https://pollinations.ai/?ref=badge).

Two ways to practise out loud in one page. **Sing a note** shows your pitch live,
in cents, and scores how long you stayed in tune. **Speak a line** records you,
asks a Pollinations transcription model what it heard, and has a text model tell you
which words slipped and what to change.

Live: https://rekty.github.io/singcheck/

## How it works

### Sing (no network)

`getUserMedia` → `AnalyserNode` → normalised autocorrelation on each animation frame.
The fundamental frequency becomes a MIDI value, so the page knows the nearest
semitone and the offset from it in cents. The trace is drawn against a ±50 cent
window; the summary reports time within ±25 cents, mean absolute offset, the
steadiest note and the note held most. Audio never leaves the browser in this mode.

### Speak (Pollinations)

1. **Sign in with Pollinations.** Device flow
   (`POST https://enter.pollinations.ai/api/device/code` → poll
   `/api/device/token`); the token lives in `sessionStorage` for that tab only.
2. **Record.** `MediaRecorder` (opus/webm, mp4 or ogg — whichever the browser
   supports), 20 seconds maximum.
3. **Transcribe.** `POST https://gen.pollinations.ai/v1/audio/transcriptions`,
   multipart, `model=openai/whisper-large-v3` — free (`paid_only: false`), and its
   `supported_endpoints` include `/v1/audio/transcriptions`.
4. **Coach.** `POST https://gen.pollinations.ai/v1/chat/completions` with
   `openai/gpt-5.4-nano` and the target line plus the transcript. The reply is JSON:
   `grade` (`clear` / `almost` / `unclear`), `said`, `heard_gap`, `wins[1-2]`,
   `fixes[1-3]`, `next_try`. The prompt forbids inventing words the transcript does
   not contain and forbids commenting on the person's voice or accent.
5. **Hear it.** `POST https://gen.pollinations.ai/v1/audio/speech` with
   `openai/tts-1` reads the next line so you can copy the rhythm.

Every model call bills **your own** Pollinations balance.

## Models

| Role | Model | Why |
|---|---|---|
| Transcription | `openai/whisper-large-v3` | free, advertises `/v1/audio/transcriptions` |
| Feedback | `openai/gpt-5.4-nano` | free text model, cheap enough to run per take |
| Read-back | `openai/tts-1` | free speech model |
| Pitch | — | autocorrelation in the page, no model, no upload |

## Run it

No build, no dependency:

```bash
python3 -m http.server 8782
# open http://localhost:8782
```

A microphone needs `https` or `localhost`.

## Files

| Path | What |
|---|---|
| `index.html` | the whole app: device sign-in, pitch meter and summary, recorder, transcription, coach prompt, read-back |
| `docs/screenshot.png` | the app with a take scored |
| `docs/badge-made-with.svg` | the official “Made with pollinations.ai” badge, vendored so the page does not depend on another host |

MIT.
