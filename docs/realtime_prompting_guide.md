# Realtime Prompting Guide

OpenAI's **gpt-realtime** is a speech-to-speech model that powers realtime voice agents with low latency and high fidelity. This guide distills practical prompting strategies for designing production-ready experiences.

> **Tip:** Iterate relentlessly. Small wording tweaks can significantly change model behavior (e.g., replacing “inaudible” with “unintelligible” improved noisy-input handling).

---

## General Guidance

- Favor concise bullet points over long paragraphs.
- Provide concrete examples the model can mimic.
- Remove ambiguity—conflicting or underspecified rules degrade performance.
- Constrain output language explicitly when needed.
- Add “variety” rules to prevent repetitive phrasing.
- Express logic in plain text (e.g., “IF MORE THAN THREE FAILURES THEN ESCALATE”).
- Use CAPITALIZATION sparingly to emphasize critical rules.

---

## Prompt Skeleton

Organize the system prompt into labeled sections. Tailor the structure to your domain by adding or removing sections.

```text
# Role & Objective
# Personality & Tone
# Context
# Reference Pronunciations
# Tools
# Instructions / Rules
# Conversation Flow
# Safety & Escalation
```

---

## Role & Objective

Define the agent’s persona and success criteria. Clear role statements strongly anchor responses.

### Example: Accent-Specific Support
```text
# Role & Objective
You are french quebecois speaking customer service bot. Your task is to answer the user's question.
```

### Example: Character Roleplay
```text
# Role & Objective
You are a high-energy game-show host guiding the caller to guess a secret number from 1 to 100 to win 1,000,000$.
```

---

## Personality & Tone

Specify style, cadence, and length to keep voice responses consistent.

### Baseline Template
```text
# Personality & Tone
## Personality
- Friendly, calm and approachable expert customer service assistant.

## Tone
- Warm, concise, confident, never fawning.

## Length
- 2–3 sentences per turn.
```

### Multi-Emotion Delivery
```text
# Personality & Tone
- Start your response very happy.
- Midway, change to sad.
- End extremely angry.
```

### Pacing (Sounding Fast)
Playback `speed` only accelerates audio; add pacing instructions to truly sound faster.

```text
# Personality & Tone
## Pacing
- Deliver your audio response fast, but do not sound rushed.
- Do not modify the content of your response—only increase speaking speed for the same response.
```

### Language Constraints
Lock the conversation to a target language or design bilingual flows.

```text
## Language
- The conversation will be only in English.
- Do not respond in any other language even if the user asks.
- If the user speaks another language, politely explain that support is limited to English.
```

#### Example: Language Tutor
```text
# Role & Objective
- You are a friendly, knowledgeable voice tutor for French learners.
- Balance immersive French practice with supportive English guidance.

## Language
### Explanations
Use English for grammar, vocabulary, or cultural context.

### Conversation
Speak in French for practice, examples, and dialogue.
```

### Variety Rule
Mitigate repetitive openings.

```text
## Variety
- Do not repeat the same sentence twice.
- Vary your responses so they do not sound robotic.
```

---

## Reference Pronunciations

Give phonetic cues for tricky words. Keep the list short and update based on issues.

```text
# Reference Pronunciations
Pronounce “SQL” as “sequel.”
Pronounce “PostgreSQL” as “post-gress.”
Pronounce “Kyiv” as “KEE-iv.”
Pronounce “Huawei” as “HWAH-way.”
```

### Alphanumeric Pronunciations

Instruct the model to articulate digits individually when relaying critical codes.

```text
# Instructions / Rules
- When reading numbers or codes, speak each character separately (e.g., 4-1-5).
- Repeat exactly the provided number.
```

Embed these requirements inside flow states if only certain phases demand them.

---

## Instruction Quality

High-performing prompts resemble GPT‑4.1/GPT‑5 style: precise, conflict-free, and fully specified.

### Prompt Critique Template
Ask a text model (e.g., GPT‑5) to critique your system prompt.

```text
## Role & Objective
You are a Prompt-Critique Expert.
...
```

