# Embedded Harness Awesome

A project building a background layer for coding-agent users to capture and curate high-quality training datasets from everyday conversations, corrections, and supported outcomes.

The product output is an inspectable dataset with provenance and verification status. LoRA fine-tuning is a downstream benchmark node to test the dataset's usefulness, rather than a required part of the product. The central experiment is whether selected experience improves future task performance enough to justify the full cost.

OpenCode compatibility is required. The initial capture mechanism is still to be evaluated. A local dashboard for inspecting data and source breakdowns is a possible later direction, not a committed deliverable.

Status: problem framing and experiment design. No implementation or measured results yet.

V1 will offer batch and live background curation over a bounded local capture queue. Neither mode requires archiving every conversation indefinitely. Next: review the literature and specify the high-level design together, then implement with DeepSeek.

## Thinking and research

- [Planning area](docs/planning/README.md)
- [First principles and product thinking](docs/planning/product-thinking.md)
- [Roadmap and experiment design](docs/planning/roadmap.md)
- [Literature review](docs/planning/literature-review.md)
- [Decision log](docs/planning/decisions.md)
- [Data shape and capture learning guide](docs/planning/data-and-capture.md)
- [Design direction and storage bounds](docs/planning/design-direction.md)
- [Handoff for specification and implementation](docs/planning/handoff.md)

This is a working hypothesis, not a claim of novel continual learning or guaranteed improvement.
