---
name: performance-summary
description: Generate a performance summary comparing quantized model variants against a baseline, pulling evaluation results from GitHub PR descriptions.
user-invocable: true
---

# Performance Summary Skill

You are generating a performance summary that compares quantized LLM variants against an unquantized baseline. The summary is a markdown file written to `summaries/` in this repo.

## Step 1: Gather Inputs from the User

**You MUST ask the user for ALL of the following before proceeding.** Use AskUserQuestion or direct conversation — do not guess or assume any of these.

1. **PR links** — The GitHub PR URLs (or PR numbers) containing evaluation results. There must be at least two: one baseline and one or more quantized variants.
2. **Summary title** — A short descriptive title for the summary (e.g., "Llama-3.1 NVFP4 Quantization Ablations", "Llama-3.2 W8A8 vs. Baseline").
3. **Which PR is the baseline?** — Ask the user to confirm which PR contains the unquantized baseline results, or whether you should auto-detect it (the one without quantization keywords in the model name).
4. **Any additional notes?** — Ask if there are any important contextual notes to include in the summary (e.g., "activation quantization was skipped on H100", "results may be affected by X").

Do NOT proceed to Step 2 until you have all of this information.

## Step 2: Fetch PR Data

Fetch each PR's title and body using the `gh` CLI:

```bash
gh pr view <PR_NUMBER_OR_URL> --json title,body
```

Run all fetches in parallel since they are independent.

From each PR body, extract:
- **Model name** (from the `## Evaluation Results: <model>` header)
- **Session ID** (from `**Session ID:**`)
- **Server configuration** (Model Size GB, KV Cache Size GB, KV Cache Tokens, Recommended Concurrency, Max Model Length, vLLM Version)
- **Task results table** (Task, Metric, Mean, Std Dev, Repetitions) — use the **Mean** column as the accuracy score
- **Concurrency issues table** (Task, Issues) — if present

## Step 3: Identify Baseline and Variants

- The **baseline** is the unquantized model (largest Model Size, or as confirmed by the user in Step 1).
- All other models are **variants** to compare against the baseline.
- Derive a short configuration name for each variant from its model name. Strip the common base model prefix and use the remaining suffix (e.g., `Llama-3.1-8B-Instruct-NVFP4-GPTQ` → `NVFP4-GPTQ`).

## Step 4: Compute Summary Statistics

For each variant, compute the following against the baseline — **use Python to verify all calculations:**

### Per-task metrics:
- **Recovery %** = `(variant_score / baseline_score) * 100`
- **Absolute Δ** = `variant_score - baseline_score` (format with sign: `+0.0016` or `-0.0083`)

### Aggregate metrics:
- **Average accuracy** = mean of all task scores for a given model
- **Average recovery %** = `(variant_avg / baseline_avg) * 100`
- **Average Δ** = `variant_avg - baseline_avg`

### Python verification template:

```python
python3 -c "
baseline = {'task1': score, 'task2': score, ...}
variant1 = {'task1': score, 'task2': score, ...}
# ... more variants

for name, d in [('Variant1', variant1), ...]:
    avg = sum(d.values()) / len(d)
    bl_avg = sum(baseline.values()) / len(baseline)
    recovery = avg / bl_avg * 100
    delta = avg - bl_avg
    print(f'{name} avg: {avg:.4f}, recovery: {recovery:.2f}%, delta: {delta:+.4f}')
    for task in d:
        r = d[task] / baseline[task] * 100
        dd = d[task] - baseline[task]
        print(f'  {task}: recovery={r:.2f}%, delta={dd:+.4f}')
"
```

**Always run this verification.** Use the verified output when writing the summary — do not rely on mental arithmetic.

## Step 5: Write the Summary

Write the summary file to `summaries/<Title-Slug>_performance_summary.md` where `<Title-Slug>` is the kebab-or-underscore form of the summary title (e.g., `Llama-3.1-NVFP4-Ablations_performance_summary.md`).

