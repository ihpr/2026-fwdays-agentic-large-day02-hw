---
name: token-optimizer
description: Audits and optimizes Cursor rules, skills, and subagent prompts for minimal token usage while preserving effectiveness. Use after creating or editing rules (.mdc), skills (SKILL.md), or subagent/task prompts.
---

# Token Optimizer

Run this audit on any newly created or modified rule, skill, or subagent prompt.

## Process

1. **Read** the target file fully.
2. **Score** each section against the checklist below.
3. **Rewrite** the file with optimizations applied.
4. **Output** the judgement (see format below).

## Optimization Checklist

Apply every applicable reduction:

| # | Check | Action |
|---|-------|--------|
| 1 | Redundant explanation | Remove — the agent already knows common concepts |
| 2 | Verbose phrasing | Shorten — prefer imperative fragments over full sentences |
| 3 | Paragraphs where bullets suffice | Convert to bullets |
| 4 | Repeated instructions | Consolidate into one occurrence |
| 5 | Filler words (e.g. "please", "make sure to", "it is important that") | Delete |
| 6 | Obvious context (e.g. "AI is a language model") | Delete |
| 7 | Unnecessary examples | Keep one, remove duplicates |
| 8 | Long code blocks that could be pseudocode | Shorten |
| 9 | Nested references deeper than 1 level | Flatten |
| 10 | Inline explanations of well-known tools/APIs | Remove |

## Judgement Output Format

After optimizing, output exactly this block:

```
TOKEN OPTIMIZATION JUDGEMENT
─────────────────────────────
File: <path>
Before: ~<N> tokens
After:  ~<N> tokens
Saved:  ~<N> tokens (<X>%)
─────────────────────────────
Key changes:
- <change 1>
- <change 2>
- <change 3>
```

Keep "Key changes" to 3–5 bullet points max. Each bullet ≤ 12 words.

## Estimation

Estimate tokens using: `words × 1.3`. Count words with `wc -w` before and after.

## Constraints

- Never remove domain-specific knowledge the agent cannot infer.
- Never remove trigger descriptions from skill metadata.
- Never change the semantic meaning of instructions.
- Preserve all file/path references and script invocations.
