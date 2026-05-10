# Prompt Design Document

**Project:** DebateIQ — AI Debate Strategist
**Course:** AI Practitioner Warm-Up Assignment
**Demo Topic:** Neapolitan vs New York Style Pizza (default — app supports any topic)
**Model:** claude-sonnet-4-6 via Anthropic API

---

## Overview

DebateIQ uses a **5-layer chained reasoning system** where each prompt receives all previous layers as context. This ensures the AI genuinely builds new reasoning at every step rather than repeating itself — which is the core requirement of the assignment.

The system is **topic-agnostic** — users can debate anything by typing any subject into the topic field, or choose from preset topics covering food, technology, society, sport, and education. The pizza topic is the default for the assignment demo.

The framework behind the prompt design is **SCIPAB** (Situation → Challenge → Impact → Position → Action → Benefit), mapped onto the 5 layers as follows:

| SCIPAB Element | Debate Layer | Purpose |
| --- | --- | --- |
| Situation | Layer 1 — Context Analysis | Establish the landscape objectively |
| Position | Layer 2 — Argument Builder | Take and defend a clear stance |
| Challenge | Layer 3 — Counter Argument | Surface the strongest opposition |
| Impact | Layer 4 — Self-Critique | Assess where the position is weak |
| Action + Benefit | Layer 5 — Final Strategy | Synthesise into an informed verdict |

The logical flow the system follows:
**YOU THINK** (setup topic + position) → **AI THINKS** (5 layers) → **YOU MEASURE** (telemetry) → **YOU IMPROVE** (coach feedback)

---

## Prompt Variables

Every prompt uses the same set of injectable variables, which the user configures before running:

| Variable | Example Value (Pizza Demo) | What it does |
| --- | --- | --- |
| `{persona}` | Domain Expert | Sets the AI's voice and expertise lens |
| `{tone}` | passionate and authoritative | Controls rhetorical style |
| `{source}` | peer-reviewed academic research | Grounds arguments in a specific knowledge base |
| `{topic}` | Neapolitan vs New York Style Pizza | The debate subject — can be anything |
| `{position}` | Neapolitan Pizza is the superior style | The stance being argued |
| `{word_count}` | 200-250 | Controls response length |

---

## Layer 1 — Context Analysis

**Purpose:** Objectively map the debate. No position is taken yet. This establishes shared ground and surfaces hidden assumptions that deeper layers can exploit or defend.

**Design choices:**

- Explicitly instructs "Do NOT take a position" to prevent premature bias
- Asks for exactly 4-5 key factors to force prioritisation over vague listing
- The word "objectively" is deliberate — it primes the model for neutral analysis
- Works identically for any topic: pizza, AI ethics, remote work, sport — the prompt structure is universal

**Prompt:**

```text
You are writing as a {persona}. Tone: {tone}. Draw on the expertise of {source}.

Layer 1 — Context Analysis.
Topic: "{topic}". Position to examine: "{position}".

Objectively map this debate. Identify exactly 4-5 key factors that define the debate landscape.
Surface hidden assumptions on BOTH sides. Do NOT take a position yet — stay neutral and analytical.
{word_count} words maximum.
```

**SCIPAB mapping:** Situation — establishes current reality of the debate, regardless of topic.

---

## Layer 2 — Argument Builder

**Purpose:** Take a clear position and build 3 structured arguments with Claim → Evidence → Example for each. Receives Layer 1 as context so arguments respond to the actual landscape identified, not a generic stance.

**Design choices:**

- "Building directly on that context" forces the model to acknowledge Layer 1 output
- Claim → Evidence → Example structure mirrors formal debate format required by the rubric
- "Be specific — no vague generalities" combats the model's tendency toward platitudes
- The `{source}` variable grounds evidence in a credible knowledge base appropriate to the topic

**Prompt:**

```text
You are writing as a {persona}. Tone: {tone}. Draw on the expertise of {source}.

Layer 2 — Argument Builder.
Topic: "{topic}". You are DEFENDING: "{position}".

Context established in Layer 1:
{layer_1_output}

Building directly on that context, construct 3 distinct, evidence-backed arguments defending
the position. For each: state the Claim, provide Evidence, give a concrete Example.
Be specific — no vague generalities.
{word_count} words maximum.
```

**SCIPAB mapping:** Position — proposed solution stated with specificity.

---

## Layer 3 — Counter Argument

**Purpose:** Generate the strongest possible opposition. Receives Layers 1 and 2, and is explicitly instructed to dismantle the Layer 2 arguments directly — making the counter-arguments genuinely targeted, not generic.

**Design choices:**

- "No strawmen" is a critical instruction that forces intellectual honesty
- "Reference the Layer 2 arguments directly and dismantle them" creates genuine dialectic tension
- Including both prior layers gives the model full context to mount a credible challenge
- This layer is the hardest test of prompt quality — weak prompts produce weak opposition

**Prompt:**

```text
You are writing as a {persona}. Tone: {tone}. Draw on the expertise of {source}.

Layer 3 — Counter Argument.
Topic: "{topic}". Now CHALLENGE: "{position}".

What was established so far:
Layer 1 (Context): {layer_1_output}
Layer 2 (Arguments for position): {layer_2_output}

Now generate the 3 strongest possible opposing arguments. No strawmen — these must be
genuinely compelling counter-arguments. Reference the Layer 2 arguments directly and dismantle them.
{word_count} words maximum.
```

**SCIPAB mapping:** Challenge — the complication or problem with the position.

---

## Layer 4 — Self-Critique

**Purpose:** Turn the lens inward and honestly identify weaknesses in the Layer 2 arguments, with specific reference to which Layer 3 counter-arguments land hardest. This layer demonstrates intellectual maturity and balanced reasoning.

**Design choices:**

