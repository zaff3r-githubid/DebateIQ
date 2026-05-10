# Reflection — DebateIQ AI Debate Strategist (Completed)

**Student:** [Your Name]
**Course:** AI Practitioner Warm-Up Assignment
**Date:** [Date]

---

## 1. What I built

DebateIQ is a web application that uses Claude claude-sonnet-4-6 to analyse any debate topic through five progressive reasoning layers. Rather than generating a single AI response, it runs a chained pipeline where each layer receives the full output of all previous layers as context — so the reasoning genuinely deepens at every step rather than repeating itself. Users can debate any topic they choose: they either type a custom subject or pick from six presets covering food, technology, society, sport, and education. The app also displays live telemetry (token counts, latency per layer, estimated cost) and generates an AI score card across four dimensions plus personalised coaching feedback. The entire app is a single static HTML file hosted on GitHub Pages, with a Cloudflare Worker proxy keeping the Anthropic API key completely hidden from the browser and the repository.

---

## 2. Why multi-layer reasoning matters

Before this project, I assumed that a well-worded single prompt was enough to get a high-quality AI response — that the quality of the output was mostly about how clearly you asked the question. What this project showed me is that a single prompt, no matter how well crafted, produces a response that has never been challenged. It argues one side without knowing what the other side would say, and it has no mechanism to find its own weaknesses. The moment I added Layer 3 (Counter Argument) and Layer 4 (Self-Critique) and fed those outputs back into the chain, the final answer changed dramatically — it was more nuanced, more honest about trade-offs, and far more persuasive because it had already absorbed the strongest objections. The difference between a single-shot response and a five-layer chain isn't polish — it's the difference between a first draft and a peer-reviewed argument.

---

## 3. The most interesting prompt design decision

The single design decision I am most proud of is the instruction in Layer 3: *"No strawmen — these must be genuinely compelling counter-arguments. Reference the Layer 2 arguments directly and dismantle them."* That one instruction is what separates a real dialectic from AI theatre. Without it, the model tends to produce generic opposition that doesn't actually engage with what Layer 2 said — and then Layer 4's self-critique has nothing meaningful to work with. By forcing Layer 3 to target specific claims from Layer 2, the whole chain becomes a genuine back-and-forth rather than a series of unconnected monologues. A close second is the instruction in Layer 4 that says *"Be intellectually honest — overconfidence is a weakness."* That phrase counteracts the model's natural tendency to defend its own prior output, and it consistently produced the most candid analysis of the whole pipeline.

---

## 4. What the telemetry revealed

The most striking thing the telemetry revealed was how dramatically token usage grows across the layers — and how that growth is entirely predictable once you understand why it happens. Layer 1 processes only the system prompt and the topic. Layer 5 processes the system prompt plus the full output of Layers 1 through 4, which by that point is thousands of words of accumulated reasoning. Watching the per-layer token count in the card footers made that structure visible in a way that reading the prompts alone never would. The other surprise was latency: later layers are not only processing more tokens, they are also generating longer responses, so the wait time compounds. The total pipeline takes longer than I expected — but seeing it in the telemetry bar makes it easy to explain: you are paying for depth, and you can see exactly what that depth costs.

From my live run:

| Metric | Value |
| --- | --- |
| Total input tokens | [fill in from your live run] |
| Total output tokens | [fill in from your live run] |
| Total tokens | [fill in from your live run] |
| Average latency | [fill in] ms |
| Estimated cost | $[fill in] |

One pattern I noticed was that later layers used significantly more tokens than earlier ones. This makes sense because each layer receives all prior layers as context — meaning Layer 5 processes roughly five times the text that Layer 1 does. This is the cost of deep reasoning, and it is a trade-off worth making.

---

## 5. What Layer 4 (Self-Critique) taught me

Layer 4 was the most surprising part of the entire system to build and observe. I expected the AI to hedge — to produce polite, vague criticism that didn't really commit to anything. Instead, when given the explicit instruction to be intellectually honest and the full context of both the arguments and the counter-arguments, it identified weaknesses I hadn't anticipated myself. It pointed out, for example, that the Neapolitan arguments in Layer 2 leaned heavily on tradition and certification as proxies for quality — but that Layer 3 had correctly exposed this as an appeal to authority rather than a functional argument. The model wasn't just identifying that a weakness existed; it was tracing the logical flaw back to where the argument had gone wrong. What struck me most was that Layer 4's output made Layer 5 noticeably better — the final strategy was more credible because it had already acknowledged where the position was vulnerable, rather than pretending those vulnerabilities didn't exist.

---

## 6. Challenges I encountered

