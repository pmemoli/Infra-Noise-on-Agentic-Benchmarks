# Master Thesis Proposal: Quantifying and Mitigating Infrastructure and Sandbox Noise in Agent Evaluation

## Motivation

Agentic benchmarks run inside live infrastructure: containerized sandboxes, code executors, and tool back-ends whose transient failures (timeouts, rate limits, resource contention, nondeterministic returns) are orthogonal to model capability yet are folded into the reported score [1]. A recent systematic study shows that flaws in agentic task setup and reward design alone can distort measured performance by up to 100% in relative terms [1]. The problem reaches below the task layer: changing evaluation batch size, GPU count, or GPU version can shift generated responses, producing accuracy swings of up to 9% for a reasoning model from configuration changes alone [2]. This is the LLM-era analogue of flaky tests in software engineering, where outcomes alternate between pass and fail with no change to the artifact under test, mostly because of asynchronous waits, concurrency, and order dependencies in the environment [3]. These lines of work treat such noise as something to eliminate [2, 3]. This thesis instead treats it as a controllable variable and asks what it does to agents.

## Research Questions

1. Can a model recover from injected infrastructure disturbances, and at what cost (extra steps, tokens, latency, budget)?

2. Up to what noise intensity does recovery remain feasible before performance collapses, and how does that threshold scale with model size and scaffolding?

3. Where recovery fails, especially for small models, can a lightweight mitigation layer close the gap, and by how much?

## Approach

The project builds an evaluation harness with tunable disturbers that inject representative infrastructure faults at controlled rates on top of existing agentic tasks, holding the task distribution fixed so capability and robustness stay disentangled. It defines metrics beyond pass rate (recovery rate, cost-to-recover, and the noise threshold at which success becomes unattainable) to yield a per-model robustness profile rather than a single scalar, measured across a spectrum of model sizes. It then designs and evaluates a mitigation strategy aimed at small models (for example noise-aware scaffolding, retry and repair policies, or controller-level denoising) and quantifies the gain under matched budgets. 

The deliverable is a reusable benchmark plus an empirical account of when robustness can be bought cheaply and when it requires capability.

## References

- [1] Zhu et al. Establishing Best Practices for Building Rigorous Agentic Benchmarks. NeurIPS 2025 Datasets and Benchmarks Track.

- [2] Yuan et al. Understanding and Mitigating Numerical Sources of Nondeterminism in LLM Inference. NeurIPS 2025.

- [3] Luo, Hariri, Eloussi, Marinov. An Empirical Analysis of Flaky Tests. FSE 2014.

