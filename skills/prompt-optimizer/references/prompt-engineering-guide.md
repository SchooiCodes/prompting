# Prompt Engineering — Condensed Knowledge Base

Derived from primary documentation by OpenAI, Anthropic, Google, and NVIDIA.
For the full research with URLs, see the repository's `research/` directory.

## How models work (what the prompt is actually doing)

- Input is tokenized into small units mapped to numbers; the model predicts the
  most likely next token and decodes the output. Wording, ordering, and
  formatting are part of the context the prediction is conditioned on.
- The **context window** caps how much input a model can consider at once
  (~100k to ~1M tokens). Place long reference material near the top; put the
  query last — this can improve quality by up to ~30% on complex inputs.
- Transformer **attention** lets any token in the context influence the output,
  which is why in-prompt instructions, roles, and examples steer generation.
- Output is **non-deterministic**: identical prompts can differ. Treat
  prompting as test-and-iterate, not one-shot perfection.
- Instruction priority: **system/developer > user > assistant**. Model
  families and snapshots respond differently; re-validate when switching.

## Model types

| | Reasoning models | General (GPT-style) models |
| --- | --- | --- |
| Strength | Complex multi-step reasoning, ambiguity | Fast, cheap, straightforward tasks |
| Prompt style | Short, direct, goal-oriented | Explicit, step-by-step instructions |
| Do NOT | Ask for "think step by step" or demand visible reasoning | Assume intent without stating it |
| Do | Use delimiters; state the goal clearly | Give examples and precise constraints |

Analogy: reasoning model = senior colleague given a goal; general model = junior
colleague who needs explicit steps.

## The four components of a prompt

| Component | Required | Role |
| --- | --- | --- |
| Task | Yes | The instruction or question |
| System instructions | No | Persistent role, tone, goals, constraints |
| Few-shot examples | No | Demonstrate pattern, format, tone |
| Contextual information | No | Data the model cannot know from training |

Quality = content (is everything relevant present?) + structure (are parts
labeled, ordered, delimited?).

## Techniques

1. **Clarity and specificity.** State exactly what is wanted. Apply the clarity
   test: would a colleague with no extra context know what to do? Include
   constraints (length, format, audience, requirements).
2. **Positive instructions.** Say what to do, not only what to avoid. Replace
   "Do not use markdown" with "Write in flowing prose paragraphs."
3. **Role/persona.** One focused sentence about expertise or tone sharpens output.
4. **Delimiters / XML tags.** Use `<context>`, `<instructions>`, `<examples>`,
   `<input>` when mixing distinct content types. For simple prompts, headings
   and whitespace suffice — do not over-structure.
5. **Few-shot examples.** Show the desired output. Use 3–5 relevant, diverse,
   structured examples; start with one and add only if needed. Ensure examples
   embody exactly the wanted behavior.
6. **Reasoning.** Ask for step-by-step thinking when it helps, with the final
   answer in `<answer>` tags. Prefer built-in/extended thinking. Never force CoT
   on reasoning models.
7. **Chaining.** For multi-step work, split into separate calls: draft → review →
   refine. Each step has one job; pass output forward.
8. **Output format.** Specify format explicitly; use Structured Outputs (JSON
   schema) when a machine consumes the result. Use prefilling to control format
   and "skip the preamble" to suppress hedging.
9. **Grounding.** Give relevant private context and ask the model to quote
   source passages first. Permit "I don't know" when uncertain.
10. **Recap.** Close with a short reminder of the critical constraint and format.
11. **Decoding controls.** Lower temperature for deterministic, factual tasks;
    higher for creative variety.

## Common pitfalls

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| Generic/off-target output | Vague request | Make the instruction explicit and specific |
| Inconsistent format | Format described, not shown | Add examples or prefill |
| Unreliable complex task | Too many jobs in one prompt | Split into chained steps |
| Preamble/hedging | No style instruction | Ask for the answer directly |
| Hallucinated facts | No uncertainty permission | Allow "I don't know"; require quotes |
| Suggestions instead of action | Ambiguous verb | Use "Change X," not "Can you suggest..." |
| Redundancy/drift | Repeated instructions | Remove duplicates; one source of truth |
| Works on A, fails on B | Model/snapshot difference | Re-tune per model; simplify for reasoning |

## Final checklist for an optimized prompt

1. Intent stated in one unambiguous sentence.
2. Role/persona added only if it focuses output.
3. Necessary context present, placed up top; query at the end.
4. Instructions, context, examples, input clearly delimited (if complexity warrants).
5. 1–5 relevant, diverse examples where format/tone matters.
6. Constraints explicit and phrased positively.
7. Output format specified (schema/JSON if machine-read).
8. Reasoning requested only where it helps; not forced on reasoning models.
9. Task decomposed into a chain if too complex for one prompt.
10. "I don't know" permitted; sources grounded.
11. Tuned for the target model family.
12. Success criteria and test inputs defined.
