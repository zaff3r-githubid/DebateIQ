# DebateIQ — AI Debate Strategist

Multi-layer AI reasoning system for the AI Practitioner course.
Debate any topic through 5 progressive reasoning layers, with live telemetry, a score card, coach feedback, and a live voice debate mode. Neapolitan vs New York Style Pizza is the default demo topic.

---

## How the API key is protected

```text
Visitor's browser  →  Cloudflare Worker (your key lives here, encrypted)  →  Anthropic API
```

The Anthropic API key is stored as an **encrypted Cloudflare secret** — it never appears in this repo, in the browser, or in network requests visible to visitors. The GitHub repo is safe to be public.

---

## Setup: Part 1 — Cloudflare Worker (one-time, ~5 minutes)

### Step 1 — Create a free Cloudflare account

Go to [cloudflare.com](https://cloudflare.com) → Sign up (free tier is enough).

### Step 2 — Deploy the Worker via the Dashboard (no CLI needed)

1. Log in to [dash.cloudflare.com](https://dash.cloudflare.com)
2. Click **Workers & Pages** in the left sidebar
3. Click **Create** → **Create Worker**
4. Give it the name `debateiq-proxy`
5. Click **Deploy** (ignore the default code for now)
6. On the next screen click **Edit code**
7. Delete everything in the editor, paste the entire contents of `worker.js` from this repo
8. Click **Deploy**

### Step 3 — Add your API key as a secret

1. Go back to your Worker → click **Settings** tab → **Variables**
2. Under **Environment Variables** click **Add variable**
3. Set:
   - **Variable name:** `ANTHROPIC_API_KEY`
   - **Value:** your key from [console.anthropic.com](https://console.anthropic.com)
   - Toggle **Encrypt** ON
4. Click **Save and deploy**

### Step 4 — Copy your Worker URL

Your Worker URL is shown at the top of the Worker page, e.g.:

```text
https://debateiq-proxy.YOUR-SUBDOMAIN.workers.dev
```

---

## Setup: Part 2 — GitHub Pages

### Step 5 — Update the Worker URL in index.html

Open `index.html` and find this line near the top of the `<script>` block:

```javascript
const WORKER_URL = 'https://debateiq-proxy.YOUR-SUBDOMAIN.workers.dev';
```

Replace `YOUR-SUBDOMAIN` with your actual Cloudflare subdomain, then save.

### Step 6 — Push to GitHub

1. Create a new **public** GitHub repo (e.g. `debateiq`)
2. Push these files:

```text
index.html
README.md
worker.js        ← safe to include, contains no secrets
wrangler.toml    ← safe to include, contains no secrets
```

1. Go to **Settings → Pages → Source → Deploy from branch → main → / (root) → Save**
2. Your app is live at `https://YOUR-USERNAME.github.io/debateiq/`

---

## Local development (your private copy)

Since the Worker URL is public and contains no secrets, the same `index.html` works locally — just open it in your browser. It calls the Cloudflare Worker which holds your key securely.

If you want a fully offline local setup, install [Wrangler](https://developers.cloudflare.com/workers/wrangler/) and run the Worker locally:

```bash
npm install -g wrangler
wrangler secret put ANTHROPIC_API_KEY
wrangler dev worker.js
```

Then temporarily set `WORKER_URL = 'http://localhost:8787'` in index.html for local testing.

---

## Features

| Feature | Description |
| --- | --- |
| 5-Layer Reasoning | Context → Arguments → Counter → Self-Critique → Final Strategy |
| Live Telemetry | Input tokens, output tokens, total tokens, latency, estimated cost |
| Score Card | AI judge scores Strength, Evidence, Logic, Persuasion (0-100) |
| Coach Feedback | Personal strengths, weaknesses, exercises, and quick wins |
| Live Debate | Speak your arguments; AI counters in real-time |
| Voice Playback | Browser TTS (free) or ElevenLabs (optional premium) |
| Export | JSON, Markdown, Print/PDF, Copy all |
| History | Last 30 debates saved to localStorage |

## Assignment Rubric Coverage

| Category | Points | How it's covered |
| --- | --- | --- |
| Multi-layer reasoning | 25 | 5 layers, each receives all prior layers as context — genuinely builds, no repetition |
| Output clarity | 15 | Typing animation, layer cards, animated score bars, coach markdown |
| Telemetry Optimization | 20 | Per-layer + cumulative tokens, latency, and cost displayed live |
| Reasoning depth | 15 | Chained prompts force new reasoning at every layer |
| Video demo | 10 | Record the app running + telemetry dashboard |
