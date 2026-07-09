# LLM Evaluation Pipeline Results

Collection of evaluation results and performance summaries from the RHOAI model evaluation pipeline. Results are automatically added via pull requests.

## Repository Structure

```
results/          Collated JSON files from evaluation sessions
summaries/        Markdown performance summaries comparing model variants
.claude/skills    Claude skills for generating useful artifacts from evaluation data
```

### `results/`

Each file follows the naming convention `collated-{session_id}.json` and contains:

- **Server configuration** — vLLM version, model size, KV cache size, recommended concurrency, max model length
- **Evaluation results** per benchmark task, including:
  - Inference parameters (temperature, top_p, top_k, max tokens, seed, concurrency)
  - Aggregate statistics across multiple repetitions (mean/std of metrics and duration)
  - Per-run details: individual metrics, proxy statistics (token counts, latencies), vLLM metrics (throughput, preemptions, queue times), and log statistics

### `summaries/`

Markdown reports comparing quantized model variants against an unquantized baseline, generated with the [performance-summary](./claude/skills/performance-summary) Claude skill. Each summary includes accuracy recovery percentages, model size reduction, and resource utilization across benchmarks.