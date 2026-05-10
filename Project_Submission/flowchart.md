# System Flowchart — DebateIQ

**Project:** DebateIQ — AI Debate Strategist
**Course:** AI Practitioner Warm-Up Assignment

---

## High-Level Architecture

```text
┌─────────────────────────────────────────────────────────────────┐
│                        USER / BROWSER                           │
│            Opens DebateIQ via GitHub Pages                      │
└─────────────────────┬───────────────────────────────────────────┘
                      │  HTTPS request
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                   GITHUB PAGES (Hosting)                        │
│           Serves index.html — single static file                │
│           Global CDN · Auto-deploys on git push                 │
└─────────────────────┬───────────────────────────────────────────┘
                      │  API call (no API key in browser)
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│              CLOUDFLARE WORKER (Secure Proxy)                   │
│    debateiq-proxy.YOUR-SUBDOMAIN.workers.dev                    │
│    ┌─────────────────────────────────────────────┐              │
│    │  ANTHROPIC_API_KEY stored as encrypted secret│             │
│    └─────────────────────────────────────────────┘              │
│    · Receives request from browser                              │
│    · Attaches secret API key                                    │
│    · Forwards to Anthropic · Returns response                   │
└─────────────────────┬───────────────────────────────────────────┘
                      │  Authenticated API call
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                  ANTHROPIC CLAUDE API                           │
│              Model: claude-sonnet-4-6                           │
│              Max tokens: 1200-1400 per call                     │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5-Layer Reasoning Pipeline (Core Logic)

```text
User enters topic + position (or picks a preset), selects Persona · Tone · Source
                            │
                            ▼
              ┌─────────────────────────┐
              │   LAYER 1               │
              │   Context Analysis      │◄── Prompt: neutral landscape mapping
              │   "What is the problem?"│    No position taken
              └────────────┬────────────┘
                           │  Layer 1 output passed as context
                           ▼
              ┌─────────────────────────┐
              │   LAYER 2               │
              │   Argument Builder      │◄── Prompt: defend position
              │   "What is my stance?"  │    + Layer 1 context
              └────────────┬────────────┘
                           │  Layers 1+2 output passed as context
                           ▼
              ┌─────────────────────────┐
              │   LAYER 3               │
              │   Counter Argument      │◄── Prompt: challenge position
              │   "What opposes me?"    │    + Layers 1+2 context
              └────────────┬────────────┘
                           │  Layers 1+2+3 output passed as context
                           ▼
              ┌─────────────────────────┐
              │   LAYER 4               │
              │   Self-Critique         │◄── Prompt: find weaknesses
              │   "Where am I weak?"    │    + Layers 1+2+3 context
              └────────────┬────────────┘
                           │  All 4 layers passed as context
                           ▼
              ┌─────────────────────────┐
              │   LAYER 5               │
              │   Final Strategy        │◄── Prompt: synthesise all layers
              │   "What's the verdict?" │    Full chain = deepest reasoning
              └────────────┬────────────┘
                           │
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
    ┌────────────┐  ┌────────────┐  ┌────────────┐
    │ SCORE CARD │  │   COACH    │  │  INSIGHTS  │
    │ JSON judge │  │ Feedback   │  │  Charts    │
    │ 4 metrics  │  │ Coaching   │  │  Telemetry │
    └────────────┘  └────────────┘  └────────────┘
```

---

## Telemetry Flow

```text
Each API call → { input_tokens, output_tokens } returned by Anthropic
                            │
                            ▼
            Running accumulators in browser state:
            ┌──────────────────────────────────┐
            │  total_input_tokens  += n         │
            │  total_output_tokens += n         │
            │  total_tokens        += n         │
            │  avg_latency = rolling average    │
            │  est_cost = in×$3/M + out×$15/M  │
            └──────────────────────────────────┘
                            │
                            ▼
            Displayed live in Telemetry Bar (top of results)
            Displayed per-layer in each Layer Card footer
            Displayed in Insights Panel (bar chart)
```

---

## Live Debate Flow

```text
User clicks "Start Live Debate"
        │
        ▼
AI makes opening statement (Prompt 8 → Cloudflare → Anthropic)
        │
        ▼
AI speaks opening (ElevenLabs TTS  OR  Browser SpeechSynthesis)
        │
        ▼
    ┌───────────────────────────────────────┐
    │         DEBATE LOOP                   │
    │                                       │
    │  Browser Web Speech API listens       │
    │  User speaks → transcribed to text    │
    │            │                          │
    │            ▼                          │
    │  Transcript added to conversation     │
    │  history (last 6 messages)            │
    │            │                          │
    │            ▼                          │
    │  Prompt 9 → Cloudflare → Anthropic    │
    │  AI generates counter-argument        │
    │            │                          │
    │            ▼                          │
    │  AI speaks response (TTS)             │
    │            │                          │
    │            ▼                          │
    │  Repeat until user clicks "End"       │
    └───────────────────────────────────────┘
        │
        ▼
Prompt 10 → Cloudflare → Anthropic
AI judges the full transcript → JSON result
        │
        ▼
Winner declared · Scores shown · Improvement tip displayed
```

---

## Data Storage

```text
┌─────────────────────────────────────────────────────┐
│                  BROWSER localStorage                │
│                                                      │
│  diq_hist  →  last 30 debate results (JSON)          │
│  diq_el    →  ElevenLabs key (optional, encrypted)   │
│                                                      │
│  No data ever sent to any server except:             │
│  · The prompt text → Cloudflare Worker → Anthropic   │
│  · ElevenLabs TTS text (if key provided)             │
└─────────────────────────────────────────────────────┘
```

---

## File Structure

```text
debateiq/
├── index.html          ← Full app: UI + all JavaScript (single file)
├── worker.js           ← Cloudflare Worker proxy (no secrets inside)
├── wrangler.toml       ← Cloudflare CLI config
├── README.md           ← Setup and deployment guide
└── Project_Submission/
    ├── prompt_design.md
    ├── flowchart.md     ← this file
    ├── slide_deck.md
    ├── video_script.md
    └── reflection.md
```
