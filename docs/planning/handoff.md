# Handoff for the next phase

Updated: 2026-11-16. Planning checkpoint, not authorization to skip the design discussion.

## Current state

The repo contains product thinking, decisions, an expanded literature review, and proposed architecture/evaluation notes. No collector, selector, dashboard, training pipeline, or performance results exist yet.

Sequence agreed with Aaron: literature review -> collaborative high-level specification -> implementation with DeepSeek. An awesome-harness change is expected to support handoff between agents; its mechanism is not available or specified here. DeepSeek's implementation role does not select the source model, selector, or downstream benchmark student.

## Read first

1. [Product thinking](product-thinking.md): user and problem.
2. [Decision log](decisions.md): agreed boundaries and historical changes.
3. [Literature review](literature-review.md): findings, limitations, dates/counts, and remaining reading.
4. [Design direction](design-direction.md): preliminary responsibilities, two processing modes, and storage constraints.
5. [Data and capture guide](data-and-capture.md): data layers and version-specific integration questions.
6. [Roadmap](roadmap.md): experiments and proposed milestones.

## Preserve these constraints

- Initial product: high-quality dataset capture/curation/export for coding-agent users.
- OpenCode support required; common data semantics should allow other agents later.
- No student-specific input required by selection.
- Tool use and function implementation are the first evaluation families.
- LoRA is a separate benchmark node, not the product output.
- Both live background and batch curation are v1 options; no requirement to assess every token/event.
- Bounded capture and retention; do not keep all transcripts indefinitely for v2.
- Bounded classification outputs with abstention, evidence status, and versioned policy.
- Dashboard, cloud processing, and tool-call authorization classification are later directions.
- Do not claim novelty, correctness, efficiency gains, or finished implementation without evidence.

## Resolve in the high-level specification

- Installed OpenCode version and one supported observation surface, validated on a tiny fixture.
- Minimum event/episode/export shapes and enough context for reproducible checks.
- What qualifies for acceptance and when to abstain.
- Batch/live triggers, default mode, model lifecycle, concurrency, and overload behavior.
- Measured workload and numeric RAM, byte, record, queue, and retention budgets.
- Storage/export versioning and eviction without dangling evidence references.
- Concrete tool/function tasks, independent graders, splits, and compute budget.
- Selector and cheaper baselines; benchmark student/backend chosen independently.

## First implementation slice after specification

Prefer an end-to-end fixture path: a short OpenCode episode -> bounded capture -> assembled episode -> structured decision -> inspectable export. Prove correct linkage and retention behavior before adding model serving or training. This is a proposed slice to agree during specification, not a finalized implementation plan.

Do not inspect or export private session histories wholesale merely to bootstrap the fixture. Use a small deliberate test session and preserve Aaron's ability to inspect each transformation.
