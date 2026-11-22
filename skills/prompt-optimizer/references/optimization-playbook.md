# Optimization Playbook

Practical detail for diagnosing and rewriting prompts. Load this when the
draft is non-trivial or the user wants to understand *why* changes help.

## Diagnostic rubric

Score each dimension as OK / weak / missing, then fix only the weak or missing
ones. Record the symptom, not a lecture.

| Dimension | Check | Minimal fix |
| --- | --- | --- |
| Intent | Is the goal one unambiguous sentence? | Lead with a direct objective verb ("Write," "Classify," "Extract") |
| Audience/goal | Who is this for and why? | Add one short line of purpose/audience context |
| Context | Is all required data present? Is it near the top? | Add `{{PLACEHOLDER}}` for missing data; move long docs above the query |
| Structure | Are instructions, data, and examples distinguishable? | Add headings or XML tags only where boundaries matter |
| Examples | Would showing beat describing? | Add 1–5 short, relevant, diverse `<example>` blocks |
| Constraints | Are length, format, style, dos/don'ts explicit and positive? | Convert prohibitions into positive instructions |
| Output format | Can the consumer parse the result? | Name the format; add a JSON schema for machine output |
| Reasoning | Does the task need multi-step thinking? | Use built-in thinking; add CoT only for general models on hard tasks |
| Decomposition | Is more than one job being asked at once? | Split into a chain with explicit handoffs |
| Robustness | What happens when information is missing? | Add "say 'I don't know' / ask a clarifying question" |
| Model fit | Is the prompt tuned for the target model? | Simplify for reasoning models; add explicit steps for general ones |
| Redundancy | Are instructions repeated or contradictory? | Keep one source of truth |

## Clarifying-question bank

Ask only questions whose answers change the rewrite. Skip the rest and state an
assumption instead.

- What model will run this prompt? (Reasoning vs. general changes the structure.)
- What should the output look like — prose, table, JSON, code? Who consumes it?
- Who is the audience and what does success look like?
- Is there reference material or private data the model must use?
- Are there hard constraints — length, tone, style, forbidden content?
- Do you want a single prompt or a multi-step workflow?
- Is there an example of a good result you can point to?

For a **test/evaluate** request, ask specifically:

- What exactly is the artifact to test (name, code, config, plugin definition)?
- What input does it take, how does it process it, and what should it output?
- What concrete sample inputs should be run, and what is the expected result for each?

## Transformation patterns

| Pattern | Apply when | Operation |
| --- | --- | --- |
| Objective-first | Any prompt | Replace the opening with a single clear task sentence |
| Persona-focus | Tone/expertise matters | Add one sentence of role and audience |
| Context-first | Long reference material | Move context to the top; end with the query |
| Delimiting | Mixed instructions/data/examples | Wrap each type in consistently named tags |
| Example-driven | Format/tone is hard to describe | Add input/output examples |
| Positive-framing | Many "don'ts" | Rewrite each don't as a do |
| Schema-binding | Machine consumes output | Append a JSON schema or exact format spec |
| Decomposition | Several jobs at once | Break into chained steps with handoff names |
| Guardrailing | Missing info is likely | Add "I don't know" and quote-grounding rules |
| Simplification | Prompt is bloated or targets a reasoning model | Remove tags/recaps/redundant steps |
| Clarify-first | Core task, artifact, or success criteria are missing | Ask 1–3 questions and stop; do not emit a placeholder template |

## Handling missing information

Use this decision rule before rewriting anything:

| What is missing | Action |
| --- | --- |
| Runtime data the user will paste in later (document, dataset, raw text) | Keep a clearly labeled placeholder, e.g. `{{PASTE_CONTRACT_HERE}}` |
| Secondary detail (tone, length, audience) | Assume it, state the assumption, proceed |
| Core task, artifact to test, expected behavior, or test inputs | Ask 1–3 concrete questions and stop — do not deliver a rewrite |

**Anti-pattern (never do this):**

```text
Plugin: {{PLUGIN_NAME_OR_DESCRIPTION_OR_CODE}}
Expected behavior: {{WHAT_THE_PLUGIN_SHOULD_DO}}
Test inputs: {{SAMPLE_INPUTS_TO_RUN}}
If a test input or the expected behavior is missing, respond with exactly
"Not enough information to test" instead of guessing.
```

