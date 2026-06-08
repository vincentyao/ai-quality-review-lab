# Agent Runtime Reliability Checklist

A QA and release-readiness checklist for reviewing AI Agent systems before broader testing, pilot usage, or production release.

This checklist focuses on **runtime reliability**, **tool safety**, **traceability**, **cost control**, **permission boundaries**, and **failure recovery**.

It is designed for AI products where an agent can perform multi-step reasoning, call tools, interact with external systems, read or write data, or trigger user-facing actions.

---

## 1. Scope of Review

This checklist is not a model benchmark.

It does not only ask:

```text
Is the model smart enough?
```

It asks:

```text
Can this agent behave reliably inside a real product environment?
```

The review covers:

* Agent loop control
* Tool calling reliability
* Permission boundaries
* Runtime state management
* Memory safety
* Human-in-the-loop design
* Trace and replay capability
* Cost and latency control
* Failure handling
* Release readiness

---

## 2. Agent Runtime vs Evaluation Harness

Before reviewing an agent system, distinguish two different concepts:

### Agent Runtime Harness

The runtime system that allows an agent to operate.

It usually includes:

* Model interface
* Agent loop
* Tool registry
* Runtime state
* Memory
* Permissions
* Channels
* Logging
* Human review
* Cost guardrails

### Agent Evaluation Harness

The evaluation system used to test and compare agent behavior.

It usually includes:

* Task definitions
* Eval cases
* Graders
* Metrics
* Transcripts
* Repeated trials
* Reports
* CI / regression gates

Both are important, but this checklist focuses on **runtime reliability**.

---

## 3. Agent Loop Control

### Review Questions

* Is there a clearly defined agent loop?
* Does each agent turn have a maximum number of steps?
* Is there a timeout for each task or trajectory?
* Can the agent stop safely when it cannot complete a task?
* Are infinite loops prevented?
* Are repeated tool calls detected?
* Can the system distinguish between intermediate reasoning, tool calls, and final answers?
* Is there a clear fallback when the agent is stuck?

### Risk Indicators

* No maximum step limit
* No timeout
* No stop condition
* Repeated tool calls without progress
* Agent keeps asking itself or tools the same thing
* Final response generated without checking task completion

### Suggested Checks

* Run tasks that require multiple steps
* Run tasks that cannot be completed
* Run tasks with missing information
* Run tasks where tools return errors
* Verify whether the agent stops safely

---

## 4. Tool Registry and Tool Calling

### Review Questions

* Are tools registered in a central registry?
* Does each tool have a clear name, description, input schema, and output schema?
* Are tool inputs validated before execution?
* Are tool outputs normalized before being returned to the agent?
* Are tool failures handled explicitly?
* Are dangerous tools separated from low-risk tools?
* Are tools versioned or documented?
* Can unused or experimental tools be disabled?

### Risk Indicators

* Tools are called through loose string matching
* Tool schemas are missing or vague
* Tool arguments are not validated
* Tool errors are returned directly without structure
* High-risk tools are exposed by default
* The agent can call tools it does not need

### Suggested Checks

* Test invalid tool arguments
* Test missing required fields
* Test malformed tool output
* Test unavailable tools
* Test tool timeout behavior
* Test whether the agent retries safely or fails gracefully

---

## 5. Permission Boundaries

### Review Questions

* Are tools classified by permission level?
* Are read-only actions separated from write actions?
* Are irreversible actions protected by explicit confirmation?
* Are high-risk actions blocked by default?
* Does the agent operate with least privilege?
* Is user authorization checked before tool execution?
* Are system-level permissions separated from user-level permissions?

### Suggested Permission Tiers

```text
Tier 0: Read-only, low-risk actions
Tier 1: User-visible but reversible actions
Tier 2: Write actions requiring user confirmation
Tier 3: Irreversible or high-risk actions requiring human approval
Tier 4: Prohibited actions
```

### Risk Indicators

* All tools share the same permission level
* Agent can perform write actions without confirmation
* Agent can access data outside the current user scope
* Agent can call admin-level tools during normal user sessions
* No approval flow for high-impact actions

---

## 6. Human-in-the-Loop Controls

### Review Questions

* Which actions require human confirmation?
* Is human review required before irreversible operations?
* Can the agent ask for clarification when confidence is low?
* Can a human override or stop the agent?
* Is the confirmation message understandable to the user?
* Is the approval event logged?

