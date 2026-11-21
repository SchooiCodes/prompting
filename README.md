# Prompting Research & Prompt Optimizer Skill

A small repository that does two things:

1. **Documents how AI prompting works** and how to write effective prompts, based
   on primary documentation from OpenAI, Anthropic, Google, and NVIDIA.
2. **Provides an AI skill** (`prompt-optimizer`) that teaches a model to diagnose
   and rewrite a user's prompt into a high-performing one.

---

## Contents

```text
prompting/
├── README.md                          # This file
├── research/
│   ├── prompt-engineering-research.md # Consolidated findings from primary docs
│   └── sources.md                     # URL list and retrieval notes
└── skills/
    └── prompt-optimizer/
        ├── SKILL.md                   # Skill definition and workflow
        └── references/
            ├── prompt-engineering-guide.md  # Condensed knowledge base
            └── optimization-playbook.md     # Rubric, question bank, examples
```

---

## Research summary

The full write-up is in [`research/prompt-engineering-research.md`](research/prompt-engineering-research.md).
The headline points:

### How prompting works
- Models turn text into **tokens**, predict the next token, and decode the result.
  Every wording, ordering, and formatting choice changes that prediction.
- The **context window** limits input size (~100k to ~1M tokens). Put long
  reference material first and the query last.
- Output is **non-deterministic** — prompting is an iterative, test-driven craft.
- Authority order: **system/developer > user > assistant**.

### What makes a prompt good
- **Clear and specific** intent ("would a colleague understand it?").
- **Positive instructions** ("do X") instead of long lists of "don'ts".
- **Context** the model cannot otherwise know; **examples** when format or tone
  matters (3–5, relevant and diverse).
- **Structure** — delimiters like `<context>` and `<examples>` when mixing
  instructions, data, and examples; plain text is fine for simple tasks.
- **Explicit output format**, with a JSON schema when a machine consumes it.
- **Guardrails** — permit "I don't know" and require source grounding.
- **Decomposition** — chain complex tasks (draft → review → refine).

### Model awareness
- **Reasoning models** (e.g. o-series) want short, direct instructions and do
  *not* want "think step by step" — they reason internally.
- **General models** are fast and cheap but need explicit, detailed instructions.

### Improving prompts
Define success criteria, build a small test set, then loop:
`draft → evaluate → diagnose → rewrite → re-evaluate`. Meta-prompting and
frameworks such as DSPy automate parts of this search.

---

## The `prompt-optimizer` skill

A skill is a self-contained folder with a `SKILL.md` (YAML frontmatter + Markdown
instructions) and optional bundled `references/`, `scripts/`, and `assets/`.
This skill implements progressive disclosure: the frontmatter stays in context,
the body loads when the skill triggers, and the two reference files load only
when the model needs deeper detail.

### What it does

Given a draft prompt, the skill guides the model to:

1. Capture the goal, draft, target model, and desired output; ask 1–3 targeted
   questions only when the answer changes the rewrite.
2. Diagnose the prompt against a 12-dimension rubric.
3. Rewrite it using only the techniques the task needs (proportionality).
4. Verify it against a final checklist.
5. Deliver the **optimized prompt** first, then a short change list, assumptions,
   and a test plan with success criteria and 2–3 test inputs.

Guardrails keep it honest: preserve the user's intent, use `{{PLACEHOLDER}}` for
missing context instead of inventing facts, simplify for reasoning models, and
avoid over-structuring trivial prompts.

### Install

The canonical copy lives in `skills/prompt-optimizer/`. To make it available to
OpenCode, copy it into the skills directory:

```powershell
Copy-Item -Recurse -Force .\skills\prompt-optimizer `
  "$env:USERPROFILE\.config\opencode\skills\prompt-optimizer"
```

It has already been installed in this environment. On a fresh OpenCode session,
a skill appears in the available skills list once it is in that directory.

### Use

Ask for a rewrite in natural language, for example:

> Optimize this prompt: "Write something about our new product for social media."

> Here is a prompt that gives bad output. Improve it and explain why:
> "Read this contract, find risky clauses, and rewrite them."

The skill should trigger automatically on requests to optimize, improve, refine,
rewrite, debug, or strengthen a prompt.

### Validate / package

The skill-creator scripts validate and produce a distributable zip:

```powershell
$env:PYTHONIOENCODING = 'utf-8'
py -3 "$env:USERPROFILE\.config\opencode\skills\skill-creator\scripts\package_skill.py" `
  .\skills\prompt-optimizer .\
```

The skill passes validation as of the latest commit.

---

## Repository history

Work was committed incrementally:

| Commit | Change |
| --- | --- |
| `60903aa` | Initialize repository with `.gitignore` |
| `4ccb247` | Add consolidated research and sources |
| `8e04a5f` | Add `prompt-optimizer` skill and references |

---

## Sources

Primary documentation consulted (full list in [`research/sources.md`](research/sources.md)):

- [OpenAI — Prompt engineering](https://developers.openai.com/api/docs/guides/prompt-engineering)
- [OpenAI — Reasoning best practices](https://developers.openai.com/api/docs/guides/reasoning-best-practices)
- [Anthropic — Prompt engineering overview](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview)
- [Anthropic — Claude prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)
- [Anthropic — Best practices for prompt engineering](https://claude.com/resources/articles/best-practices-for-prompt-engineering)
- [Google — Introduction to prompt design](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/prompts/introduction-prompt-design)
- [Google — Prompt design strategies](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/prompts/prompt-design-strategies)
- [NVIDIA — AI tokens explained](https://blogs.nvidia.com/blog/ai-tokens-explained)
- [DSPy — Optimizers](https://dspy.ai/learn/optimization/optimizers/)

*Research collected 2026-11-21.*