### Required format and sections

Use the exact section structure below. Adapt table column counts and variant names to the actual data. For a single-variant comparison (1 baseline + 1 quantized), collapse the "Variant Ranking" section.

```markdown
# Performance Summary: <Title>

## Overview

This report compares <N> <quantization-method>-quantized variant(s) of <base-model> against the unquantized baseline across <M> evaluation benchmarks.

<If multiple variants, list them with a one-line description of what differentiates each.>

<Include any user-provided notes here as a bold **Important Note:** paragraph.>

## Model Configurations

| Configuration | Model Size | Size Reduction | Session ID |
|---------------|------------|----------------|------------|
| **Baseline (Unquantized)** | X.XX GB | — | <session> |
| **<Variant 1>** | X.XX GB | XX.X% | <session> |
| ... | ... | ... | ... |

Size Reduction = `(1 - variant_size / baseline_size) * 100`

## Accuracy Scores

| Task | Baseline | <Variant 1> | <Variant 2> | ... |
|------|----------|-------------|-------------|-----|
| **<task_name>** | X.XXXX | X.XXXX | X.XXXX | ... |
| ... | ... | ... | ... | ... |
| **Average** | **X.XXXX** | **X.XXXX** | **X.XXXX** | ... |

## Accuracy Recovery vs. Baseline

| Task | <Variant 1> | <Variant 2> | ... |
|------|-------------|-------------|-----|
| **<task_name>** | XX.XX% (-X.XXXX) | XX.XX% (+X.XXXX) | ... |
| ... | ... | ... | ... |
| **Average** | **XX.XX% (-X.XXXX)** | **XX.XX% (-X.XXXX)** | ... |

Format each cell as: `recovery% (signed_delta)`.
Use `+` prefix for positive deltas, `-` for negative.

## Variant Ranking (by Average Recovery)

Only include this section when there are 2+ quantized variants.

| Rank | Configuration | Avg Recovery | Avg Δ |
|------|---------------|--------------|-------|
| 1 | **<Best variant>** | XX.XX% | -X.XXXX |
| ... | ... | ... | ... |

## Resource Utilization

| Metric | Baseline | <Variant 1> | <Variant 2> | ... |
|--------|----------|-------------|-------------|-----|
| KV Cache Size (GB) | XX.XX | XX.XX | XX.XX | ... |
| KV Cache Tokens | XXX,XXX | XXX,XXX | XXX,XXX | ... |
| Recommended Concurrency | XX.XX | XX.XX | XX.XX | ... |

Add a one-line note below the table summarizing the resource impact (e.g., "All quantized variants free up GPU memory, enabling ~17% larger KV caches...").

## Concurrency Issues

Only include this section if any model reported concurrency issues.

| Configuration | Task | Issues |
|---------------|------|--------|
| <Config> | <task> | <description> |
| ... | ... | ... |

---

**Evaluation Date(s):** YYYY-MM-DD (or range if spanning multiple days)
**vLLM Version:** X.XX.X
**Max Model Length:** XX,XXX tokens
**Source PRs:** [#N](url), [#N](url), ...
```

### Formatting rules

- Use 4 decimal places for accuracy scores (e.g., `0.8627`).
- Use 2 decimal places for recovery percentages (e.g., `98.18%`).
- Use 4 decimal places for absolute deltas (e.g., `-0.0157`).
- Use 1 decimal place for size reduction percentage (e.g., `62.0%`).
- Bold task names in the first column, bold the Average row entirely.
- Use `\|` to escape pipe characters in task names like `math_500|0`.
- Use commas in large numbers (e.g., `541,328`).
- The footer uses `**Label:** Value` formatting, one per line.

## Step 6: Present for Review

After writing the file, present a short summary to the user covering:
- The output file path
- The ranking of variants (if multiple)
- Any notable findings (tasks where recovery exceeds 100%, tasks with largest drops, concurrency concerns)

Ask the user to review and confirm before considering the task complete.