### Good HITL Patterns

* Confirm before sending external messages
* Confirm before writing or deleting data
* Confirm before making financial, legal, medical, or account-level recommendations
* Confirm before triggering third-party system actions
* Confirm when confidence is low or context is incomplete

### Risk Indicators

* Agent acts autonomously in sensitive workflows
* User approval text is vague
* Human approval is not logged
* Agent continues after user rejection
* The user cannot inspect what will happen before approval

---

## 7. Runtime State Management

### Review Questions

* Is task state stored explicitly?
* Can the system resume safely after interruption?
* Is state separated by user, session, and task?
* Can stale state affect a new task?
* Is temporary state cleared after task completion?
* Are partial failures recorded?
* Can the agent recover after process restart?

### Risk Indicators

* State exists only inside prompt history
* User sessions share memory or context accidentally
* Failed actions leave inconsistent state
* Retried tasks duplicate previous actions
* There is no idempotency key for write operations

### Suggested Checks

* Refresh the page during an agent task
* Restart the backend during execution
* Run two sessions in parallel
* Retry the same task twice
* Verify whether duplicate actions occur

---

## 8. Memory Safety

### Review Questions

* What types of memory does the agent use?
* Is memory write controlled?
* Can users inspect or delete remembered information?
* Can outdated memory influence future answers?
* Is sensitive information stored accidentally?
* Is memory scoped correctly by user or organization?
* Is retrieved memory ranked, filtered, or validated?

### Memory Types to Review

```text
Working memory: short-term task state
Episodic memory: past events or interactions
Semantic memory: facts, preferences, or knowledge
Vector memory: retrieved chunks or embeddings
```

### Risk Indicators

* Agent stores everything by default
* Memory writes happen without user awareness
* Sensitive data appears in memory or logs
* Old memory is treated as current truth
* Memory from one user affects another user

---

## 9. Traceability and Replay

### Review Questions

* Is every agent run assigned a unique run ID?
* Is every step recorded?
* Are model inputs and outputs logged safely?
* Are tool calls, tool arguments, and tool results recorded?
* Are latency, token usage, and cost tracked?
* Can failed runs be replayed or reconstructed?
* Are sensitive fields redacted in traces?

### Minimum Trace Fields

```text
run_id
user_id or session_id
task
step_id
timestamp
model
input_summary
tool_call
tool_args
tool_result
final_output
latency_ms
token_usage
estimated_cost
error
approval_event
```

### Risk Indicators

* Failures cannot be reproduced
* Tool calls are not logged
* Logs contain sensitive secrets
* No link between user complaint and agent trace
* No visibility into token cost or latency

---

## 10. Cost and Latency Guardrails

### Review Questions

* Is there a maximum token budget per run?
* Is there a maximum cost per run?
* Is there a maximum number of tool calls?
* Are expensive tools controlled?
* Is latency tracked by model call and tool call?
* Is fallback behavior defined when limits are reached?
* Can runaway loops be stopped automatically?

### Risk Indicators

* No token or cost limit
* Agent can keep calling tools indefinitely
* Expensive model is used for every step
* No alert for unusual cost spikes
* No timeout for slow tools

### Suggested Guardrails

```text
Max steps per task
Max model calls per task
Max tool calls per task
Max token budget
Max cost budget
Max runtime duration
Fallback model or safe failure response
```

---

## 11. Error Handling and Recovery

### Review Questions

* Are tool errors handled gracefully?
* Does the agent explain failures clearly to the user?
* Can the agent recover from transient failures?
* Are retries controlled with limits?
* Is retry behavior idempotent?
* Are partial results handled safely?
* Are severe failures escalated to humans or support?

### Error Types to Test

* Tool timeout
* Tool unavailable
* Invalid tool response
* Permission denied
* Missing user data
* Conflicting context
* Rate limit
* Model refusal
* Empty retrieval result
* Unexpected exception

### Risk Indicators

* Raw stack traces shown to users
* Agent hallucinates tool success after failure
* Agent retries indefinitely
* User sees a generic error with no recovery path
* Backend logs error but frontend shows success

---

## 12. Security and Tool Misuse

### Review Questions

