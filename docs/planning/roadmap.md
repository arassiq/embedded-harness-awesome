# Roadmap and experiment design

Status: proposed milestones, not a delivery schedule. Resolve task, budget, and hardware choices before setting numeric success thresholds.

Agreed phase order: expand/review literature, then specify the high-level design with Aaron, then implement with DeepSeek. The [design direction](design-direction.md) and [handoff](handoff.md) prepare that discussion; they do not finalize the specification. No implementation is started by this documentation work.

## Research question

Can a background decision model select verified coding-agent experience that improves a small model more efficiently than random selection or simple rules?

The core product captures, curates, and exports datasets. A separate LoRA benchmark node tests downstream usefulness; it is not required to use the product. The first comparison isolates data selection. A later experiment may measure repeated learning through use.

## Product and learning groundwork

- [x] Choose individual coding-agent users adapting smaller models as the initial audience; treat demand and tolerance for rough edges as unvalidated hypotheses.
- [x] Require OpenCode compatibility without coupling the common data format to OpenCode.
- [ ] Work through the [data and capture guide](data-and-capture.md), tracing one small episode from raw events to a proposed training example.
- [ ] Check the installed OpenCode version and compare supported plugin hooks, event streams, and session-history access before choosing a capture mechanism.
- [ ] Validate that a prospective user can inspect and use an export independently of our training benchmark.
- [ ] Review the new trajectory-training, cheap-signal, context-reuse, and bounded-sampling references in the [literature review](literature-review.md).
- [ ] Agree the high-level specification and produce a concrete DeepSeek implementation handoff when the awesome-harness handoff support is ready.

## 0. Define the problem and evaluation

- [x] Choose tool use and function implementation as the starting task families.
- [ ] Choose concrete tasks, language, tools, and an OpenCode integration mechanism.
- [ ] Define tool-use success by the required environment outcome and function-implementation success by independent tests, each within a fixed attempt budget.
- [ ] Freeze evaluation conversation prefixes, repository/environment states, and available tools across model comparisons. Exclude later corrections and solutions from prefixes.
- [ ] Define success, acceptable regressions, verification evidence, and an affordable total budget.
- [ ] Choose the student checkpoint, training backend, and candidate selector after a hardware check.
- [ ] Create development tasks, a held-out test set, and a held-out retention set.
- [ ] Split by independent task, session, or repository as appropriate; group related attempts and near duplicates before splitting.
- [ ] Measure frozen-student performance to confirm there is a meaningful, learnable gap.

Exit: a written evaluation contract and a reproducible baseline. Synthetic or replayed traces may help bootstrap the pipeline, but must be reported separately from everyday usage evidence.

## 1. Capture usable experience

- [ ] Define a minimal trace format: task/session IDs, timestamps, context, attempts, tool results, corrections, outcome evidence, and source model/harness versions.
- [ ] Implement one integration and an inspectable local export.
- [ ] Exclude sensitive content and credentials; record collection boundaries.
- [ ] Convert a small set of verified resolutions into documented training exports, with rendering for a specific student kept at the export/benchmark boundary.
- [ ] Record provenance and verification status for every example.
- [ ] Specify byte limits, per-record limits, expiry, and overflow/eviction behavior for pending data, retained examples, evidence, audit samples, metrics, and export copies.
- [ ] Support both batch and live background processing with shared policies, bounded queues, and no indefinite raw-transcript archive.
- [ ] Check retained examples remain self-contained after temporary context expires.

Exit: inspectable examples with supported targets. Log volume alone is not success.

## 2. Compare selection strategies

- [ ] Implement random sampling from the same eligible pool.
- [ ] Implement a simple rule baseline, such as corrected attempts with verified final success.
- [ ] Compare cheap behavioral/execution signals inspired by Signals, without treating informativeness as correctness.
- [ ] Add Clef or another decision model behind an interchangeable selector interface.
- [ ] Define a student-independent selection rubric, abstention behavior, and a record of scores and decisions. Use episode context and outcome evidence rather than student-specific failure profiles.
- [ ] Audit selected and rejected examples against independent verification.
- [ ] Measure selection latency, tokens, hardware time, and any hosted charges.
- [ ] Compare batch/live processing on the same replay stream: peak memory, disk high-water mark, queue age/loss, throughput, agent slowdown, cost, and example availability.

