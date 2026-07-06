# Pre-Start Notes

This is my note-taking space for the runway before formally starting on July 29th on ETH Zurich.

## Goals

The ultimate goal of the thesis is exploring these 3 research questions:

1. Can a model recover from injected infrastructure disturbances, and at what cost (extra steps, tokens, latency, budget)?

2. Up to what noise intensity does recovery remain feasible before performance collapses, and how does that threshold scale with model size and scaffolding?

3. Where recovery fails, especially for small models, can a lightweight mitigation layer close the gap, and by how much?

And building an evaluation harness with tunable disturbers that inject representative infrastructure faults at controlled rates on top of existing agentic tasks. Maybe if time allows (most likely) also build some simple mitigation layer, or maybe even fine tuning the model to make it robust to tool failure.

Right now the priority is **reading the literature**, and becoming familiar with the **tech stack** over which I'm injecting noise. Curious mainly on emulating whatever claude code does for virtualizing its environment. I also want to get a better idea on the concrete infraestructure noise we'd inject  

## July 6th

Today I'm reading in depth:

- [1] Zhu et al. Establishing Best Practices for Building Rigorous Agentic Benchmarks. NeurIPS 2025 Datasets and Benchmarks Track.
- [2] https://www.anthropic.com/engineering/infrastructure-noise

### Best practices paper [1]

This paper proposes an Agentic Benchmark Checklist for rigorous evaluation and design of Agentic Benchmarks. As opposed with traditional multiple choice (e.g. MMLU) and language modeling benchmarks, agentic benchmarks can be quite complex to set up. These consist of the following:

**The task setup**

- Task description (just a markdown prompt)
- An environment (containerized)
- Tools (api calls, wolfram alpha, whatever)

**The outcome evaluation**

- evaluating the task E2E and finally comparing in the result in some way (LLM as a judge, string/substring matching, etc.).

They notice 2 recurrent issues on many benchmarks:

- **task validity**: The model is perfectly capable of solving the task but the benchmark fails (or viceversa), such as through rate limits or accessing ground truth labels. 

- **outcome validity**: It is not the case that the evaluation returns true IFF it is actually true. The outcomes can be evaluated as true or false when thats not the case, such as with faulty or incomplete unit testing.

And then propose a checklist to tackle these two issues (and another one relating to presenting the results) and evaluate the checklist over 17 benchmarks + experiment with applying changes based on it.

#### What are we getting from this

The authors present 17 agentic benchmarks chosen by popularity. It really makes sense to work with some of these at the start, maybe 2 or 3.

| Benchmark | Evaluated Capability | Evaluation Design |
|---|---|---|
| SWE-bench | Software Engineering | Unit Testing |
| SWE-Lancer | Software Engineering | End-to-end Testing |
| KernelBench | Software Engineering | Fuzz Testing |
| BIRD | Software Engineering | End-to-end Testing |
| Aider-Edit | Software Engineering | Unit Testing |
| Codeforces | Software Engineering | Unit Testing |
| LiveBench Coding | Software Engineering | Unit Testing |
| Aider-Polyglot | Software Engineering | Unit Testing |
| FrontierMath | Challenging Math Problem-solving | Answer Match |
| MLE-bench | ML Engineering | Quality Measure |
| RE-bench | ML Engineering | Quality Measure |
| τ-bench | Environment Interaction | Substring Matching, State Matching |
| WebArena | Environment Interaction | Whole String Matching, Substring Matching, LLM-as-a-Judge, State Matching |
| OSWorld | Environment Interaction | State Matching |
| WebVoyager | Environment Interaction | LLM-as-a-Judge |
| Cybench | Cybersecurity | Answer Matching |
| GAIA | General Assistant | Answer Matching |

It is also clear that our task relating to studying Agent robustness to disturbances relates to benchmark **task validity**, we don't really care about outcome. Some points of the ABC seem pretty relevant:

##### Checklist for tools

Tools come in two flavors:

- Self-Hosted tools like Python and CLI tools
- API-based tools

ABC for tools are:

1. Versions of self hosted tools are specified
2. Required API tools are consistently accessible (we can inject errors here through **rate limits** or just making it unavailable)
3. Evaluation terminates or handles errors appropiately (in our harness, we may not want to terminate it and rather see how it handles it)

I also notice that self hosted tools can also fail, through say resource contention.

##### Checklist for the environment

ABC for the env is:

1. Residual data can become stale between runs (we can inject this, not quite sure how though) 
    e.g. Relies on an sqlite or something that is not cleared between runs

2. Agent can't access ground truth information (maybe we can inject this and see how "honest" it is? feels too complex)

3. Setup is frozen and does not change over time (we can inject this, not quite sure how though)
    e.g. The benchmark checks for HTML selectors (like class names or page titles) on live web pages, or maybe it relies on a live database in constant change.

#### Possible failure points for task validity

We don't really want to eliminate these, but rather introduce them as a controlled variable. From the checklist I see 4 failure points:

1. Rate limits & inaccesibility on API tools (easy to implement and general)
2. Inaccesibility for CLI tools (resource contention, lack of access requirements)
3. Stale environment data
4. Drifting environment setup

1. and 2. seem recoverable, whereas 3. and 4. relate to task setup that doesn't seem recoverable...

### Anthropic Infra Noise paper [2]

This one is extremely relevant to what we are doing. They found that the runtime environment is very important for agentic benchmarks, and configurations of the environment can shift benchmark % accuracy by more than the difference between top models. 

They faced a lot of infra errors on their run of TerminalBench 2.0 related to exceeding the allocated resources for the agentic environmnet. They found that increasing the resource ceiling (originally ceiling = allocation) greatly **reduced the infra errors** (as expected) and **increased benchmark accuracy**. For 1x-3x ceiling, benchmark scores fluctate between margins of noise (p<0.4) but beyond that the scores increase and become significant (uncapped p < 0.01).  

More resources help agents solve problems they weren't able to before by exploring more aggressive resource-intensive solutions. So in a way the resource ceiling also affects *what is being measured*. Low resources measure efficent solution capabilities whereas high resource measure absolute benchmark capacity in an unrealistic scenario.  

They primarily measure the infra impact by increasing the resource ceiling of environments (which ends up reducing error rates from 6% to 0.5% when going x1 to x6 the original resources), but there are other sources of variance such as time/rate limits from APIs.

#### What are we getting from this

This is very interesting since it provides one very clear tunable parameter (the resource ceiling) which seems to account for a lot of the infraestructure errors.

From the previous paper, another very clear tunable parameter is API errors for the tools. I'm thinking of API rate and time limits, and simple unavailability for unkown reasons. 

Thus covering infra errors self hosted tools, and also API tools. 

What is not so clear to me is what metrics are we getting from this, pass rate is clearly one (for self hosted tools its not hitting the ceiling and for apis its not hitting the time limit) and effect on benchmark accuracy is another. But what about the whole recovery deal?... 

Tomorrow i'm reading the other paper from the proposal, looking at other relevant ones and summarizing my thoughts...
