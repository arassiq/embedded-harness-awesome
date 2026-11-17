# First principles and product thinking

Initial framing: 2026-11-12. These notes capture the current discussion, not validated user research.

## Why we chose this project

Everyday agent work produces attempts, corrections, tool results, and occasional verified successes. Our hypothesis is that some of this experience could help a smaller model handle related future work, but capturing every interaction does not automatically produce useful training data.

We want to understand how much learning we can get from a limited compute budget. A decision model could help identify experience worth learning from, while a harness provides the outcomes and evaluation needed to check whether that judgment was useful.

The personal motivation is to build a credible MLE portfolio project: a usable system with a clear demonstration, reproducible experiments, and defensible tradeoffs. Recruiter appeal is a motivation, not an experimental success metric. We should communicate measured results without claiming a breakthrough or novelty we have not established.

## Starting from the user's problem

Initial target audience: coding-agent users who want to capture their own experience and fine-tune smaller models for their workflows.

Starting use case: capture the best supported examples from everyday agent use into a high-quality dataset for fine-tuning smaller models to perform tasks more efficiently. The product output is the dataset, its provenance, and evidence of quality. Improved model efficiency is a downstream hypothesis to benchmark.

Two candidate audiences discussed are specialized companies building custom tools and individual agent users adapting smaller models to their workflows. We choose the second initially. Aaron's product hypothesis is that this gives us a focused early audience willing to tolerate an unpolished tool in exchange for useful data. The size of either audience, demand for fine-tuning, and tolerance for rough edges have not been validated. We should not assume all coding-agent users want to train models.

Proposed job: turn useful episodes from ordinary work into inspectable, reusable training data without manually curating an entire dataset. Describe a dataset as golden only when it meets a stated evidence standard; model-selected candidates are not automatically verified.

Example scenario to validate: an assistant chooses the wrong search tool or repository location, receives a correction, and completes the task successfully. Can learning from several such episodes improve tool use on new tasks?

As of 2026-11-13, tool use and function implementation are the agreed starting task families. As of 2026-11-16, OpenCode compatibility is required. The exact capture mechanism, language, and tools remain open. Collect natural human–agent interactions; evaluate continuations from fixed conversation contexts and environment states using objective outcome checks.

The initial selector is independent of the downstream student. It should accept experience from different agent sources through a common episode format without needing student-specific scores. Cross-model transfer is a hypothesis to test, not a guarantee that every conversation is suitable for training.

## What must be true

1. Useful mistakes recur often enough that learning can repay its cost.
2. We can recover a valid training target from the interaction and its outcome.
3. The small model has enough capacity to learn the behavior.
4. A selection model adds value beyond random sampling and inexpensive rules.
5. Improvements transfer to unseen tasks rather than only replaying captured answers.
6. Selection, verification, training, and evaluation together fit the intended budget.

The key distinction is between a good response and a useful training example. A correct example may be redundant. The initial selector uses supported outcomes, meaningful corrections, and diversity without requiring knowledge of the student's weaknesses; downstream evaluations test whether this produces useful learning.

## Proposed product flow

1. Capture task context, attempts, tool results, corrections, and resolution through an available harness integration.
2. Apply inexpensive checks for missing outcomes, duplicates, and unusable records.
3. Use a decision model to nominate examples for learning or further verification.
4. Verify targets using available execution evidence or a separately defined review process.
5. Export a versioned dataset with provenance, selection reasons, and verification status.

A separate benchmark node consumes an export, trains a candidate LoRA adapter, and evaluates unseen tasks, retained skills, and total cost. Dataset generation should work without running this node.

Background processing means asynchronous work outside the request path. It does not make the compute free. Raw traces stay local by default; any hosted selector needs an explicit data boundary, and credentials and sensitive content must be excluded from training exports.

As of 2026-11-16, v1 includes batch and live background processing options over the same pipeline. Lightweight event capture is separate from model inference. Use bounded temporary capture, selective durable examples, and explicit expiry/eviction; do not archive all chats for hypothetical future classifiers. Exact limits and default processing mode are open. See [design direction](design-direction.md).

## Role of the decision model

Clef or Clef-Flash is a candidate selector, not an established judge for this workload. It could assess whether an episode contains a meaningful correction, sufficient context, and supported outcome evidence. It should be able to abstain and should not require a student-specific failure profile.

Execution checks establish supported outcomes where possible. Student performance after training establishes learning value. A selector's confidence is neither of those measurements.

## LoRA as a downstream benchmark node

LoRA fine-tuning is outside the core dataset product. It is one specific downstream benchmark to prove or disprove usefulness of the generated data. The benchmark produces an adapter, evaluation report, and provenance linked to a dataset version.

LoRA freezes the base weights and learns additional parameters; it is still a weight-learning method. Adapters make benchmark candidates smaller to store, version, compare, enable, and disable. They do not guarantee better accuracy, eliminate training cost, or prevent forgetting while enabled.

Start with one fixed base model and one active adapter per experiment. Adapter routing, composition, and a library of specialized skills remain later hypotheses.

## Possible local dashboard

Direction only, not a committed feature or selected UI stack: extend beyond a CLI into a local dashboard on top of the same local data. Users could inspect generated examples, trace them back to source episodes, inspect selection and verification status, and see where the data comes from.

Possible breakdowns include agent app, source model, project, and task type. Every percentage needs an explicit denominator: share of raw events, episodes, exported examples, or training tokens. Counts from different units should not be presented as interchangeable. OpenCode compatibility does not commit us to embedding this dashboard inside OpenCode.

## Open questions

- Which concrete tool-use and function-implementation tasks should form the first benchmark?
- Which OpenCode capture surface exposes enough context and outcome evidence in the version we use?
- Under equal capture limits, when does batch processing outperform live background processing in memory, cost, throughput, and dataset usefulness?
- What temporary queue and retained-dataset quotas fit actual agent use?
- What data shapes and capture mechanisms do we need to understand before implementing? See the [learning guide](data-and-capture.md).
- Will individual coding-agent users actually export and use these datasets, and what rough edges will they tolerate?
- What is the cheapest trustworthy way to verify a resolution?
- Does student-independent selection outperform random sampling and outcome-based rules across downstream models?
- Is retrieval of previous corrections sufficient for the same user benefit?
- What hardware, monetary budget, and completion time define affordable for this project?
- When does a general rule or harness fix solve the problem more directly than training?

## What would change our direction

If rules select equally useful data at lower cost, simplify the selector. If retrieval delivers the same benefit more cheaply, report that and reconsider adapter training. If outcomes cannot be verified, narrow the task or add explicit review. If the student cannot learn the behavior, reconsider the model or task before adding orchestration.
