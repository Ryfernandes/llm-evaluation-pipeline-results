# Performance Summary: Llama-3.1 NVFP4 Quantization Ablations

## Overview

This report compares four NVFP4-quantized variants of Llama-3.1-8B-Instruct against the unquantized baseline across five evaluation benchmarks. The ablation study evaluates the impact of different weight calibration strategies on accuracy recovery:

- **NVFP4** — No weight calibration
- **NVFP4-GPTQ** — GPTQ weight calibration (default block size 128)
- **NVFP4-GPTQ-BlockSize16** — GPTQ weight calibration with block size 16
- **NVFP4-GPTQ-SmoothQuant** — GPTQ weight calibration + SmoothQuant

**Important Note:** All NVFP4 models ran on H100 machines, so activation quantization was skipped. This configuration improves accuracy recovery but results in slower inference and does not achieve full compression.

## Model Configurations

| Configuration | Model Size | Size Reduction | Session ID |
|---------------|------------|----------------|------------|
| **Baseline (Unquantized)** | 15.00 GB | — | 070720260942 |
| **NVFP4** | 5.70 GB | 62.0% | 070820260833 |
| **NVFP4-GPTQ** | 5.70 GB | 62.0% | 070720261540 |
| **NVFP4-GPTQ-BlockSize16** | 5.70 GB | 62.0% | 070720261539 |
| **NVFP4-GPTQ-SmoothQuant** | 5.70 GB | 62.0% | 070820260824 |

## Accuracy Scores

| Task | Baseline | NVFP4 | NVFP4-GPTQ | NVFP4-GPTQ-BS16 | NVFP4-GPTQ-SQ |
|------|----------|-------|------------|-----------------|---------------|
| **gsm8k_platinum_cot_llama** | 0.8627 | 0.8470 | 0.8544 | 0.8409 | 0.8456 |
| **ifeval** | 0.8094 | 0.8058 | 0.8110 | 0.8201 | 0.8066 |
| **math_500\|0** | 0.4727 | 0.4267 | 0.4427 | 0.4467 | 0.4520 |
| **mmlu_cot_llama** | 0.7270 | 0.7050 | 0.7136 | 0.7154 | 0.7136 |
| **mmlu_pro_chat** | 0.4668 | 0.4390 | 0.4481 | 0.4455 | 0.4466 |
| **Average** | **0.6677** | **0.6447** | **0.6540** | **0.6537** | **0.6529** |

## Accuracy Recovery vs. Baseline

| Task | NVFP4 | NVFP4-GPTQ | NVFP4-GPTQ-BS16 | NVFP4-GPTQ-SQ |
|------|-------|------------|-----------------|---------------|
| **gsm8k_platinum_cot_llama** | 98.18% (-0.0157) | 99.04% (-0.0083) | 97.47% (-0.0218) | 98.02% (-0.0171) |
| **ifeval** | 99.56% (-0.0036) | 100.20% (+0.0016) | 101.32% (+0.0107) | 99.65% (-0.0028) |
| **math_500\|0** | 90.27% (-0.0460) | 93.65% (-0.0300) | 94.50% (-0.0260) | 95.62% (-0.0207) |
| **mmlu_cot_llama** | 96.97% (-0.0220) | 98.16% (-0.0134) | 98.40% (-0.0116) | 98.16% (-0.0134) |
| **mmlu_pro_chat** | 94.04% (-0.0278) | 95.99% (-0.0187) | 95.44% (-0.0213) | 95.67% (-0.0202) |
| **Average** | **96.55% (-0.0230)** | **97.94% (-0.0138)** | **97.90% (-0.0140)** | **97.78% (-0.0148)** |

## Variant Ranking (by Average Recovery)

| Rank | Configuration | Avg Recovery | Avg Δ |
|------|---------------|--------------|-------|
| 1 | **NVFP4-GPTQ** | 97.94% | -0.0138 |
| 2 | **NVFP4-GPTQ-BlockSize16** | 97.90% | -0.0140 |
| 3 | **NVFP4-GPTQ-SmoothQuant** | 97.78% | -0.0148 |
| 4 | **NVFP4** | 96.55% | -0.0230 |

## Resource Utilization

| Metric | Baseline | NVFP4 | NVFP4-GPTQ | NVFP4-GPTQ-BS16 | NVFP4-GPTQ-SQ |
|--------|----------|-------|------------|-----------------|---------------|
| KV Cache Size (GB) | 56.52 | 66.08 | 66.08 | 65.82 | 66.08 |
| KV Cache Tokens | 462,960 | 541,328 | 541,328 | 539,200 | 541,328 |
| Recommended Concurrency | 14.47 | 16.92 | 16.92 | 16.85 | 16.92 |

All quantized variants free up GPU memory, enabling ~17% larger KV caches and higher concurrency than the baseline.

## Concurrency Issues

| Configuration | Task | Issues |
|---------------|------|--------|
| Baseline | mmlu_cot_llama | max KV cache: 88.6% |
| Baseline | mmlu_pro_chat | preemptions: 4,377, max KV cache: 100% |
| NVFP4 | mmlu_pro_chat | max KV cache: 93.6% |
| NVFP4-GPTQ | mmlu_cot_llama | max KV cache: 89.0% |
| NVFP4-GPTQ | mmlu_pro_chat | preemptions: 2,265, max KV cache: 100% |
| NVFP4-GPTQ-BlockSize16 | mmlu_pro_chat | preemptions: 2,296, max KV cache: 100% |
| NVFP4-GPTQ-SmoothQuant | mmlu_pro_chat | max KV cache: 91.0% |

---

**Evaluation Dates:** 2026-07-07 – 2026-07-08
**vLLM Version:** 0.24.0
**Max Model Length:** 32,000 tokens
**Source PRs:** [#23](https://github.com/Ryfernandes/llm-evaluation-pipeline-results/pull/23), [#25](https://github.com/Ryfernandes/llm-evaluation-pipeline-results/pull/25), [#26](https://github.com/Ryfernandes/llm-evaluation-pipeline-results/pull/26), [#27](https://github.com/Ryfernandes/llm-evaluation-pipeline-results/pull/27), [#28](https://github.com/Ryfernandes/llm-evaluation-pipeline-results/pull/28)
