# Understanding the data and how we capture it

Created: 2026-11-16. Learning guide and design sketch, not an implemented schema or integration. Official documentation was checked on this date; verify the installed agent version before implementation.

## Start with four separate things

| Layer | Meaning | Example |
| --- | --- | --- |
| Raw event | Something observable happened in an agent session | User message, tool invocation, tool response, assistant message |
| Episode | Events grouped around a task or resolution | Request, failed implementation, correction, successful implementation, test evidence |
| Selection record | Our assessment of an episode | Accept as candidate, reject, or abstain; reasons and evidence references |
| Training export | A deliberately constructed example for downstream training | Conversation context followed by a supported assistant response or tool call |

A transcript is not automatically a training dataset. Selection decides which material is worth considering; verification records what evidence supports it; export decides what the model will see as input and what it will learn to produce.

These layers are logical data shapes, not a requirement to store every layer forever. V1 uses bounded temporary capture and quota-limited retained examples. Both batch and live background processing are in scope, with a shared data policy. See [design direction](design-direction.md) for bounds, scheduling tradeoffs, and unresolved numeric budgets.

## Agent app, source model, selector, and student

- Agent app/harness: OpenCode, Claude Code, or Codex. It manages sessions, tools, messages, and context. Capture usually attaches here.
- Source model: the model that produced an answer inside that app. An app can use different models, including within a session, so record this per event when exposed.
- Selector: Clef or another mechanism assessing the collected episode.
- Student: a downstream model trained on an export, used by our benchmark or by a user. Selection does not require its scores.

Supporting OpenCode is not the same as supporting only one model provider. Conversely, a model API alone may not show local file changes, actual tool execution, or subsequent human feedback.

## Walk through one episode

Illustrative sequence:

1. User: implement `normalize_tags` so tags are lowercase and duplicates are removed in first-seen order.
2. Assistant writes an implementation that lowercases but retains duplicates.
3. A test fails on repeated tags.
4. User clarifies that order must be preserved.
5. Assistant writes a corrected implementation.
6. Tests pass in a recorded repository state.

The capture layer records what happened without assuming that the final response is correct. The episode assembler links the tool invocations and results, corrections, code changes, and evidence. Selection may nominate this as a meaningful correction. Verification records the tests and their coverage limits.

There are at least two possible exports:

- Original request -> supported final implementation, if the original context contains everything needed to justify that implementation.
- Request + earlier attempt + correction -> repaired implementation, to teach recovery after feedback.

These teach different behaviors. Export policy should be explicit; do not silently mix them. Do not label a failed intermediate attempt as a successful target. Later test results can justify selecting a target without being included as information available before it was produced.

Independent held-out tasks must use different problems or appropriately separated task groups, not another turn from this same episode.

## Proposed common event shape

This is our conceptual format, not the native schema of any agent. Fields, naming, and storage remain open.

```json
{
  "schema_version": "draft-0",
  "event_id": "event-004",
  "session_id": "session-001",
  "turn_id": "turn-002",
  "source": {
    "agent_app": "opencode",
    "agent_version": null,
    "model_provider": null,
    "model_id": null
  },
  "kind": "tool_result",
  "tool_call_id": "call-001",
  "observed_at": "2026-11-16T12:00:00Z",
  "sequence": 4,
  "payload": {
    "tool_name": "run_tests",
    "exit_code": 1,
    "output_ref": "artifact-test-output-001"
  },
  "context_ref": "context-001",
  "raw_ref": "source-record-004",
  "redaction_status": "pending"
}
```

Preserve native IDs and source timestamps when available, in addition to our ingestion IDs/time. `null` means unknown, not an inferred value. A tool result needs a link to its invocation. Large artifacts can be referenced rather than duplicated. A command exiting successfully is execution evidence, not automatically task success.

Context records may need repository revision, working-tree changes, relevant files, tool definitions, runtime/dependency versions, and instructions. Record what is available and what is missing; do not assume a transcript is enough to replay the task. Avoid collecting unrelated files or credentials.

