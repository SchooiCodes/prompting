# Prompt Engineering — Collected Research

A consolidated research brief on how large language models (LLMs) work, why
prompts matter, and the techniques used to write high-performing prompts.

Collected 2026-11-21 from primary documentation published by OpenAI, Anthropic,
Google, and NVIDIA. See [sources.md](sources.md) for the full source list.

> Scope note: prompt engineering is an empirical craft, not a fixed recipe. The
> official guides agree that models, snapshots, and task types behave differently.
> The reliable method is: state intent clearly, supply the right context and
> examples, structure the input, then test and iterate against success criteria.

---

## 1. What prompt engineering is

- **OpenAI:** "Prompt engineering is the process of writing effective instructions
  for a model, such that it consistently generates content that meets your
  requirements." Because model output is non-deterministic, it is "a mix of art
  and science," but techniques and best practices produce good results
  consistently.
- **Anthropic:** "The craft of structuring instructions to get better outputs from
  AI models. It's how you phrase queries, specify style, provide context, and
  guide the model's behavior to achieve your goals." A vague instruction and a
  well-crafted prompt can be the difference between generic output and exactly
  what you need.
- **Google:** "Prompt design is the process of creating prompts that elicit the
  desired response from language models." The iterative loop of repeatedly
  updating prompts and assessing responses is called *prompt engineering*.

Key framing from Anthropic: before engineering a prompt, define (1) clear success
criteria, (2) a way to empirically test against them, and (3) a first draft to
improve. Not every failure is a prompting problem — latency and cost are often
better solved by choosing a different model.

---

## 2. How the model actually works (why prompts have leverage)

### Tokens
- Models do not read words directly. Input is split into **tokens** — small units
  of data (a short word can be one token; a long word is split, e.g. `darkness`
  → `dark` + `ness`). Each token maps to a number.
- During inference, the prompt (text, image, audio, etc.) is translated into a
  sequence of input tokens. The model processes them, generates output tokens,
  then converts those back into the expected format. *(NVIDIA)*
- Model pricing and speed are commonly measured per token; the number of tokens
  used in training correlates with model quality (pretraining scaling law).
  *(NVIDIA)*

### Prediction, not retrieval
- LLMs are trained on vast text and learn patterns/relationships between units of
  language. Given text (the prompt), they predict what is likely to come next —
  "like a sophisticated autocompletion tool." Every design choice in a prompt
  changes what the model is likely to predict next. *(Google)*
- This is why the *wording, ordering, and formatting* of a prompt matter: they are
  part of the context the next-token prediction is conditioned on.

### Context window
- A model can only consider a limited amount of data per request; this memory
  limit is the **context window**, measured in tokens. Sizes range from the low
  100k range up to ~1 million tokens for newer models. *(OpenAI)*
- Long documents should be placed near the top of the prompt, above the query and
  instructions; putting the query at the end can improve response quality by up
  to ~30% in tests with complex, multi-document inputs. *(Anthropic)*

### Transformer / attention (conceptual)
- Modern LLMs use the **transformer** architecture, in which an **attention**
  mechanism lets the model weigh the relevance of every token in the context
  relative to every other token when producing each next token. This is how a
  prompt's instructions, examples, and context can influence a response anywhere
  in the output. (Background: IBM explainers; the mechanism is why relevant
  context and examples placed in the prompt steer generation.)

### Non-determinism and instruction hierarchy
- Output is non-deterministic: identical prompts can yield different results.
  Treat prompting as an iterative, test-driven process. *(OpenAI)*
- Instructions carry different authority. In OpenAI's chain of command:
  `developer`/system instructions > user messages > assistant messages. System
  (or developer) prompts set persistent behavior; user messages carry the
  request. *(OpenAI model spec)*
- Different model *families and snapshots* respond differently to the same
  prompt; techniques must be re-validated when the model changes. *(OpenAI)*