This fails twice over: the placeholders hold the *core task definition* (not
runtime data), and the "stop if missing" rule guarantees the prompt halts
because those placeholders will still be unfilled. A prompt cannot both require
values and forbid proceeding without them.

**Correct handling for the same request:**

> To test the plugin, I need three things: (1) the plugin itself — its name, code,
> or configuration; (2) what it takes as input and should produce as output; and
> (3) the sample inputs to run, with the expected result for each. Please provide
> those and I'll run every case and report input / expected / actual / PASS or FAIL.

Only after those answers arrive does a rewrite (or the test itself) make sense.

## Worked examples

### Example 1 — Vague content request

**Before:**
> Write something about our new product for social media.

**After:**
> You are a social media copywriter for a B2B analytics startup.
>
> <context>
> Product: {{PRODUCT_NAME}} — a dashboard that tracks customer retention.
> Audience: marketing managers at mid-sized SaaS companies.
> Goal: drive sign-ups for a 14-day free trial.
> Tone: confident and concise, no hype.
> </context>
>
> <task>
> Write three distinct LinkedIn posts (60–90 words each). Each post must:
> 1. Open with a one-line observation about retention.
> 2. Describe one concrete benefit of {{PRODUCT_NAME}}.
> 3. End with a call to action for the free trial.
> </task>
>
> Output as a numbered list, plain text only, no emoji, no preamble.
>
> Recap: three posts, 60–90 words each, each with observation → benefit → CTA.

**Why:** Added role, context, count, structure, and positive constraints; removed
ambiguity about "something."

### Example 2 — Need for format consistency

**Before:**
> Extract the names and dates from this text and give me the result.

**After:**
> Extract every person name and associated date from the text in `<input>`.
>
> <input>
> {{PASTE_TEXT_HERE}}
> </input>
>
> Return a JSON array of objects with exactly these keys: "name" (string) and
> "date" (ISO 8601 string). Include only names that have a date; if a name has
> multiple dates, output one object per date. If nothing is found, return `[]`.
> Do not include explanations, markdown fences, or a preamble.

**Why:** JSON schema, inclusion rule, empty case, and output restrictions
replace a vague "result."

### Example 3 — Over-structured prompt for a simple task (a reasoning model)

**Before:**
> <ROLE>You are an expert</ROLE><TASK>Calculate 17*24</TASK>
> <INSTRUCTIONS>Think step by step and show all reasoning...</INSTRUCTIONS>

**After:**
> What is 17 × 24? Answer with the number only.

**Why:** For a reasoning model, arithmetic needs neither a persona nor mandatory
visible chain-of-thought. Simpler is faster and less error-prone.

### Example 4 — Multi-step task

**Before:**
> Read this contract, find any risky clauses, and rewrite them to be fairer.

**After:**
> Run this as three separate steps and return all three results:
>
> Step 1 — `<extract>`: List every clause in `<contract>` that limits liability,
> auto-renews, or restricts termination. Quote each clause verbatim.
>
> Step 2 — `<review>`: For each extracted clause, explain the risk to the client
> in one sentence and rate it high / medium / low.
>
> Step 3 — `<revise>`: Rewrite each high or medium clause in plain language that
> is fairer to the client while preserving the agreement's intent.
>
> <contract>{{PASTE_CONTRACT}}</contract>
>
> If a step cannot be completed from the text, say "Not present in the contract."

**Why:** Separates extraction, judgment, and rewriting into a chain with explicit
handoffs and a missing-information rule.

## Evaluation guidance

Always pair the rewrite with a lightweight test plan:

1. **Define success** in one measurable sentence (e.g. "Returns valid JSON with
   correct names and dates for 9 of 10 test documents").
2. **Provide 2–3 test inputs**: a typical case, an edge case, and a case where
   information is missing.
3. **Compare** the original and optimized prompts on the same inputs; keep the
   optimized version only if it measurably improves the result.
4. **Iterate** on the dimension that failed, not the whole prompt.

For high-stakes or repeated use, build a small fixture set and run it before
each prompt change; consider automatic prompt optimization frameworks when many
prompt variants must be searched against a metric.