## Episode, selection, and export records

An episode should reference ordered event IDs, its task description, relevant context, outcome evidence, and source provenance. Retained examples must include or retain their required evidence within the storage quota; temporary source records may expire. Branches, retries, interrupted sessions, and multiple tasks in one conversation need explicit handling; a session ID is not necessarily a task ID.

A selection record should contain an episode ID, bounded decision and reason codes, rubric/selector version, evidence references, and any measured scores. Prefer predefined options rather than requiring generated prose. Keep verification status separate: unknown, supported by a named check, or independently reviewed. Exact enums remain undecided. A positive user reply and a model confidence score do not independently establish correctness.

Distinguish `other` (understood but outside the defined categories) from `uncertain` (insufficient evidence to classify). Structured output is not a guarantee of deterministic or correct judgment. Code can apply a versioned acceptance policy to the model's fallible classifications.

An export should link back to the episode and selection record and specify:

- Ordered context messages and tool definitions needed by the example.
- Supported assistant response or action to learn.
- Which assistant turns/tokens receive training loss; user messages and tool results generally provide context, not assistant targets.
- Data split, provenance, verification evidence, and export-policy version.
- Any source-to-target tool mapping or model-specific chat formatting applied downstream.

The common dataset should preserve meaning without depending on one student's chat template. Formatting for a particular model belongs at the export consumer boundary. An adapter cannot make a tool available if the receiving harness does not implement it.

## Possible v2 export: tool-call accept/deny classification

Direction only: the same captured experience might support training a classifier to assess whether a proposed tool call is authorized under a specified policy and user intent. This is a different downstream use case from selecting training episodes. An episode being useful for training does not imply that every action in it should be allowed.

Aaron's proposed input view is recent user instructions excluding agent responses, together with the proposed tool call. Treat this as a baseline to test, not a settled context policy. Earlier user instructions may still apply. Assistant claims are not independent evidence of user authorization, but discarding all assistant/tool history can remove necessary factual context.

For example, "edit the file you just found" requires identifying that file from previous tool activity. A user-only view may be insufficient. Preserve message roles and provenance within the bounded capture window, then construct appropriate input views from retained examples. Do not keep every agent response indefinitely for this speculative use case; skip a v2 example when necessary context has expired.

Proposed pre-call input components:

- Applicable user instructions and explicit approvals up to the decision point, including earlier constraints still in force.
- Proposed tool name, arguments, and tool definition; a name alone does not describe the action.
- Relevant environment facts, such as resolved target paths and repository state.
- An explicit policy and version defining what accept/deny means.
- Selected prior tool results or conversation references when needed to resolve the request, with their source roles preserved. Treat factual context separately from authorization.

Possible labels are `allow`, `deny`, and `needs_review` for ambiguous cases; exact options and any execution behavior remain undecided. Do not equate denial with a failed tool execution, or successful execution with authorization. Logged human approval/denial can be evidence, but may reflect convenience or preferences rather than the policy we want to teach. Record label source and policy separately; missing approval is not a denial label.

The classifier input must contain only information available before the proposed call. Later results or replies may help review the label but cannot leak into the pre-call input. Capture proposed, blocked, cancelled, and executed calls distinctly where the integration exposes them; observing only executed calls does not provide a representative deny class.

Before adopting this direction:

- Compare user-only context with user instructions plus relevant prior context on fixed labeled cases.
- Establish policy-grounded reference labels, including authorized, unauthorized, and ambiguous examples.
- Measure false allows, false denials, review rate, and latency, rather than only overall accuracy.
- Check which pre-call and approval events the installed OpenCode version actually exposes.

No live permission enforcement or authorization-classifier training is part of v1. Live background dataset curation is a separate v1 option. Within the existing capture and retention budget, keep event ordering, roles, call arguments, approval references, and provenance where available. Supporting a possible v2 export must not require unbounded collection.

## Capture mechanisms in plain language