The critique returns ambiguity, missing definitions, and conflicting rules along with targeted improvements.

### Prompt Optimization Meta-Prompt
Iterate on problem areas using:

```text
Here's my current prompt...
But I see this issue happening...
```

---

## Handling Unclear Audio

Prevent spurious replies by defining behavior for noise or silence.

```text
## Unclear Audio
- Respond only to clear audio or text.
- If input is unintelligible, ask for clarification in the user’s language.
```

---

## Tools

Explain available tools, usage rules, and alignment with actual API capabilities.

### Tool Inventory
Ensure the prompt’s tool list mirrors the runtime tool configuration.

```json
[
  {
    "name": "lookup_account",
    "description": "Retrieve a customer account using either an email or phone number..."
  },
  {
    "name": "check_outage",
    "description": "Check for network outages affecting a given service address..."
  }
]
```

### Tool Call Preambles
Guide the model to mask latency with short spoken fillers before calling tools.

```text
# Tools
- Before any tool call, say a short line like “I’m checking that now.”
```

For finer control, embed sample phrases directly in each tool description.

### Avoid Redundant Confirmation

```text
# Tools
- When calling a tool, do not ask for user confirmation. Be proactive.
```

### Tool Usage Rules

Define when to use/avoid each tool and how to sequence them. Include failure handling guidance.

### Tool-Level Behavior

Categorize tools by proactiveness, confirmation requirements, or preambles.

```text
## lookup_account — PROACTIVE
Use when verifying identity.

## refund_credit — CONFIRMATION FIRST
Confirmation phrase: “I can issue a credit... would you like me to go ahead?”
```

### Rephrase Supervisor Tool (Responder–Thinker)

For voice responders paired with text planners, instruct the responder to rephrase the thinker’s output into natural speech.

Key rules:

- Use neutral fillers before tool calls.
- After retrieving supervisor output, respond with: opener + one-sentence gist + up to three key details + quick confirmation.
- Read numbers in speech-friendly formats.

Common tools include `answer(question)`, `escalate_to_human()`, and `finish_session()`.

---

## Conversation Flow

Segment the dialogue into phases with goals, instructions, and exit criteria. This reduces drift and keeps conversations focused.

### Linear Flow Template

1. **Greeting** – Identify service, invite goal, confirm customer.
2. **Discover** – Classify issue; gather minimal details.
3. **Verify** – Confirm identity; retrieve account.
4. **Diagnose** – Determine outage vs. local issue.
5. **Resolve** – Apply fix/credit/appointment.
6. **Confirm/Close** – Restate results; invite final questions.

Add sample phrases to each phase for stylistic anchors (with variety rules to avoid repetition).

### Advanced Patterns

- **State Machine JSON**: Encode states, instructions, and transitions for maintainability.
- **Dynamic Flow via `session.update`**: Provide only state-relevant instructions/tools to reduce cognitive load.

---

## Safety & Escalation

Define clear escalation criteria and mandatory language when transferring to humans.

```text
# Safety & Escalation
Escalate if: safety risks, explicit human request, severe frustration, repeated tool failures, or out-of-scope content.
Say: “Thanks for your patience—I’m connecting you with a specialist now.”
Then call `escalate_to_human`.
```

---

## Sample Phrases

Provide brand-aligned anchors for acknowledgements, clarifiers, bridges, empathy, and closers. Remind the model to vary responses.

Embed phrases directly inside relevant flow states when possible.

---

## Summary Checklist

- [ ] Define role, tone, pacing, and language.
- [ ] Include reference pronunciations and digit-reading rules.
- [ ] Align tool descriptions with actual implementations.
- [ ] Provide per-tool behavior, preambles, and sequencing guidance.
- [ ] Structure conversation flow with exit criteria (state machine or dynamic updates).
- [ ] Specify safety/escalation triggers and phrasing.
- [ ] Add sample phrases plus a variety rule.
- [ ] Continuously critique and iterate on the prompt.

Following these patterns helps gpt-realtime deliver expressive, reliable, and compliant voice interactions at scale.
