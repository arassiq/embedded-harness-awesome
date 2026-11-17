# Design direction before specification

Updated: 2026-11-16. This consolidates decisions and hypotheses for the next high-level design discussion. It is not an implementation specification. Numeric budgets, APIs, storage technology, and exact schemas remain undecided.

## Product boundary

The product captures and curates inspectable training datasets from ordinary coding-agent use. The initial audience is individual agent users interested in adapting smaller models to their workflows. OpenCode compatibility is required. Selection remains independent of a downstream student's scores or failure profile.

The output is selected examples with sufficient context, provenance, bounded classifications, and explicit verification status. A model-selected candidate is not automatically a golden example.

LoRA training is a separate downstream benchmark node that consumes an export to test usefulness. It does not need to run for collection, curation, or export to work.

## Processing flow

```text
OpenCode events / supported history access
                   |
          lightweight capture
                   |
         bounded local pending queue
                   |
       shared episode assembly and checks
                   |
        selector + verification evidence
                   |
      quota-limited self-contained examples
                   |
           versioned dataset export
                   |
       optional downstream LoRA benchmark
```

The diagram describes responsibilities, not a selected deployment or framework. Redaction and size checks should happen before durable storage. Store enough evidence to interpret a retained example; omit or skip examples that cannot fit the budget without losing essential meaning.

## Two v1 processing options

Both batch and live processing are in v1 scope. They use the same example semantics and selection policy. Neither mode implies pre-call permission enforcement.

| Option | Behavior | Tradeoff to measure |
| --- | --- | --- |
| Batch | Collect within limits; process on demand or on a schedule | Can keep selector unloaded between runs, but needs queue space and delays availability |
| Live | Process incrementally in an asynchronous worker when enough context exists | Earlier decisions, but potentially sustained model memory and competition with the coding agent |

Live does not mean calling a model for every token or tool event. Cheap capture and checks can run first; incomplete episodes can remain pending until sufficient evidence or expiry. A task may require later corrections or test results before it becomes eligible, regardless of mode.

Batch does not mean unlimited capture until a distant job runs. Both modes need the same queue limits, overflow policy, per-record limits, and bounded decision metadata. Backend batching, concurrency, and model loading are separate settings from when work is triggered. Default mode is still open.

## Bounded storage is a v1 constraint

Do not archive full conversations indefinitely for hypothetical future uses. Capture enough recent context temporarily, then retain only self-contained selected examples and necessary evidence.

| Store | Intended contents | Required bounds |
| --- | --- | --- |
| Pending capture | Recent events/artifacts awaiting episode assembly and review | Byte cap, per-record limit, expiry, overflow behavior |
| Selected dataset | Accepted examples, necessary context/evidence, and provenance | Total byte quota, per-example limit, explicit replacement/eviction policy |
| Decision/audit sample | Small samples of rejects or uncertain cases for error analysis, if enabled | Separate fixed quota and expiry; no permanent log for every rejection |
| Aggregate metrics | Counts by controlled source/task/decision categories | Bounded category cardinality and time retention |
| Exports and caches | Dataset versions, temporary processing files, model caches | Explicit retention/accounting; versioning must not silently duplicate data forever |

Expiry alone does not bound storage under high event rates. A fixed episode count also does not bound bytes when outputs vary in size. Metrics, evidence references, and metadata need bounds as well as message bodies.

Exact values and eviction policy require a workload measurement. A candidate implementation must handle full queues without blocking normal agent work indefinitely, record truncation or loss, and ensure retained examples do not depend on deleted context. References to externally owned agent history may expire and cannot promise replayability.

Deleting data sacrifices late feedback and retrospective reprocessing. Accept this explicitly; skip incomplete examples rather than reconstructing missing evidence. If v2 needs context that has expired, that example is unavailable to v2.

## Bounded decisions

Use predefined classifications and reason codes; keep evidence type, task type, context sufficiency, and export decision separate. `other` indicates an understood category outside the taxonomy; `uncertain` indicates insufficient evidence. Store selector/rubric versions and evidence links where retained.

Model judgments remain fallible. Code can apply a deterministic eligibility policy to structured outputs. A tool returning exit code zero, user satisfaction, or model confidence alone is not a universal correctness certificate.

## Evaluation questions before choosing defaults

The first task families are tool use and function implementation. Freeze conversation prefixes, environment state, available tools, and attempt budgets across comparisons. Use independent outcome checks; prevent later corrections/solutions from leaking into evaluation inputs.

Compare random sampling, simple success/correction rules, cheap trace signals, and model-based selection from a common eligible stream. Assess downstream success at matched training-token budgets and total cost at a common quality target. Report sample counts, uncertainty, retained-skill changes, and coverage of sources/task types. Trace informativeness and classifier confidence are intermediate metrics, not downstream learning results.

For batch versus live, replay the same event stream under the same data policy. First fix rubric/version and complete-episode input to test processing overhead; separately measure effects of acting on partial or earlier context. Otherwise a timing difference could actually be a different-data experiment.

Measure peak collector and selector RAM/accelerator memory separately, disk high-water mark, queue age and loss, calls/tokens, throughput, end-to-end example availability, and agent slowdown. Async processing can still consume substantial memory; batch processing can still consume substantial disk.

## Later directions

- A local dashboard for inspecting examples and source proportions, built on the same records. Percentages need explicit units and denominators.
- A v2 tool-call accept/deny export with only pre-call context and policy-grounded labels, separate from dataset eligibility. It must fit the retention budget.
- Cloud-hosted asynchronous selection and optional cloud storage in v2/v3. Bedrock is a candidate to investigate, not a selected backend or a claim that Clef is supported. Check model/import compatibility, data boundary, request limits, cost, and retention before selecting it. Remote inference may reduce local model memory but does not eliminate queues or storage quotas.
- Learned scheduling, adapter routing, and model-specific optimization only after a simpler baseline provides evidence.

## Next phase

After this literature pass, work through a high-level specification with Aaron. Then implement with DeepSeek using a reviewed handoff. The forthcoming awesome-harness handoff capability is expected but not assumed available. No capture integration or runtime code is implemented yet. See the [handoff](handoff.md) for the next discussion.