### Reasoning models vs. general (GPT-style) models
- **Reasoning models** (e.g. o-series) generate an internal chain of thought and
  excel at complex, multi-step planning and ambiguous information. They are
  slower and more expensive. Prompt them with **simple, direct instructions**;
  do **not** ask them to "think step by step" or show reasoning — they already do
  it internally. Use delimiters and clear section structure. *(OpenAI)*
- **General/GPT-style models** are fast and cost-efficient but benefit from
  **explicit** instructions about how to accomplish the task. *(OpenAI)*
- Analogy from OpenAI: a reasoning model is like a senior co-worker you give a
  goal to; a GPT model is like a junior co-worker who needs explicit steps.

---

## 3. Components of a prompt

Google's taxonomy — a prompt can contain:

| Component | Required? | Purpose |
| --- | --- | --- |
| **Task** | Yes | The question or instruction the model must answer |
| **System instructions** | Optional | Persistent behavior, role, tone, goals, constraints |
| **Few-shot examples** | Optional | Show the desired pattern, format, tone, or style |
| **Contextual information** | Optional | Data the model needs (often outside its training) |

Prompt quality depends on **content** (is all relevant information present?) and
**structure** (is it labeled, ordered, and delimited so the model can parse it?).
*(Google)*

Google's sample prompt template (persona + instructions + optional constraints,
context, output format, few-shot examples, recap):

```text
<OBJECTIVE_AND_PERSONA>
You are a [persona, e.g. "math teacher" or "automotive expert"].
Your task is to...
</OBJECTIVE_AND_PERSONA>
<INSTRUCTIONS>
To complete the task, follow these steps:
1. ...
2. ...
</INSTRUCTIONS>
<CONSTRAINTS>
Dos and don'ts: 1. Do ... 2. Don't ...
</CONSTRAINTS>
<CONTEXT>The provided context</CONTEXT>
<OUTPUT_FORMAT>1. ... 2. ...</OUTPUT_FORMAT>
<FEW_SHOT_EXAMPLES>
Example #1 — Input: ... / Output: ...
</FEW_SHOT_EXAMPLES>
<RECAP>
Re-emphasize key aspects: constraints, output format, etc.
</RECAP>
```

---

## 4. Core techniques

### 4.1 Be clear, explicit, and specific
- State exactly what you want; do not rely on the model to infer it. Use simple,
  unambiguous language. *(Anthropic, Google)*
- **Clarity test (Anthropic):** show the prompt to a colleague with minimal
  context and ask them to follow it. If they'd be confused, the model will be too.
- Specificity means including: clear constraints (word count, format, timeline),
  relevant context (audience, goal), desired output structure, and requirements
  or restrictions. *(Anthropic)*
- Example: "Create a meal plan for a Mediterranean diet" → "Design a Mediterranean
  diet meal plan for pre-diabetic management. 1,800 calories daily, emphasis on
  low glycemic foods. List breakfast, lunch, dinner, and one snack with complete
  nutritional breakdowns."
- **Tell the model what to do, not what not to do.** Instead of "Do not use
  markdown," use "Write in smoothly flowing prose paragraphs." *(Anthropic)*

### 4.2 Give the model a role / persona
- Setting a role in the system prompt focuses behavior and tone. Even one sentence
  helps: "You are a helpful coding assistant specializing in Python."
  *(Anthropic, Google)*

### 4.3 Structure with delimiters and XML tags
- Wrap distinct content types in tags (`<instructions>`, `<context>`, `<input>`)
  so the model can tell them apart; use consistent, descriptive names and nest
  tags for hierarchy. Especially valuable for complex prompts mixing instructions,
  data, and examples. *(Anthropic, OpenAI, Google)*
- OpenAI recommends delimiters such as markdown, XML tags, and section titles to
  mark distinct parts of the input — especially for reasoning models.
- **Modern caveat (Anthropic):** for simple prompts, clear headings, whitespace,
  and explicit language work as well with less overhead. Use XML where content
  boundaries truly matter.

