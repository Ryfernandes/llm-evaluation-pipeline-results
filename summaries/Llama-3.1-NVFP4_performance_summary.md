# Performance Summary: NVFP4 Quantization vs. Unquantized Baseline

## Overview

This report compares the performance of the NVFP4-quantized Llama-3.1-8B-Instruct model against its unquantized baseline across five evaluation benchmarks.

**Important Note:** The NVFP4 model ran on H100 machines, so activation quantization was skipped. This configuration improves accuracy recovery but results in slower inference and does not achieve full compression.

## Model Comparison

| Configuration | Model Size | Size Reduction | Session ID |
|---------------|------------|----------------|------------|
| **Baseline (Unquantized)** | 15.00 GB | - | 070720260942 |
| **NVFP4 (Quantized)** | 5.70 GB | 62.0% | 070720260946 |

## Accuracy Recovery Results

| Task | Baseline Accuracy | NVFP4 Accuracy | Recovery % | Absolute Δ |
|------|-------------------|----------------|------------|------------|
| **gsm8k_platinum_cot_llama** | 0.8627 | 0.8506 | **98.60%** | -0.0121 |
| **ifeval** | 0.8094 | 0.8185 | **101.12%** | +0.0091 |
| **math_500\|0** | 0.4727 | 0.4467 | **94.50%** | -0.0260 |
| **mmlu_cot_llama** | 0.7270 | 0.7011 | **96.44%** | -0.0259 |
| **mmlu_pro_chat** | 0.4668 | 0.4383 | **93.89%** | -0.0285 |
| **Average** | **0.6677** | **0.6510** | **96.91%** | **-0.0167** |

## Resource Utilization

### Baseline Model
- KV Cache Size: 56.52 GB
- KV Cache Tokens: 462,960
- Recommended Concurrency: 14.47

### NVFP4 Model
- KV Cache Size: 66.08 GB (+17%)
- KV Cache Tokens: 541,328 (+17%)
- Recommended Concurrency: 16.92 (+17%)

**Note:** Both models experienced KV cache pressure on `mmlu_pro_chat` (100% utilization) with significant preemptions (4377 for baseline, 1606 for NVFP4).

---

**Evaluation Date:** 2026-07-07  
**vLLM Version:** 0.24.0  
**Max Model Length:** 32,000 tokens
