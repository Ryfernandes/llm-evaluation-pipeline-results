# Performance Summary: Llama-3.1 RTN FP8 Test

## Overview

This report compares 1 FP8-quantized variant of Llama-3.1-8B-Instruct against the unquantized baseline across 5 evaluation benchmarks.

- **FP8-DYNAMIC**: RTN FP8 quantization with dynamic activation scaling (`ryfernandes/Llama-3.1-8B-Instruct-FP8-DYNAMIC`)

**Important Note:** Evaluation was run on a single H100.

## Model Configurations

| Configuration | Model Size | Size Reduction | Session ID |
|---------------|------------|----------------|------------|
| **Baseline (Unquantized)** | 15.00 GB | — | 070720260942 |
| **FP8-DYNAMIC** | 8.55 GB | 43.0% | 070820261100 |

## Accuracy Scores

| Task | Baseline | FP8-DYNAMIC |
|------|----------|-------------|
| **gsm8k_platinum_cot_llama** | 0.8627 | 0.8550 |
| **ifeval** | 0.8094 | 0.8165 |
| **math_500\|0** | 0.4727 | 0.4600 |
| **mmlu_cot_llama** | 0.7270 | 0.7219 |
| **mmlu_pro_chat** | 0.4668 | 0.4649 |
| **Average** | **0.6677** | **0.6637** |

## Accuracy Recovery vs. Baseline

| Task | FP8-DYNAMIC |
|------|-------------|
| **gsm8k_platinum_cot_llama** | 99.11% (-0.0077) |
| **ifeval** | 100.88% (+0.0071) |
| **math_500\|0** | 97.31% (-0.0127) |
| **mmlu_cot_llama** | 99.30% (-0.0051) |
| **mmlu_pro_chat** | 99.59% (-0.0019) |
| **Average** | **99.39% (-0.0041)** |

## Resource Utilization

| Metric | Baseline | FP8-DYNAMIC |
|--------|----------|-------------|
| KV Cache Size (GB) | 56.52 | 62.94 |
| KV Cache Tokens | 462,960 | 515,600 |
| Recommended Concurrency | 14.47 | 16.11 |

FP8 quantization reduces model size by 43.0%, freeing up GPU memory and enabling an 11.4% larger KV cache (515,600 vs. 462,960 tokens) with 11.3% higher recommended concurrency.

## Concurrency Issues

| Configuration | Task | Issues |
|---------------|------|--------|
| Baseline (Unquantized) | mmlu_cot_llama | max KV cache: 88.6% |
| Baseline (Unquantized) | mmlu_pro_chat | preemptions: 4,377, max KV cache: 100.0% |
| FP8-DYNAMIC | mmlu_pro_chat | max KV cache: 97.7% |

---

**Evaluation Date(s):** 2026-07-07 – 2026-07-08
**vLLM Version:** 0.24.0
**Max Model Length:** 32,000 tokens
**Source PRs:** [#23](https://github.com/Ryfernandes/llm-evaluation-pipeline-results/pull/23), [#29](https://github.com/Ryfernandes/llm-evaluation-pipeline-results/pull/29)