The hardest part was designing prompts that forced the model to produce genuinely new reasoning at each layer rather than rephrasing what it had already said. Early versions of Layer 2 would essentially echo Layer 1 with a slight tilt toward the chosen position. The fix was specificity: adding the instruction *"Building directly on that context, construct 3 distinct, evidence-backed arguments"* and the structure Claim → Evidence → Example. That structure gave the model something concrete to produce that it couldn't have produced from Layer 1 alone, because Layer 1 explicitly avoided taking a position. The second significant challenge was securing the API key. Setting up the Cloudflare Worker was straightforward once I understood the architecture, but there was an unfamiliar step involved in storing the key as an encrypted secret rather than a hardcoded value — and verifying that the Worker was live (since Cloudflare shows preview URLs as inactive until you deploy to production). Getting comfortable with that deployment model took more trial and error than the prompt design itself.

---

## 7. What I would do differently or add next time

The improvement I would prioritise first is a side-by-side debate mode — where two instances of the 5-layer pipeline run simultaneously, one defending each position, and a judge layer compares the two final strategies to declare a winner. That would be a much more dramatic demonstration of the reasoning system and would show what the pipeline looks like when it is fully symmetric. Second, I would add cloud persistence using Supabase so that debate history survives across devices and browsers — right now it lives in localStorage, which means it is lost if the user clears their browser. Third, I would invest more in mobile layout. The current design works on desktop and is readable on a phone, but the layer pipeline cards and telemetry bar are clearly designed with a wide screen in mind. A responsive layout that stacks gracefully on smaller screens would make the app genuinely usable in a mobile context.

---

## 8. The SCIPAB framework in practice

SCIPAB gave me a structural backbone for the prompt chain that I would not have had otherwise. Before discovering that mapping, I knew I wanted five layers but I wasn't certain what each one was *for* — they risked collapsing into variations of the same thing. Mapping each layer to a SCIPAB element gave each one a distinct role: Layer 1 (Situation) is forbidden from taking a position, Layer 2 (Position) must commit to a stance, Layer 3 (Challenge) must attack that stance, Layer 4 (Impact) must measure the cost of those attacks, and Layer 5 (Action + Benefit) must synthesise everything into a conclusion worth acting on. That framework is what keeps the layers from repeating each other. It also made the prompt instructions easier to write — once I knew that Layer 4's job was to quantify impact, the instruction "identify 3-4 weaknesses and suggest specific improvements" wrote itself. SCIPAB is a communication framework, not an AI framework, but it turns out to be a very effective template for structuring multi-step reasoning chains.

---

## 9. The most surprising output from the AI

The moment that genuinely stopped me was in Layer 4 during a Neapolitan vs New York Style run. I expected the self-critique to identify weak evidence or overreach in the claims. Instead it produced this:

> "The most significant weakness in the Layer 2 arguments is that they conflate *historical legitimacy* with *present-day superiority*. Arguing that Neapolitan pizza is superior because it is older and more rigorously certified is, at its core, a genetic fallacy — the quality of an argument does not follow from its origin. Layer 3 correctly identified this: the VPN certification standard is a measure of authenticity, not pleasure. A Neapolitan pizza that is certified and mediocre is worse than a New York slice that is technically informal but executed brilliantly. The Layer 2 arguments would have been significantly stronger if they had grounded the superiority claim in measurable sensory or nutritional outcomes rather than provenance."

What made this notable was that the AI had identified a *category error* in its own earlier reasoning — not just a weak piece of evidence, but a structurally flawed type of argument. That is the kind of critique that improves an argument from the inside, not just patches a hole. I had not thought about framing it that way when I wrote the Layer 2 prompt, and the self-critique effectively told me how Layer 2 should have been written differently.

---

## 10. What I would tell someone starting this assignment

Design the context handoff between layers before you write a single prompt. The most important thing in a chained reasoning system is not what each layer *says* — it is how each layer *receives* what came before it. If Layer 3 does not explicitly reference the Layer 2 arguments by name and dismantle them, you do not have a debate chain — you have five separate monologues. Write the instruction that forces that reference first, then build the rest of the prompt around it. The quality gap between a chain where each layer actively builds on the previous one and a chain where layers merely run in sequence is enormous, and it is entirely determined by how you pass and use context.

---

## Final Score (from the app)

Fill this in after running a full debate:

| Dimension | Score /100 |
| --- | --- |
| Argument Strength | [fill in] |
| Evidence Quality | [fill in] |
| Logical Coherence | [fill in] |
| Persuasiveness | [fill in] |
| **Overall** | **[fill in]** |

**AI Verdict:** [paste the one-sentence verdict from the Score Card]

**Coach's top recommendation:** [paste the recommendation from the Coach Feedback]