| Mechanism | How it works | Main question for us |
| --- | --- | --- |
| Local hook | Agent calls our handler at lifecycle events | Which events and fields are actually exposed? |
| Plugin | An extension runs inside the agent's supported extension system | Can we observe normal usage without changing it? |
| Event stream | Our process subscribes to events emitted by a running agent/server | Can we reconnect, recover missed events, and retrieve full content? |
| History import | Read an explicit export or supported session-history interface after use | Does it preserve tools, edits, branches, and outcome evidence? |
| SDK/app-server wrapper | Our app starts or drives agent sessions and records their events | Does this require users to work through our interface? |
| HTTP webhook | A source sends events to an HTTP endpoint we provide | Does the agent actually expose this, and what does delivery contain? |

A hook is not necessarily a webhook. MCP can expose tools to an agent, but an MCP tool server does not automatically observe every conversation or every other tool call. A model-request proxy also does not necessarily see tool execution outcomes. These are architectural distinctions, not interchangeable capture options.

## OpenCode: required compatibility target

Official sources checked:

- [Plugin overview](https://opencode.ai/docs/build/plugins/): server plugins can add hooks and integrations; plugins can also have TUI/GUI parts. This supports investigating an in-agent collector rather than requiring a separate chat interface.
- [API reference](https://opencode.ai/docs/api): documents session and event surfaces, including tool/session event types. It labels the HTTP API surface experimental; event delivery does not by itself guarantee a complete, durable transcript.
- [V1 server docs](https://opencode.ai/v1/docs/server/): describe a headless HTTP server and event access. These are explicitly older docs and must not be mixed with newer API examples.

Our proposed investigation order is a plugin collector for ordinary OpenCode use, then event/history access for recovery and imports. This is a hypothesis, not a selected implementation. Check the installed version and exact hooks first. Supporting a local collector does not require building a GUI plugin.

Questions to answer with a tiny fixture session:

1. Can we observe user/assistant messages, tool inputs and results, and model identity?
2. Are hooks pre-execution or post-execution, and can capture stay off the critical path?
3. Are streamed text fragments updates to one message, or separate records we must assemble?
4. Can we recover after a restart without missing or duplicating events?
5. How are compaction, branches, subagents, and edits represented?
6. Which context and outcome evidence must we record ourselves?

## Other agents: comparison surfaces, not current commitments

Claude Code documents lifecycle hooks and a transcript path in hook input. Investigate event payloads plus transcript access rather than assuming one hook contains the whole session. [Official hooks reference](https://code.claude.com/docs/en/hooks).

Codex offers structured JSONL events for noninteractive runs and an app-server protocol for applications managing conversations. These are documented integration surfaces, but driving a session is different from passively collecting an existing user's session. [Official noninteractive documentation](https://learn.chatgpt.com/docs/non-interactive-mode), [official app-server documentation](https://learn.chatgpt.com/docs/app-server).

No Codex, Claude Code, or OpenCode integration has been configured or validated by this project yet. Native local transcript formats, where available, also need version and stability checks before relying on them.

## Suggested learning sequence

- [ ] Explain event vs episode vs selection vs export in your own words.
- [ ] Check the installed OpenCode version and read the matching integration docs.
- [ ] Make a small fixture conversation with one function request, one tool execution, and one correction.
- [ ] Inspect its native data manually; identify IDs, ordering, inputs, results, and missing context.
- [ ] Map the fixture to the proposed common format on paper before building a collector.
- [ ] Construct one repair example and one direct-solution example; explain what each teaches.
- [ ] Define evidence sufficient for acceptance and when the selector must abstain.
- [ ] Only then choose the first capture mechanism and minimum schema.

Confusion Compass: which layer is unclear, how does a tool result link to the action that caused it, and what information belongs in the training input versus its target? State a guess, check it against the fixture, and explain the mapping using a second task.

## Optional dashboard direction

The same records could later support browsing examples and showing source composition locally. For example, 60 exported examples from OpenCode out of 100 total exported examples is 60% by example count; it says nothing about the share of tokens or raw events. Keep source app, source model, task type, and verification status as separate dimensions. No dashboard is required for the initial capture experiment.
