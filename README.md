# Relay — CommuniLens

Sign ⇄ speech accessibility demo. A single-page web app that recognises **static hand-shapes** on-device (Google MediaPipe Hand Landmarker), maps trained signs to words, and uses the **Gemini API** to compose one natural spoken sentence — plus a reverse "Listen" direction that captions the hearing person's speech.

Everything runs client-side. No build step, no server.

## Run locally

Camera APIs require a secure context. `localhost` counts as secure, so any static server works:

```bash
python3 -m http.server 8000
# then open http://localhost:8000 in Chrome
```

## Configure the Gemini key

Open `index.html` and set your key in the config block near the top of the `<script>`:

```js
const GEMINI_API_KEY = "";                 // paste your Gemini key here
const GEMINI_MODEL    = "gemini-3.6-flash"; // current Flash model
```

⚠️ **Never commit a live key to this public repo.** Keep `GEMINI_API_KEY = ""` in git; add the real key only in your local copy or at deploy time. The key is visible in the hosted client source (acceptable for a short-lived demo) — restrict it in the Google Cloud console and rotate/delete it after the event. Without a key, the app still runs and speaks via an offline fallback.

## Deploy (Firebase Hosting)

```bash
npm install -g firebase-tools
firebase login
firebase init hosting   # public dir: .  — decline overwriting index.html (firebase.json is included)
firebase deploy         # prints the HTTPS URL (camera needs HTTPS on mobile)
```

## Usage

1. Hold a hand shape, type a word, tap **Record** ~4–5× per sign. Trained signs persist across refreshes (`localStorage`).
2. Hold a trained shape → its word joins the on-screen **gloss** list.
3. Tap **🔊 Speak** → Gemini composes and speaks one sentence, shown as a caption.
4. Tap **🎙️ Listen** → transcribes the hearing person's speech into captions (Chrome; not iOS Safari).

Best on desktop Chrome or Android Chrome. On iOS Safari the camera/recognition/speech-out work, but "Listen" (SpeechRecognition) is unavailable.
