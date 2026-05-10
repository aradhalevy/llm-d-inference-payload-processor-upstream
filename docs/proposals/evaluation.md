# IPP and Filter/Scorer/Picker — Evaluation Proposal

This proposal defines end-to-end benchmark evaluation for the payload processor running plugins.
The initial proposal covers evaluating the following scenarios:

running multiple models on different pools - user specifies the model directly.
running multiple models on different pools - model/pool is picked by ModelSelector.

## Benchmarking Goals

We want to evaluate the IPP and the Filter / Scorer / Picker components performance, across two major aspects:

1. **Overheads** — verify that the overhead introduced by the IPP and by each component (Filter/Scorer/Picker) is minor, both end-to-end and per-component in isolation.
2. **Prediction quality** — Evaluate how close each scorer's predictions are to the actual latency, and whether the Picker therefore picks the right model. This could be later used to optimize the scorer.

## Evaluation Approach

The evaluation will be driven by the llm-d-benchmark framework. It handles stack standup, workload execution, teardown, and analysis.

As our project progresses, we would first focus on computing overhead statistics and in a later stage add prediction quality statistics as well.

### Metrics

Our collected metrics will be used to evaluate Overheads and Prediction quality, and understand the system behaviour. We aim to collect metrics about the system load and state (e.g. queue length), as well as metrics about the performance on specific requests (e.g TTFT).
We will start with an initial list of collected metrics that will expand as needed:

1. Concurrent requests - The number of concurrent requests the IPP handles at a given time.
2. queue length - The number of requests waiting to be served by the IPP at a given time.
3. GPU utilization - The allocated GPU divided by actual GPU used in a given time.
3. Total request latency - End to end latency of a specific request.
4. TTFT - Time to first token of a specific request.
5. TPOT - Time per output token of a specific request.

### Overheads evaluation

Goal: verify that the overhead introduced is minor.

- **IPP baseline overhead** — evaluate metrics with IPP in the path (default plugins only) vs. with it bypassed and request routed directly to the appropriate InferencePool.
- **Per-component overhead** —  evaluate metrics with the IPP running each component + other trivial components (e.g. latency Scorer with trivial Filter and Picker) vs. the IPP baseline above.

### Prediction Quality evaluation

Goal: verify prediction quality.

For each scored request, the scorer's per-candidate-pool predictions are recorded alongside the actual performance metrics of requests (e.g. TTFT). 
The actual metrics will act as a ground truth of requests.

From these collected metrics (ground truth and scorer's predictions) we compute per run the prediction error, MAE and MAPE between predicted and observed latency.
Later on, These results could be used to optimize the predictions quality.

## Configurations

We evaluate five stack configurations:

- **Configuration A**: one pool, one model (no IPP) - same number of model serving pods
- **Configuration B**: one pool, one model - same number of model serving pods
- **Configuration C**: two pools running the same model - same number of model serving pods
- **Configuration D**: two pools running the same model - 2/3 ratio of model serving pods between pools
- **Configuration E**: two pools running different models of same class (e.g. "Frontier / Large") - 2/3 ratio of model serving pods between pools

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