* Are external inputs treated as untrusted?
* Are tool calls protected by allowlists?
* Are domains, paths, and actions restricted?
* Are secrets redacted from prompts, tool results, and traces?
* Are prompt injection scenarios tested?
* Are high-risk tools isolated?
* Are supply-chain risks considered for third-party tools?

### Common Agent Runtime Risks

```text
Prompt injection
Tool misuse
Over-permission
Data leakage
Confused deputy
Unsafe autonomy
Supply-chain risk
Secret exposure
```

### Suggested Checks

* Inject malicious instructions through retrieved content
* Ask the agent to ignore system rules
* Ask the agent to access unauthorized data
* Ask the agent to call tools outside the task scope
* Provide tool output containing fake instructions
* Verify whether secrets appear in final answers or traces

---

## 13. User Trust and Explanation

### Review Questions

* Does the agent explain what it is doing?
* Does the user know when a tool is being used?
* Are high-impact outputs explained?
* Does the agent distinguish facts, assumptions, and uncertainty?
* Are recommendations framed safely?
* Can users understand why a decision or suggestion was made?

### Risk Indicators

* Agent makes confident claims without evidence
* Agent gives financial, legal, medical, or account-level suggestions without caveats
* Agent hides tool usage from users
* Agent cannot explain its result
* Users cannot challenge or correct the result

---

## 14. Release Readiness Criteria

Before releasing an agent feature to real users, the following should be true:

* Core workflows have smoke tests
* High-risk workflows have explicit test cases
* Agent loop has max steps and timeout
* Tool calls are schema-validated
* Tool permissions are tiered
* High-risk actions require confirmation
* Trace logging is available
* Sensitive data is redacted from logs
* Cost and latency are monitored
* Basic eval cases exist
* Failure paths are tested
* Human escalation path is defined
* Product owner understands known limitations

---

## 15. Suggested Review Output

A runtime reliability review should produce:

```text
1. Summary of reviewed agent flows
2. Key reliability risks
3. Tool and permission risk assessment
4. Failure mode observations
5. Traceability and observability gaps
6. Cost and latency risks
7. Human-in-the-loop recommendations
8. Priority fixes before broader release
9. Suggested eval cases
10. Release readiness decision
```

---

## 16. Severity Model

### P0 - Release Blocker

Issues that can cause unsafe actions, data leakage, irreversible user impact, or uncontrolled agent behavior.

Examples:

* Agent can execute high-risk tools without approval
* User data can leak across sessions
* No stop condition for autonomous actions
* Secrets appear in prompts, traces, or outputs

### P1 - High Risk

Issues that significantly reduce reliability, trust, or operational safety.

Examples:

* No trace for failed runs
* No retry limit
* Tool errors are not handled
* Score or recommendation lacks explanation
* No permission separation between read and write tools

### P2 - Medium Risk

Issues that affect usability, debuggability, or maintainability.

Examples:

* Poor error messages
* Missing loading states
* No cost breakdown
* Incomplete empty states
* Weak observability

### P3 - Improvement

Useful enhancements that improve quality but do not block early testing.

Examples:

* Better dashboard for agent traces
* More granular metrics
* Better documentation
* Improved test data coverage

---

## 17. Practical QA Test Ideas

### Basic Runtime Tests

* Agent completes a simple task successfully
* Agent handles missing input
* Agent asks for clarification
* Agent stops after max steps
* Agent handles tool timeout
* Agent handles tool error
* Agent avoids repeated tool calls

### Tool Safety Tests

* Agent attempts unauthorized tool call
* Agent receives malicious tool output
* Agent is asked to perform a high-risk action
* Agent is asked to access data outside the current user scope
* Agent is asked to ignore system rules

### Reliability Tests

* Same task executed multiple times
* Task executed under slow tool response
* Backend restarted mid-task
* Partial failure occurs after one successful tool call
* Multiple user sessions run in parallel

### Trust Tests

* Agent explains why it made a recommendation
* Agent cites data source or tool result
* Agent acknowledges uncertainty
* Agent refuses unsupported claims
* Agent gives safe next steps after failure

---

## 18. Final Review Question

The most important question is not:

```text
Can the agent complete the happy path?
```

The real question is:

```text
Can the agent fail safely, recover clearly, and remain understandable to users when the environment becomes messy?
```

If the answer is no, the agent is not ready for broad release.
