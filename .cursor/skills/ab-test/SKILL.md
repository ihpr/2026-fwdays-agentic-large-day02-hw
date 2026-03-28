---
name: ab-test
description: Runs A/B tests on Cursor rules, skills, or subagent prompts by comparing prompt execution with the target enabled (A) vs disabled (B). Disables targets by adding .off extension, restores after test. Use when the user wants to A/B test a rule, skill, or subagent, or compare prompt behavior with and without a specific configuration.
---

# A/B Test

## Inputs (ask if missing)

- **Target** — rule (`.mdc`), skill (`SKILL.md`), or subagent prompt path
- **Prompt** — prompt to run in both A and B
- **Context** — optional file(s) to include

## Workflow

### 1. Validate

- Confirm target exists and has content
- If `.off` version exists → warn and stop

### 2. Run A (Enabled)

- Announce: `Running test A (target ENABLED)...`
- Execute prompt via Task (`generalPurpose`, `readonly: true`)
- Capture output as `result_A`

### 3. Disable

- Rename: append `.off` (e.g. `security.mdc` → `security.mdc.off`)
- Verify rename succeeded

### 4. Run B (Disabled)

- Announce: `Running test B (target DISABLED)...`
- Execute same prompt via Task (identical params)
- Capture output as `result_B`

### 5. Restore (CRITICAL — always execute, even on error)

- Remove `.off` extension
- If restore fails → alert user with `.off` path

### 6. Output

Display in chat — **never save to file.**

## Output Format

```
A/B TEST RESULTS
─────────────────────────────
Target: <path>  Type: <Rule|Skill|Subagent>
Prompt: "<prompt>"
─────────────────────────────

── A (ENABLED) ──
<result_A>

── B (DISABLED) ──
<result_B>

── Comparison ──
| Aspect           | A (Enabled) | B (Disabled) |
|------------------|-------------|--------------|
| Followed target? | Yes / No    | N/A          |
| Response length  | ~N words    | ~N words     |
| Key differences  | <summary>   | <summary>    |
| Quality          | <note>      | <note>       |

── Verdict ──
<1–3 sentences: does the target meaningfully affect output?>
```

## Safety

- Always restore target, even on error — only rename, never modify content
- Never save results to disk
- Subagents run `readonly`

## Example

"A/B test security rule against reviewing `src/auth/login.ts`"
→ Target: `.cursor/rules/security.mdc` → Run A → Disable `.off` → Run B → Restore → Output comparison
