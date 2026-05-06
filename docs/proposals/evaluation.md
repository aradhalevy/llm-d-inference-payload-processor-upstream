# IPP and Filter/Scorer/Picker — Evaluation Proposal

## Benchmarking Goals

We want to evaluate the IPP and the Filter / Scorer / Picker components performance, across two major aspects:

1. **Overheads** — verify that the overhead introduced by the IPP and by each component (Filter/Scorer/Picker) is minor, both end-to-end and per-component in isolation.
2. **Prediction quality** — optimize how close each scorer's predictions are to the actual latency / cost, and whether the Picker therefore picks the right model.

## Evaluation Approach

The evaluation will be driven by the llm-d-benchmark framework. It handles stack standup, workload execution, teardown, and analysis.

The evaluation will be built in two stages:

### Stage 1 — Overheads

Goal: verify that the overhead introduced is minor.

- **IPP baseline overhead** — request latency and TTFT with the IPP in the path (default plugins only) vs. with it bypassed and request routed directly to the appropriate InferencePool (requests will be sent with `X-Gateway-Base-Model-Name` header).
- **Per-component overhead** — latency and TTFT with the IPP running each component + other trivial components (e.g. latency Scorer with trivial Filter and Picker) vs. the IPP baseline above.

### Stage 2 — Prediction Quality

Goal: optimize prediction quality.

For each scored request, the scorer's per-candidate-pool predictions are recorded alongside the observed end-to-end latency / cost of the pool that was picked. From the joined stream we compute, per configuration, prediction error, MAE and MAPE between predicted and observed latency / cost, and iterate on the scorers to drive these down.

Ground truth for observed latency comes from two independent sources — the harness's own per-request timing (inference-perf / guidellm / vllm-benchmark) and the vLLM pod logs captured at run end. Using both lets us catch clock skew or attribution errors.

Measurement for cost is TBD.

## Configurations

We evaluate three stack configurations:

- **Configuration A** — One Inference Pool. Mainly as a baseline and for sanity checks.
- **Configuration B** — Two pools, two different models that are from the same category (e.g. "Frontier / Large"). Requests will be sent to both.
- **Configuration C** — Three pools, three different models, 2 from the same category (e.g. "Frontier / Large"), and one from another category (e.g. "Flash" / "Small"). Requests will be sent to the first group with the same category.

## Workload

Workloads come from the benchmark templates that ship with llm-d:

- **Harness:** guidellm and inference-perf.
- **Datasets:** synthetic datasets for each harness defined in llm-d-benchmark.
- **Concurrency:** 1×, 4×, 16×, 64× simultaneous requests, to see how overhead and prediction quality scale. Three different request-routing configurations:
  - All the concurrent requests are routed to the same model category.
  - First half of the concurrent requests are sent to a specific model from the category, second half are sent to the model category.
  - All the concurrent requests are sent to a specific model, except the last which is sent to the model category.

## Future Steps

Evaluate against external model providers.
