# IPP and Filter/Scorer/Picker — Evaluation Proposal

This proposal defines end-to-end benchmark evaluation for the payload processor running plugins.
The initial proposal covers evaluating the following scenarios:

running multiple models on different pools - user specifies the model directly.
running multiple models on different pools - model/pool is picked by ModelSelector.

## Benchmarking Goals

IPP needs to be evaluated across two major aspects:

1. **Overheads** — understand the overhead (if any) introduced by the IPP and by each component (Filter/Scorer/Picker). Useful for evaluating functionality introduced in IPP.
2. **Inference performance** — Evaluate the end-to-end system behavior for various scenarios. E.g., evaluate model picking, etc.

## Evaluation Approach

The evaluation will be driven by the llm-d-benchmark framework. It handles stack standup, workload execution, teardown, and analysis.
In the proposal we define the metrics and initla set of scenarios.

### Metrics

The collected metrics will be used to evaluate Overheads and inference performance, and include metrics about the system load and state (e.g. queue length), as well as metrics about the performance on specific requests (e.g TTFT).
We will start with an initial list of collected metrics that will expand as needed:

1. Concurrent requests - The number of concurrent requests the IPP handles at a given time.
2. Queue length - The number of requests waiting to be served by the IPP at a given time.
3. GPU utilization - The allocated GPU divided by actual GPU used in a given time.
3. Total request latency - End to end latency of a specific request.
4. TTFT - Time to first token of a specific request.
5. TPOT - Time per output token of a specific request.
6. Token (input, output, cached) count.

### Baseline evaluation

- evaluate metrics with IPP in the path (minimal plugins only) vs. with it bypassed and request routed directly to the appropriate InferencePool.

### Latency evaluation

IPP configured with a filter/scorer/picker pipeline for the following scenarios:
- One pool, one model - same number of model serving pods
- Two pools running the same model - same number of model serving pods
- Two pools running the same model - 2/3 ratio of model serving pods between pools
- two pools running different models of same class (e.g. "Frontier / Large") - 2/3 ratio of model serving pods between pools

## Workload

Workloads come from the benchmark templates that ship with llm-d:

- **Harness:**  [inference-perf](https://github.com/kubernetes-sigs/inference-perf), a benchmarking tool that focuses on inference performance.  
- **Datasets:** Initial evaluation will start with these datasets that work out of the box with inference-perf:
  1. ShareGPT - Emulates real chatbot conversations. Short, varied inputs, low prefix overlap across distinct conversations.
  2. SharedPrefix - Multi-tenant serving where N groups share a system prompt, each group has M users asking questions. A prefix-cache heavy benchmark.

- **Concurrency:**  concurrency will be increased gradually, to see how overhead and prediction quality scale. 

## Future Steps

Evaluate against More Use cases (more datasets that work with inference-perf, possibly different Harnesses).

Evaluate the cost Scorer, adding relevant metrics (e.g price list of million tokens per model).
