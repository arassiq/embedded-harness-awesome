# Literature review

Initial source check: 2026-11-12. This is a starting review, not an exhaustive survey. External findings below are distinct from project hypotheses; we have not reproduced these results.

Expanded: 2026-11-16. This pass focuses on trajectory curation, cheap selection baselines, downstream reuse, bounded sampling, and the two starting evaluation families. Sources were opened directly; review depth is stated per entry. This is sufficient to inform the next design discussion, not a claim that the literature review is complete.

## Dates and citation-count snapshot

Counts below are Semantic Scholar `citationCount` values retrieved on **2026-11-16**, not Google Scholar counts. Links in the count column identify the exact indexed record. Counts change, databases have different coverage, and preprint/conference versions may be split. They are context, not evidence that a method works. Dates are first arXiv release dates where applicable; conference dates are separate. The public metadata responses are preserved in [the snapshot](research/citation-snapshot-2026-11-16.json).

| Paper | First release / publication | Citations on 2026-11-16 |
| --- | --- | ---: |
| [LoRA](https://arxiv.org/abs/2106.09685) | 2021-06-17 | [25,943](https://www.semanticscholar.org/paper/a8ca46b171467ceb2d7652fbfb67fe701ad86092) |
| [LESS](https://arxiv.org/abs/2402.04333) | 2024-02-06 | [706](https://www.semanticscholar.org/paper/95898b1f82cf7ad7d96fcc85b4def7f086325af5) |
| [Chasing Random](https://arxiv.org/abs/2410.15225) | 2024-10-19 | [17](https://www.semanticscholar.org/paper/ca0c191b51a3e559ccbe44e83cf368c371f1847f) |
| [SEAL](https://arxiv.org/abs/2506.10943) | 2025-06-12 | [107](https://www.semanticscholar.org/paper/b61a64b24ddaf5fb83f8c0cd0c48dd9c4e96d47f) |
| [SDFT](https://arxiv.org/abs/2601.19897) | 2026-01-27 | [276](https://www.semanticscholar.org/paper/370a293b5dcf9d456ba7f805b5b0a3d8e984ff6e) |
| [Signals](https://arxiv.org/abs/2604.00356) | 2026-04-01 | [3](https://www.semanticscholar.org/paper/3feee811bbd0f6cd177d6dffc6109d9592e245ce) |
| [SWE-Gym](https://arxiv.org/abs/2412.21139) | 2024-12-30 | [445](https://www.semanticscholar.org/paper/d3f0dc8ec576c9a1dcc31375bbbbfe65135609a1) |
| [SWE-smith](https://arxiv.org/abs/2504.21798) | 2025-04-30 | [388](https://www.semanticscholar.org/paper/cbdfe7c75676b6ba85f66bdffe162f5991d6f536) |
| [Self-generated in-context examples](https://arxiv.org/abs/2505.00234) | 2025-05-01 | [27](https://www.semanticscholar.org/paper/1125a01656e1a022c82b15b539129e501ea8d007) |
| [HumanEval / code evaluation](https://arxiv.org/abs/2107.03374) | 2021-07-07 | [12,851](https://www.semanticscholar.org/paper/acbdbf49f9bc3f151b93d9ca9a06009f4f6eb269) |
| [Reservoir sampling](https://doi.org/10.1145/3147.3165) | 1985-03-01 | [2,112](https://www.semanticscholar.org/paper/da01f7fcc5c7eeba75bc09a41fdd946e65210090) |
| [BFCL](https://proceedings.mlr.press/v267/patil25a.html) | Conference: 2025-07-13–19 | Unavailable |

Clef is a model release/documentation source, not a paper in this table. Its sources were reviewed on 2026-11-12; no paper citation count is assigned. BFCL's conference listing dates it to 13–19 July 2025; this is a conference interval, not a verified first-preprint date. Its citation count could not be retrieved: Semantic Scholar search returned HTTP 429 and URL resolution failed. Unavailable does not mean zero.

## LoRA: Low-Rank Adaptation of Large Language Models

- Source: [Hu et al., 2021](https://arxiv.org/abs/2106.09685).
- Review depth: abstract reviewed.
- Finding: freezes pretrained weights and trains added low-rank matrices, reducing the trainable parameter count for adaptation.
- Project implication: a suitable downstream benchmark node for testing exported data without storing a full fine-tuned model for every candidate. Adapter training is not the core product output.
- Limitation: reduced trainable parameters do not establish our end-to-end compute savings or protection against interference.
- Follow-up: review target-module and rank choices, then measure memory and throughput on our actual hardware.

## LESS: Selecting Influential Data for Targeted Instruction Tuning

- Source: [Xia et al., 2024](https://arxiv.org/abs/2402.04333).
- Review depth: abstract reviewed.
- Finding: selects instruction examples using optimizer-aware gradient similarity to examples of a target capability. The paper reports transfer of selection across model sizes and families.
- Project implication: useful data depends on the capability we want to improve. Generic response quality is not enough, and selecting data with one model for another is established prior work.
- Limitation: this does not establish that a prompted decision model can cheaply select useful live agent trajectories.
- Follow-up: inspect experimental controls and decide whether a gradient-based selector is an affordable later baseline.

## Chasing Random: Instruction Selection Strategies Fail to Generalize

- Sources: [Diddee and Ippolito, NAACL Findings 2025](https://aclanthology.org/2025.findings-naacl.103/), [arXiv abstract](https://arxiv.org/abs/2410.15225), [available full text](https://arxiv.org/html/2410.15225v1).
- Review depth: abstract and full-text experimental outline reviewed; detailed tables and cost assumptions still to audit.
- Finding: selection strategies do not consistently beat random sampling across the tested datasets, budgets, and evaluations. Selection cost can exceed full-data fine-tuning cost in their settings.
- Limitation: general instruction tuning is not our live coding-agent collection setting; this does not establish that selection never helps.
- Project implication: include random selection and full eligible-pool training when affordable; account for selection overhead and variability.
- Follow-up: audit cost assumptions and compare equal training tokens separately from equal end-to-end budgets.

## Cloudflare Clef and Clef-Flash

- Sources: [Cloudflare announcement](https://blog.cloudflare.com/clef-decision-models/), [Clef model card](https://huggingface.co/Cloudflare/clef), [Clef-Flash model card](https://huggingface.co/Cloudflare/clef-flash).
- Review depth: announcement and relevant model-card architecture/usage sections reviewed.
- Finding: typed decision models score allowed choices rather than generating free-form answers. Clef is 27B parameters; Clef-Flash is 9B. Clef-Flash returns option probabilities in a single forward pass.
- Project implication: candidate for structured selection decisions with an explicit rubric and abstention option.
- Limitation: efficient decision output does not make a 9B backbone negligible, and published decision benchmarks do not validate training-example utility. Probability calibration for our workload remains untested.
- Follow-up: benchmark actual selection quality and cost against rules and a smaller classifier. Check backend support rather than assuming ordinary text-generation loading is sufficient.

## Self-Adapting Language Models (SEAL)

- Source: [Zweiger et al., 2025](https://arxiv.org/abs/2506.10943); [reviewed full-text version](https://arxiv.org/html/2506.10943v2).
- Review depth: abstract, method overview, and limitations reviewed.
- Finding: a model generates fine-tuning data and update directives; an RL outer loop rewards edits using downstream performance after adaptation.
- Limitation: the authors identify substantial training-and-evaluation overhead, forgetting across edits, and dependence on paired downstream evaluation tasks.
- Project implication: connects generated learning material to measured adaptation outcomes. Our first version uses fixed selection and training procedures rather than reproducing its learned self-edit policy.
- Follow-up: review its evaluation separation and sequential-edit experiments for lessons about repeated updates.

## Self-Distillation Enables Continual Learning (SDFT)

- Source: [Shenfeld et al., 2026](https://arxiv.org/abs/2601.19897); [reviewed full-text version](https://arxiv.org/html/2601.19897v2).
- Review depth: abstract, method overview, model-size experiment, and compute limitations reviewed.
- Finding: a demonstration-conditioned version of the model provides teacher signals for on-policy student learning. The paper reports improved retention relative to SFT in its experiments.
- Limitation: reports about 2.5 times the FLOPs and 4 times the wall-clock training time of SFT. In its Science Q&A scaling experiment, the 3B variant trails SFT; benefits are stronger at larger scales.
- Project implication: self-distillation is a possible later comparison, not evidence that a small student can learn effectively without external demonstrations or meaningful compute.
- Follow-up: evaluate whether our student can use demonstrations successfully before considering this training method.

## Signals: Trajectory Sampling and Triage for Agentic Interactions

- Sources: [Chen, Hafeez, and Paracha, 2026](https://arxiv.org/abs/2604.00356), [full text v1](https://arxiv.org/html/2604.00356v1).
- Review depth: abstract, taxonomy overview, and limitations reviewed; detailed annotation protocol still to audit.
- Finding: cheap behavioral/execution signals triage traces without model calls. On a tau-bench annotation study, the authors report 82% informative traces versus 74% for heuristic filtering and 54% for random sampling.
- Limitation: the outcome is developer-rated informativeness, not correctness or downstream training improvement. Airline/retail tasks use simulated users; real coding sessions are untested. Behavioral signals can miss fluent but incorrect answers.
- Project implication: a stronger cheap baseline than random alone; evaluate whether a decision model adds value beyond signals. Possible cascade: inexpensive signals followed by selective model assessment.
- Follow-up: reproduce only relevant detectors and audit annotation definitions before adopting the comparison.

## Training Software Engineering Agents and Verifiers with SWE-Gym

- Sources: [Pan et al., first released 2024; ICML 2025](https://arxiv.org/abs/2412.21139), [full text v2](https://arxiv.org/html/2412.21139v2).
- Review depth: abstract, agent-training setup, and relevant verifier/training details reviewed.
- Finding: provides executable Python tasks and trains agents on successful trajectories using filtered behavior cloning. It separately trains verifiers from labeled trajectories. The paper reports better resolve rates after training.
- Limitation: these are curated executable environments, not arbitrary personal chats. Its general-agent runs use full-parameter training on substantial GPU hardware; those results do not demonstrate our proposed low-cost LoRA setup.
- Project implication: success-filtered trajectories are an important baseline. Preserve task state, action/result linkage, and the difference between policy-training and verifier-training examples.
- Follow-up: inspect small fixtures and permitted dataset reuse before any download; avoid importing the entire environment collection by default.

## SWE-smith: Scaling Data for Software Engineering Agents

- Source: [Yang et al., 2025](https://arxiv.org/abs/2504.21798).
- Review depth: abstract reviewed.
- Finding: constructs executable task instances by modifying codebases to break tests, producing large-scale software-agent training data and reporting downstream gains.
- Limitation: synthetic task generation differs from observing everyday user sessions, and benchmark results do not validate our collector, selector, or storage budget.
- Project implication: executable evidence and environment management matter. This is adjacent prior work, not evidence that passive conversation capture is novel or that all collected traces are useful.
- Follow-up: compare task/trajectory schemas and storage requirements without committing to its full pipeline.

## Self-Generated In-Context Examples Improve LLM Agents for Sequential Decision-Making Tasks

- Source: [Sarukkai, Xie, and Fatahalian, 2025](https://arxiv.org/abs/2505.00234).
- Review depth: abstract reviewed; detailed method and evaluation controls remain to read.
- Finding: stores successful agent trajectories as in-context examples and reports gains across ALFWorld, Wordcraft, and InterCode-SQL; it also studies curation of the example database.
- Limitation: this is context-based reuse, not adapter training, and its domains do not establish benefits for coding conversations with real humans.
- Project implication: the exported dataset may be useful without weight updates. Retain a retrieval/example-reuse baseline when benchmarking LoRA.
- Follow-up: inspect selection cost and train/test separation before adapting its method.

## Random Sampling with a Reservoir

- Sources: [Vitter, 1985, ACM TOMS](https://doi.org/10.1145/3147.3165), [paper copy](https://www.cs.emory.edu/~cheung/Courses/584/Syllabus/papers/RandomSampling/1985-Vitter-Random-sampling-with-reservior.pdf).
- Review depth: paper abstract and opening problem definition reviewed.
- Finding: one-pass sampling of a fixed number of records from a stream of unknown length.
- Limitation: a fixed record count does not bound bytes when records vary in size; random sampling does not establish example quality. Deleting expired records changes the sampling population and needs an explicit policy.
- Project implication: a bounded random-sampling baseline is possible without archiving the whole stream. Apply a separate per-record size limit and byte quota.
- Follow-up: define eligibility, sampling unit, time window, and size policy before choosing an algorithm.

## The Berkeley Function Calling Leaderboard (BFCL)

- Source: [Patil et al., ICML 2025](https://proceedings.mlr.press/v267/patil25a.html).
- Review depth: published abstract reviewed; detailed grading and dataset versions remain to inspect.
- Finding: evaluates function calls, including serial/parallel calls, abstention, and stateful multi-step settings, using structured evaluation methods.
- Limitation: valid function calling is different from authorization, and neither alone proves a complete coding task was solved. Pin a benchmark version rather than using a changing leaderboard as the specification.
- Project implication: reference for tool schemas and evaluation cases, not an automatic replacement for environment-outcome checks.
- Follow-up: choose a small relevant subset or independent fixture design after inspecting licensing and graders.

## Evaluating Large Language Models Trained on Code (HumanEval)

- Source: [Chen et al., 2021](https://arxiv.org/abs/2107.03374).
- Review depth: abstract reviewed.
- Finding: introduces HumanEval for functional correctness of generated programs and studies repeated sampling.
- Limitation: function synthesis from docstrings differs from multi-turn repair; passing available tests does not prove correctness beyond their coverage. A public benchmark also needs a contamination assessment.
- Project implication: reference for function-implementation evals. Hold sample/attempt budgets fixed and evaluate independent tasks rather than training on benchmark solutions.
- Follow-up: inspect tests and pass@k definitions before adopting a benchmark protocol.

## Current synthesis

Data selection, adapters, and learning from demonstrations already have substantial prior work. The project hypothesis is that outcome-aware selection from ongoing coding-agent experience can improve future student performance within a realistic total budget. Neither Clef's availability nor LoRA's efficiency establishes that hypothesis.

The expanded review adds three practical challenges: outperform cheap trace signals as well as random selection; distinguish informative traces from verified training targets; and compare adapter training with context-based reuse. Bounded storage is a product constraint, not a research result. No reviewed source establishes that either live inference or batch inference is best for our workload; measure both.

## Reading queue

- [ ] Audit Chasing Random's detailed controls and cost assumptions.
- [ ] Study LESS's evaluation protocol and cost of building its gradient datastore.
- [x] Find primary work on coding-agent trajectory training and cheaper trace triage.
- [ ] Audit the trajectory-training and Signals annotation protocols beyond this initial pass.
- [x] Find a primary reference for context-based reuse of successful trajectories.
- [ ] Compare context reuse versus fine-tuning specifically for repeated developer workflows.
- [ ] Review contamination, temporal evaluation, and continual-learning retention metrics.

## Template for new sources

- Citation and version:
- First release date and venue date (if different):
- Citation count, index/record, and lookup date (or explicit unavailable status):
- Review depth:
- Research question and method:
- Evidence and experimental setting:
- Limitations:
- Implication for this project:
- Follow-up question or reproduction:
