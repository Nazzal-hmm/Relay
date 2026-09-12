# Relay — Two-way AR Translation

A real-time, two-way translation app between a Deaf/Hard-of-Hearing (DHH) signer and a hearing person who doesn't sign — built for a face-to-face scenario like a café counter.

- **Sign → speech:** the camera recognises the signer's hand-shapes on-device and speaks a sentence aloud for the hearing person.
- **Speech → captions:** the microphone hears the hearing person and shows live captions for the signer.

Everything runs client-side in a single `index.html` — no build step, no server.

**Live demo:** https://ireland-hackathon26dub-4302.web.app
*(Best on desktop Chrome or Android Chrome. On iOS Safari the camera + speaking work, but the "Listening" speech-to-text captions are unavailable.)*

---

## How it works

```
Webcam ─▶ MediaPipe Hand Landmarker (on-device) ─▶ 21 hand landmarks
       ─▶ normalise to a 42-number vector
       ─▶ 1-nearest-neighbour match vs. your trained samples
       ─▶ stability filter (N agreeing frames) ─▶ commit a sign
                                                     │
                                    ┌────────────────┴─────────────────┐
                            Instant mode                        Compose mode
                     one sign ▶ speaks a full,            signs accumulate ▶ Gemini
                     pre-set sentence immediately          writes one sentence at the pause
                                                     │
                                    ─▶ Web Speech TTS + on-screen caption

Reverse:  mic ─▶ Web Speech SpeechRecognition ─▶ live caption for the signer
```

### The 4-state turn-taking flow
1. **Setup** — camera armed, auto-detecting signs.
2. **Signing** — a recognised sign is spoken aloud; a haptic pulse confirms each phrase, a distinct second pulse marks end-of-turn.
3. **Hand-off** — the phone says *"Waiting for your response"* aloud and starts listening (for the hearing person, who may not see the screen).
4. **Listening** — live captions of the hearing person's reply, then loops back to Setup.

### Two speaking modes
- **⚡ Instant phrases (default):** one recognised keyword instantly speaks a full, pre-written sentence. Fast and demo-proof — no network round-trip. Sentences are editable in the app and travel with Export/Import.
- **🧠 Compose (Gemini):** recognised signs accumulate into a gloss, and Gemini (`gemini-3.6-flash`) composes one natural sentence at the end of the turn, with an offline fallback so it never hard-crashes.

---

## Run locally

Camera APIs need a secure context; `localhost` counts as secure, so any static server works:

```bash
cd public
python3 -m http.server 8000
# open http://localhost:8000 in Chrome
```

## Configure the Gemini key (only needed for Compose mode)

Open `public/index.html` and set your key in the config block near the top of the `<script>`:

```js
const GEMINI_API_KEY = "";                  // paste your Gemini key here (local/deploy only)
const GEMINI_MODEL    = "gemini-3.6-flash";
```

⚠️ **Never commit a live key to this public repo.** It is intentionally blank here. Add the real key only in your local copy or at deploy time. The key is visible in the hosted client source (acceptable for a short-lived demo) — restrict it in the Google Cloud console and rotate/delete it after the event. Instant mode works with no key at all.

## Train signs

Recognition is **learned from your own hand-shapes**, not hardcoded:

1. Open **⚙ Demo & training**.
2. Hold a shape steady, type a word (or tap a seed chip), tap **● Record sample** ~4–5×. Use clearly **distinct** shapes.
3. Trained signs persist in the browser (`localStorage`).

### Move signs between devices (train on laptop → demo on phone)

`localStorage` is per-device, so trained signs don't sync automatically. Use the built-in transfer:

- **⬆ Export signs** — copies a code to the clipboard, downloads `relay-signs.json`, and shows the code. Carries the phrase list too.
- **⬇ Import signs** — paste the code and tap Import, or tap Import with the box empty to pick the file.

## Deploy (Firebase Hosting)

`firebase.json` publishes only the `public/` folder.

```bash
npm install -g firebase-tools
firebase login
firebase deploy --only hosting
```

---

## Accessibility

Built to the spec's **WCAG AAA** mandate:

- Every status colour is paired with an **icon + text label** (a persistent legend is always visible) — colour is never the sole carrier of meaning.
- Contrast ratios: signal-green (~12.5:1), listen-blue (~10.7:1), primary/secondary text (~15:1 / ~11:1) against the dark background.
- Live captions and status strings are mirrored to screen-reader live regions (`assertive` for alerts, `polite` for streaming captions).
- Distinct haptic patterns for per-phrase, end-of-turn, and listening-mode-entered.
- The hand-off state speaks its prompt aloud automatically — for a hearing recipient who may be blind or not looking at the screen.
- All controls are ≥48px single-tap targets with a visible focus ring; `prefers-reduced-motion` is respected.

## Tech stack

| Layer | Tool |
|---|---|
| Hand recognition | Google MediaPipe Hand Landmarker (`@mediapipe/tasks-vision`, WASM, on-device) |
| Language (Compose mode) | Gemini API `generateContent` (`gemini-3.6-flash`) |
| Speech out | Web Speech `SpeechSynthesis` |
| Speech in | Web Speech `SpeechRecognition` (Chrome) |
| Persistence | `localStorage` |
| Hosting | Firebase Hosting (HTTPS) |

## Repo layout

```
public/index.html   The whole app (single file). Deployed and served as-is.
firebase.json       Hosting config — publishes only public/.
README.md           This file.
```
