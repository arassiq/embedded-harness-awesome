# Decision log

## 2026-11-16: Bound collection and retention

Status: agreed constraint; numeric limits and eviction policy to specify.

Use bounded temporary capture and selectively retained, self-contained training examples. Apply byte quotas, per-record limits, expiry, and explicit overflow handling. Rejects need not become permanent individual logs; any audit sample and aggregate metrics must also be bounded. Speculative v2 use cases do not justify retaining every chat/tool result indefinitely. This clarifies earlier advice to preserve original context: retain necessary roles/evidence within budget, not unlimited raw history.

## 2026-11-16: Offer batch and live curation in v1

Status: agreed v1 options; default and triggers remain open.

Separate lightweight event capture from model processing. Batch and live asynchronous workers share episode/export semantics, verification standards, and bounded storage policies. Live curation does not mean synchronous permission enforcement or model assessment of every event. Compare resource use and data outcomes before choosing defaults. Cloud processing, potentially through Bedrock if compatible, is a later v2/v3 direction.

## 2026-11-16: Review literature, specify together, then implement with DeepSeek

Status: agreed phase order.

Complete this literature pass and discuss its implications, then work through the high-level specification with Aaron. Implementation follows with DeepSeek and an explicit handoff. Aaron expects an awesome-harness handoff change; do not assume its interface or availability. Current design-direction notes are preliminary and do not constitute the final specification.

## 2026-11-16: Explore tool-call accept/deny datasets as v2

Status: possible downstream use case; input view and label policy not selected.

Captured tool activity could support a separate export for training tool-call authorization classifiers. Aaron proposes recent user instructions without assistant responses as context. Evaluate that view against one with relevant prior factual context; preserve the original roles and events so both remain possible. Include proposed call arguments and an explicit policy, and exclude future outcomes from pre-call inputs.

This is distinct from accepting an episode into a training dataset. An allowed action need not succeed, and a successful action need not be authorized. Approval labels need provenance and review. Ambiguous cases should be representable rather than forced into allow/deny. No live enforcement is committed for v1. See the [design notes](data-and-capture.md#possible-v2-export-tool-call-acceptdeny-classification).

## 2026-11-16: Use bounded classification options

Status: agreed design direction; exact schemas remain open.

Use predefined decision and reason options rather than requiring free-form assessment text. Keep `other` separate from uncertainty or abstention. Record evidence references and rubric/selector versions. Bounded output does not make model judgment deterministic or correct; a versioned code policy can determine export eligibility from those classifications.

## 2026-11-16: Dataset curation is the product; LoRA is a benchmark node

Status: agreed product boundary; supersedes the product-output framing in the 2026-11-12 adapter entry.

Capture and curate high-quality datasets from everyday agent experience so users can fine-tune smaller models for their workflows. Export datasets with provenance and verification status. LoRA training sits downstream as a separate benchmark node to demonstrate the data's usefulness. Using the dataset product does not require running our training pipeline.

## 2026-11-16: Focus first on individual coding-agent users

Status: agreed audience choice; rationale remains a product hypothesis.

The two candidate audiences are specialized companies building custom tools and agent users fine-tuning smaller models for their own workflows. Start with the second. The hypothesis is that motivated early users will accept less polish in exchange for useful datasets. Validate actual fine-tuning demand and willingness to use the export rather than assuming this applies to all agent users.

## 2026-11-16: Support OpenCode

Status: required compatibility target; capture mechanism not selected.

The project must work with OpenCode. Investigate its version-specific plugin hooks, event interfaces, and session-history access. Preserve a common episode format so other agent integrations can be added without changing selection or dataset semantics.

## 2026-11-16: Explore a local dashboard later

Status: direction only, not a committed feature, implementation, or deadline.

A local dashboard could let users inspect generated data, provenance, verification decisions, and source percentages. Define whether a percentage counts events, episodes, exported examples, or tokens. Neither a frontend stack nor an embedded OpenCode UI has been selected.

## 2026-11-16: Learn capture and data shapes before implementation

Status: immediate learning priority.

Understand raw events, assembled episodes, selection records, and training exports. Compare hooks, streams, history imports, and wrappers using official agent documentation. Walk through one small example manually before settling the integration and schema. See the [data and capture guide](data-and-capture.md).

## 2026-11-13: Start with tool use and function implementation

Status: agreed starting task families.

Use tool-use tasks in a fixed environment and function-implementation tasks with independent tests as the first evaluation settings. Define success before running the experiments. For tool use, check the required outcome rather than requiring one exact call; for implementations, check test success within a fixed attempt budget.

Collect experience from natural human–agent conversations, but evaluate with a fixed conversation prefix, repository/environment state, available tools, and attempt budget. A continuation may span several agent steps. Do not include future corrections or solutions in the evaluation prefix.

Why: preserves the conversational setting while preventing changing human responses from obscuring model comparisons. These evaluations establish bounded task execution, not yet improved live human collaboration.

Exact tools, language, fixtures, and harness integration remain open.

## 2026-11-13: Keep selection independent of the downstream student

Status: agreed initial product boundary.

Select examples using conversation context, corrections, tool evidence, and supported outcomes, without requiring student-specific scores or failure profiles. Aim to accept conversations from different agent sources through a common episode format; start with one integration.

Why: preserves modularity between experience collection, selection, and downstream fine-tuning. Not every conversation will yield a usable example, and cross-model learning benefits still need evaluation.

## 2026-11-12: Start with selection from everyday agent experience

Status: agreed direction; starting task families chosen on 2026-11-13. Exact workload and integration remain open.

Build a background layer that collects agent experience and nominates useful, verified training examples. Test whether those examples improve a smaller model relative to random selection and simple rules.

Why: connects everyday use to a measurable learning problem and combines useful systems work with an experiment we can realistically evaluate.

Reconsider if: verified outcomes are too scarce, inexpensive rules perform equally well, or selection costs outweigh its benefit.

## 2026-11-12: Use LoRA adapters as the initial training artifact

Status: historical initial direction; clarified on 2026-11-16 as a downstream benchmark choice, not the product output. Subject to hardware and student compatibility checks.

Freeze a fixed base model and train versioned candidate adapters. Store the dataset provenance, configuration, and evaluation report alongside each adapter.

Why: makes adaptation artifacts easier to store, compare, enable, and disable. This is still weight learning; it does not imply guaranteed performance or negligible training cost.

Reconsider if: the selected task cannot be learned through the supported adapter configuration or the training backend makes another approach materially simpler.

## 2026-11-12: Treat Clef as a candidate, with baselines

Status: proposed experimental design.

Keep the selector interchangeable. Compare model-based selection with random sampling and inexpensive outcome-based rules. Independently verify targets and evaluate student learning.

Why: decision-model accuracy and confidence do not establish downstream learning value. The selector must justify its own compute cost.

Reconsider if: evidence identifies a clearly better selector or removes the need for model-based selection.

## 2026-11-12: Defer a learned training controller

Status: proposed sequencing.

Start with fixed training and evaluation procedures. Learn when to train, how to allocate a budget, or how to route adapters only after establishing that selected experience improves the student.

Why: separates data-selection effects from scheduling and optimization effects, keeping the first experiment interpretable.