### 4.4 Provide few-shot examples
- Show, don't just tell. Examples are one of the most reliable ways to steer
  output format, tone, and structure. Include **3–5** examples for best results;
  start with one (one-shot) and add more only if needed. *(Anthropic)*
- Make examples **relevant** (mirror the real use case), **diverse** (cover edge
  cases so the model doesn't latch onto unintended patterns), and **structured**
  (wrap in `<example>` / `<examples>` tags). *(Anthropic)*
- Modern models pay very close attention to example details — ensure examples
  embody exactly the behavior you want and omit patterns you want to avoid.
  *(Anthropic)*
- Google: few-shot examples are especially effective for style/tone and
  customizing behavior.

### 4.5 Ask for reasoning when it helps (chain of thought)
- Requesting step-by-step reasoning before the answer helps on complex analytical
  tasks. Ask for the final answer in `<answer>` tags for easy extraction.
  *(Anthropic, Google)*
- **Prefer built-in thinking/extended-thinking when available** over manual CoT.
  On reasoning models, do NOT add "think step by step" — it is unnecessary.
  *(Anthropic, OpenAI)*
- Google warns against: incorrect CoT ordering (final structured answer before
  reasoning in examples), conflicting internal references, and non-linear logic.

### 4.6 Chain complex tasks (prompt chaining)
- Break a complex task into sequential prompts/steps, each doing one thing well,
  feeding output into the next. Trades latency for higher accuracy. *(Anthropic,
  Google)*
- Most common pattern: **self-correction** — generate a draft → review against
  criteria → refine based on the review. Each step is a separate call so you can
  log, evaluate, or branch. *(Anthropic)*

### 4.7 Specify output format / structured outputs
- State the response format explicitly: JSON, table, Markdown, bullets, word
  count, etc. *(Google)*
- For guaranteed machine-readable output, use **Structured Outputs** with a JSON
  schema rather than relying on prose instructions. *(OpenAI)*
- **Prefilling** (starting the assistant response) and examples help control
  format; explicitly ask to "skip the preamble and get straight to the answer."
  *(Anthropic)*

### 4.8 Include relevant context and ground it
- Add context the model can't know from training (proprietary data, specific
  resources). Constrain the response to those resources. *(OpenAI)*
- For long documents, ask the model to **quote relevant passages first**, then
  perform the task — this focuses attention on relevant content. *(Anthropic)*

### 4.9 Guard against hallucinations
- Explicitly give permission to say **"I don't know"** when uncertain.
  *(Anthropic)*
- Encourage source verification across multiple sources for research tasks.
  *(Anthropic)*

### 4.10 Recap / reinforce key constraints
- End the prompt with a concise recap of the most important constraints and the
  required output format. *(Google)*

### 4.11 Control decoding parameters
- **Temperature** (and related settings) control randomness: lower values give
  more deterministic, focused output; higher values give more variety. Choose
  according to the task (factual extraction vs. creative brainstorming).

---

## 5. The optimization loop (measuring and improving prompts)

Anthropic's recommended starting point before prompt engineering:

1. A clear definition of **success criteria** for the use case.
2. Ways to **empirically test** against those criteria (evals).
3. A **first draft prompt** to improve.

OpenAI's practical guidance for production:

- Keep prompt builders in a small module near the feature they support.
- Use typed function arguments or schemas for dynamic values (customer data,
  files, task options).
- Put reusable guidance in the first `developer` message; use turn-specific
  `instructions` for the current request.
- Add representative fixtures, tests, and evaluation checks **before** changing
  production prompts.
- Roll out prompt changes through deployment systems / feature flags for staged
  releases.

Google: rigorous testing and evaluation remain crucial; there is no single right
way to design a prompt.

### Meta-prompting / automatic optimization
- **Meta-prompting** means using a model to generate or rewrite prompts. Anthropic
  publishes a *prompt generator* ("metaprompt") recipe in the Claude Cookbook for
  producing a first-draft prompt from a description of the task.