- "Honestly" and "Be intellectually honest — overconfidence is a weakness" are deliberate prompts that counteract the model's tendency to defend its own prior output
- The full chain (Layers 1-3) is provided so critique can reference specific prior claims
- Asking for improvement suggestions prevents pure negativity
- This is the layer that most clearly demonstrates the value of multi-step reasoning

**Prompt:**

```text
You are writing as a {persona}. Tone: {tone}. Draw on the expertise of {source}.

Layer 4 — Self-Critique.
Topic: "{topic}". Position: "{position}".

Full reasoning chain so far:
Layer 1 (Context): {layer_1_output}
Layer 2 (Arguments): {layer_2_output}
Layer 3 (Counter-arguments): {layer_3_output}

Honestly identify 3-4 weaknesses in the Layer 2 arguments, acknowledging which counter-arguments
from Layer 3 land hardest. Suggest specific improvements for each weakness.
Be intellectually honest — overconfidence is a weakness.
{word_count} words maximum.
```

**SCIPAB mapping:** Impact — quantifying the cost of the position's weaknesses.

---

## Layer 5 — Final Strategy

**Purpose:** Synthesise all four prior layers into a balanced, nuanced conclusion. This is the most important layer and receives the largest token budget. It must end with a definitive "Final Verdict:" sentence.

**Design choices:**

- Full chain provided — the model has maximum context for synthesis
- "State the conditions under which each side wins" prevents a lazy one-sided conclusion
- "Bold" and "definitive and memorable" instruct against hedging vague endings
- This layer gets a higher max_tokens limit (1400 vs 1200) to allow for richer synthesis

**Prompt:**

```text
You are writing as a {persona}. Tone: {tone}. Draw on the expertise of {source}.

Layer 5 — Final Strategy.
Topic: "{topic}". Original position: "{position}".

Complete reasoning chain:
Layer 1 (Context): {layer_1_output}
Layer 2 (Arguments): {layer_2_output}
Layer 3 (Counter-arguments): {layer_3_output}
Layer 4 (Self-critique): {layer_4_output}

Synthesise all layers into a refined, nuanced final position. Acknowledge where the opposition
is strongest. State the conditions under which each side wins. End with a bold
"Final Verdict:" sentence that is definitive and memorable.
{word_count} words maximum.
```

**SCIPAB mapping:** Action + Benefit — what to conclude and why it matters.

---

## Prompt 6 — Debate Score Card

**Purpose:** After all 5 layers complete, a separate judge call returns structured JSON scores across four dimensions. JSON output is specified explicitly to enable reliable parsing.

**Design choices:**

- Strict JSON-only response instruction prevents prose contamination
- Four distinct dimensions map to different aspects of debate quality
- Overall score is requested alongside individual scores for a holistic view
- Works for any topic — the judge evaluates argument quality, not subject matter

```text
You are a debate judge. Score this debate strictly and fairly.
Topic: "{topic}", Position: "{position}"
[All 5 layers provided]

Score on: Argument Strength, Evidence Quality, Logical Coherence, Persuasiveness (each 0-100).
Respond ONLY with valid JSON: {"strength":85,"evidence":72,"logic":90,"persuasion":78,
"overall":81,"verdict":"...","recommendation":"..."}
```

---

## Prompt 7 — Debate Coach Feedback

**Purpose:** Personalised coaching based on the debater's actual words and score. References specific phrases from the debate to make feedback concrete rather than generic.

**Design choices:**

- Structured markdown output with named sections ensures consistent rendering
- "Reference their actual words" prevents generic platitudes
- The score is passed in so feedback can acknowledge what was measured
- Topic-agnostic — the coach evaluates debating skill, not domain knowledge

---

## Prompts 8, 9, 10 — Live Debate Mode

These three prompts handle the real-time spoken debate on any topic:

| Prompt | Purpose | Key Design Choice |
| --- | --- | --- |
| 8 — Opening | AI introduces the debate | "2-3 sentences" keeps it punchy for voice |
| 9 — Counter | AI responds to each user turn | Last 6 messages of history passed; "3-5 sentences max" keeps spoken delivery natural |
| 10 — Judge | AI declares a winner after debate ends | JSON output for structured result rendering |

---

## Available Personas

The user selects from six general-purpose debate personas that work across any topic:

| Persona | AI Value |
| --- | --- |
| Domain Expert | Deep specialist knowledge and authoritative framing |
| Critical Analyst | Sharp, investigative, evidence-focused |
| Devil's Advocate | Challenges every assumption — maximises tension |
| Science & Data | Evidence-driven, quantitative, research-backed |
| Philosopher | Ethical and moral dimensions, first principles |
| Public Advocate | Represents real-world human impact and lived experience |

---

## Available Knowledge Sources

Six general knowledge sources ground arguments in different epistemological traditions:

| Source | Best used for |
| --- | --- |
| Academic Research | Science, policy, health, technology debates |
| Quality Journalism | Current affairs, culture, social issues |
| Industry Data | Business, economics, market-driven topics |
| Historical Record | Long-running debates with historical precedent |
| Expert Consensus | Professional domains, standards bodies |
| Public Opinion | Consumer, social, and democratic debates |

---

## Telemetry Instrumentation

Every API call measures and accumulates:

| Metric | How captured |
| --- | --- |
| Input tokens | `data.usage.input_tokens` from API response |
| Output tokens | `data.usage.output_tokens` from API response |
| Total tokens | Running sum of input + output |
| Latency | `Date.now()` delta around each `fetch()` call |
| Estimated cost | Input × $3/M + Output × $15/M (Sonnet 4.6 pricing) |

Per-layer token counts are displayed in each layer card footer. Cumulative totals appear in the telemetry bar above the pipeline and in the Insights panel below.
