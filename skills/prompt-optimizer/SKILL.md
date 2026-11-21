---
name: prompt-optimizer
description: Analyze, diagnose, and rewrite user-supplied prompts into precise, high-performing prompts using established prompt-engineering best practices. This skill should be used whenever a user asks to optimize, improve, refine, rewrite, debug, or strengthen a prompt for any AI model or task, or when a draft prompt is vague, bloated, ambiguous, or under-specified and the user wants it to work better.
---

# Prompt Optimizer

## Purpose

Turn a user's rough or under-performing prompt into a clear, effective prompt that
reliably produces the intended output. Combine diagnosis (what is weak and why)
with a concrete rewrite, brief rationale, and a way to verify the improvement.

## When to use

- The user provides a draft prompt and asks to improve, optimize, or rewrite it.
- A prompt produces vague, inconsistent, or incorrect output and the user wants
  to know how to fix it.
- The user wants a prompt built from a rough description of a task.

Do not use for general conversations, or when the user asks to *execute* a prompt
rather than improve it.

## Core principles

1. **Preserve intent.** Optimize the wording and structure, never silently change
   the user's goal, audience, or constraints.
2. **Be proportional.** A simple factual question needs a short, direct prompt.
   Add structure, examples, and constraints only where they earn their place.
3. **Show, do not just tell.** Prefer a few relevant examples over paragraphs of
   abstract guidance when format or tone matters.
4. **Match the model.** Reasoning models need short, direct instructions and no
   manual chain-of-thought; general models benefit from explicit, step-by-step
   instructions. Ask which model will run the prompt when it changes the answer.
5. **Make the result testable.** An optimized prompt should come with success
   criteria and a couple of representative test inputs.

## Workflow

### Step 1 — Capture the goal and the draft

Identify: the user's intent, the draft prompt (if any), the target model/family,
and the desired output. If any are missing, make a reasoned assumption and state
it; ask a clarifying question only when the answer would materially change the
rewrite. Limit clarifications to 1–3 high-value questions.

### Step 2 — Diagnose

Scan the draft against the diagnostic rubric in
`references/optimization-playbook.md`. For each weak dimension, note the symptom
and the minimal fix. Keep the diagnosis short — the rewrite is the deliverable.

Common patterns:

| Symptom | Fix |
| --- | --- |
| Vague request | State the task in one explicit sentence |
| Missing audience/goal/context | Add only the context the model cannot know |
| Format comes out wrong | Add 1–5 examples or an explicit output schema |
| Too many jobs in one prompt | Split into chained steps (draft → review → refine) |
| Preamble and hedging | Say "skip the preamble; answer directly" |
| Fabricated facts | Permit "I don't know"; require quoted source grounding |
| Contradictory instructions | Remove duplicates; keep one source of truth |
| Bloated structure on a simple task | Strip tags/sections; write plainly |

### Step 3 — Rewrite

Apply only the techniques the task needs:

- Lead with a clear objective and, where useful, a short persona.
- Put context and reference material first; put the actual query at the end.
- Delimit distinct content (`<context>`, `<examples>`, `<input>`) when the
  prompt mixes instructions, data, and examples.
- Encode "dos" positively; avoid long lists of prohibitions.
- State the exact output format; use a JSON schema when a machine reads it.
- Keep instructions consistent and non-redundant.
- For reasoning models: simplify, remove "think step by step," keep delimiters.
- End with a one-line recap of the key constraint and output format.

Keep the rewritten prompt self-contained and copy-paste ready.

### Step 4 — Verify

Before delivering, check the rewrite against the final checklist in
`references/prompt-engineering-guide.md`: intent, context, structure, examples,
constraints, format, reasoning, decomposition, robustness, model fit.

### Step 5 — Deliver

Respond in this order:

1. **Optimized prompt** — in a single fenced code block, ready to copy.
2. **What changed and why** — a short bullet list tied to the diagnosed issues.
3. **Assumptions / open questions** — anything the user should confirm.
4. **How to test it** — define success and give 2–3 representative test inputs,
   including one edge case.

Keep prose minimal; the optimized prompt is the product.

## Guardrails

- Never claim a rewrite is guaranteed to work; prompt outcomes are
  non-deterministic and depend on the model and data.
- Do not invent facts, policies, or source content to fill gaps — mark missing
  context as a placeholder such as `{{CONTEXT}}` for the user to supply.
- Do not add chain-of-thought requests for reasoning models.
- Do not over-structure a trivial prompt; note when the original was already
  good and only small edits were needed.
- When a task is genuinely too complex for one prompt, recommend a prompt chain
  and specify the handoff between steps rather than forcing everything into one.

## References

- `references/prompt-engineering-guide.md` — condensed knowledge base: how models
  work, prompt components, techniques, model types, pitfalls, final checklist.
- `references/optimization-playbook.md` — diagnostic rubric, clarifying-question
  bank, transformation patterns, worked before/after examples, and evaluation
  guidance.