- **DSPy** is a framework for *programming* (rather than hand-writing) language
  model pipelines, including **automatic prompt optimization**: optimizers tune
  instructions and few-shot examples against a metric, using techniques such as
  bootstrap examples, few-shot sampling, and Bayesian-style search (e.g.
  MIPROv2). The core idea: treat prompts as parameters to be optimized against
  an objective, not just prose to be tweaked by hand.
  (Source: DSPy documentation — prompt optimization / optimizers.)

**Practical takeaway:** the highest-leverage workflow is
`draft → evaluate → diagnose → rewrite → re-evaluate`, with the diagnosis guided
by a checklist of the dimensions in this document.

---

## 6. Common pitfalls and how to fix them

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| Generic or off-target output | Vague request | Make the instruction explicit and specific |
| Inconsistent response format | Format only described, not shown | Add few-shot examples or prefill the response |
| Task unreliable / messy | Too many things in one prompt | Break into chained steps, one job each |
| Unnecessary preamble/hedging | No instruction about style | "Skip the preamble; answer directly" |
| Made-up information | No permission to be uncertain | Allow "I don't know"; request source quotes |
| Suggestions when implementation wanted | Action ambiguity | Be explicit: "Change this function," not "Can you suggest..." |
| Redundancy / drift between edits | Repeated instructions | Remove duplicated instructions; keep one source of truth |
| Prompt works on model A, fails on B | Model/snapshot differences | Re-test and tune per model; simplify for reasoning models |

*(Anthropic best-practices troubleshooting; Google writing issues.)*

---

## 7. Condensed checklist for an optimized prompt

1. **Intent:** Is the goal stated in one unambiguous sentence?
2. **Role:** Would a persona/tone focus the output? (Add it if so.)
3. **Context:** Is all necessary information present, and is it placed up top?
4. **Structure:** Are instructions, context, examples, and input clearly delimited?
5. **Examples:** Would 1–5 relevant, diverse examples steer format/tone better than prose?
6. **Constraints:** Are dos/don'ts, length, and boundaries explicit and positive?
7. **Format:** Is the exact output format specified (schema/JSON if machine-read)?
8. **Reasoning:** Does the task need step-by-step thinking, or built-in thinking? (Don't force CoT on reasoning models.)
9. **Decomposition:** Is the task simple enough for one prompt, or should it be chained?
10. **Robustness:** Is the model allowed to say "I don't know"? Are sources grounded?
11. **Model fit:** Is the prompt tuned for this model family (reasoning vs. general)?
12. **Evaluation:** Is there a test set / success criterion to verify improvement?

---

## 8. Sources

See [sources.md](sources.md) for the full URL list and retrieval notes.
Primary sources:

- OpenAI — *Prompt engineering guide* (developers.openai.com/api/docs/guides/prompt-engineering)
- OpenAI — *Reasoning best practices* (developers.openai.com/api/docs/guides/reasoning-best-practices)
- OpenAI — *Model Spec* (chain of command / instruction priority)
- Anthropic — *Prompt engineering overview* (platform.claude.com/docs/.../prompt-engineering/overview)
- Anthropic — *Claude prompting best practices* (platform.claude.com/docs/.../claude-prompting-best-practices)
- Anthropic — *Best practices for prompt engineering* (claude.com/resources/articles/best-practices-for-prompt-engineering)
- Anthropic — *Prompt generator / metaprompt* recipe (Claude Cookbook)
- Google — *Introduction to prompt design* and *Prompt design strategies* (docs.cloud.google.com / Gemini Enterprise Agent Platform)
- NVIDIA — *AI tokens explained* (blogs.nvidia.com/blog/ai-tokens-explained)
- DSPy — *Optimizers / prompt optimization* documentation (dspy.ai)
- IBM — *Transformer models* explainers (ibm.com/think)