Exit: reproducible selected datasets. Generic decision benchmarks do not establish suitability for our selection task.

## 3. Downstream benchmark node: test whether selection helps learning

- [ ] Train separate candidate LoRA adapters from the same base checkpoint using random, rule-selected, and model-selected examples.
- [ ] Match training-token budgets and training settings; report example counts and actual compute.
- [ ] Compare against the frozen student and retrieval of prior verified corrections.
- [ ] Use development data for thresholds and configuration; reserve the test set for reporting.
- [ ] Evaluate fresh tasks, older skills, serving cost, and complete experiment cost.
- [ ] Repeat the central comparison across seeds when affordable and report uncertainty.
- [ ] Save an adapter manifest with base revision, training data references, settings, selector version, and evaluation results.

Exit: evidence for or against the selection hypothesis. A negative result is a valid outcome.

## 4. Later benchmark extension: close the learning loop

- [ ] Add periodic bounded training attempts and candidate acceptance based on development evaluation.
- [ ] Score incoming tasks before their examples become eligible for learning.
- [ ] Track performance across several update cycles and recheck retention.
- [ ] Support loading the previous adapter after a rejected or regressing update.
- [ ] Compare equal total budgets, including selector overhead, rather than only equal training budgets.

Exit: a replayable history of experience, selection, adapter versions, costs, and measured outcomes.

## 5. Package the evidence

- [ ] Publish a reproducible experiment configuration and a shareable, reviewed dataset or fixture subset.
- [ ] Show a correction entering the pipeline and a related unseen task evaluated before and after training.
- [ ] Report accuracy versus total compute, selection ablations, retention, and failure cases.
- [ ] Write the project narrative around the actual results.

## Metrics

| Metric | What it answers |
| --- | --- |
| Held-out task success | Did the adapter improve related unseen work? |
| Incoming-task success before learning | Is the system getting better through time? |
| Retention change | What earlier capabilities regressed? |
| Verification rate in selected examples | Does selection retain supported targets? |
| Selection cost and throughput | Is background processing affordable at real traffic volume? |
| Peak RAM, disk usage, queue loss, and delay | Do live and batch options stay within resource bounds? |
| Full cost at a quality target | Does selection save enough training or serving work to repay its overhead? |
| Serving latency, tokens, retries, and escalations | Does the improvement translate to everyday user benefit? |

Report rolling metrics with window size and sample count. Changing task difficulty can move rolling scores without model improvement, so retain fixed evaluation sets and run frozen and adapted models on the same tasks. Keep test tasks and their answers out of training and selector tuning.

## Deferred work

Cloud-hosted asynchronous inference and optional cloud storage are v2/v3 directions. Bedrock is a candidate pending model-serving compatibility and cost checks, not a selected deployment. Cloud queues and retained data still need explicit quotas.

A possible v2 export would train tool-call accept/deny classifiers from pre-call user intent, proposed actions, relevant context, and policy-grounded labels. Compare a user-only context view against a richer view using context retained within the existing budget; do not keep raw history indefinitely for this purpose. This is separate from episode selection and not a commitment to live enforcement. See the [v2 design notes](data-and-capture.md#possible-v2-export-tool-call-acceptdeny-classification).

A local dashboard beyond the CLI is a direction to explore, not a committed milestone. It could show generated examples, provenance, and source proportions using explicit units and denominators. Keep it on top of the same local data rather than making it a dependency of capture or export.

A learned update scheduler, online training of the selector, per-skill adapter routing, adapter merging, full base-model fine-tuning, and reproduction of expensive RL outer loops. Revisit these only after the simpler selection experiment provides evidence.
