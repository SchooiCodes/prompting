# Research Sources

Sources consulted for `prompt-engineering-research.md`. All were retrieved and
indexed on 2026-11-21. Pages are stored via context-mode for search; the notes in
the research document are synthesized from the retrieved text.

## Primary documentation

| Publisher | Title | URL | Notes |
| --- | --- | --- | --- |
| OpenAI | Prompt engineering | https://developers.openai.com/api/docs/guides/prompt-engineering | Definition, message roles/instruction hierarchy, few-shot, context, context windows, model families, production guidance |
| OpenAI | Reasoning best practices | https://developers.openai.com/api/docs/guides/reasoning-best-practices | Reasoning vs GPT models; keep prompts simple; avoid manual CoT; use delimiters |
| OpenAI | Model Spec | https://model-spec.openai.com/ | Chain of command: developer > user > assistant instruction priority |
| Anthropic | Prompt engineering overview | https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview | When to prompt engineer, success criteria, tutorials, links to living reference |
| Anthropic | Claude prompting best practices | https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices | Clear/direct, examples, XML tags, roles, thinking, chaining, research prompts, long-context ordering |
| Anthropic | Best practices for prompt engineering (2026) | https://claude.com/resources/articles/best-practices-for-prompt-engineering | Core techniques, technique-selection guide, troubleshooting table, XML caveats |
| Anthropic | Prompt generator (metaprompt) | https://colab.research.google.com/github/anthropics/claude-cookbooks/blob/main/misc/metaprompt.ipynb | Meta-prompting / draft generation via a model |
| Google | Introduction to prompt design | https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/prompts/introduction-prompt-design | Components of a prompt; few-shot; iterative design |
| Google | Prompt design strategies | https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/prompts/prompt-design-strategies | Content vs structure, sample template, output format, recap, pitfalls |
| NVIDIA | AI tokens explained | https://blogs.nvidia.com/blog/ai-tokens-explained | Tokens, tokenization, inference/training, context window, token economics |
| DSPy | Optimizers | https://dspy.ai/learn/optimization/optimizers/ | Automatic prompt/instruction optimization, bootstrapping, MIPROv2 |
| IBM | Transformer models | https://www.ibm.com/think/topics/transformer-models | Background on transformer/attention architecture |

## Retrieval notes

- Web search was unavailable during collection (HTTP 402); primary docs were
  fetched directly and indexed instead.
- The Google `prompt-design-intro` URL on the older `cloud.google.com/vertex-ai`
  path returned 404; the current Gemini Enterprise Agent Platform path was used.
- Wikipedia's Transformer article returned 403; IBM explainers were used for
  architecture background. The "how it works" section stays conceptual and
  cites only claims found in the retrieved documentation.
- DSPy's optimizer page is lightweight (redirects to the versioned docs); the
  framework's principles are summarized at a high level only.
