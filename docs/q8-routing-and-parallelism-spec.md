# Q8.2 and Q8.4: work routing and parallel agent operation

- Status: Research synthesis and alternatives; v1 policy is accepted in [ADR 0007](adr/0007-work-routing-and-evidence.md).
- Date: 2026-09-27
- Scope: Five specialist agent definitions placed in Ubuntu Hyper-V guests, operated from the Windows-hosted ODAF dashboard and its shared command/status contract.

This document refines the earlier [Q8 research](research-q8-dashboard-routing-accountability.md). Q8.1's host-local dashboard requirement and the Q8.2/Q8.4 v1 policy in [ADR 0007](adr/0007-work-routing-and-evidence.md) are accepted. The tables below preserve alternatives and trade-offs, not unresolved v1 choices. No ODAF routing or multi-agent implementation has been measured yet.

## Problem Statement

The operator can prepare a work item alone or with an agent, then assign it directly or submit it for routing. ODAF has not decided how to select a specialist, when to start or queue its guest, whether to involve other specialists, or how to preserve accountability when they work concurrently. Treating these as one "autonomy" switch could waste host and provider capacity, dispatch to an ineligible profile, create conflicting edits, or blur the operator's approval authority.

## Solution

Keep four decisions separate: **assignment** (which agent definition or instance owns the work item), **placement and scheduling** (which guest is ready or must start, and when), **collaboration topology** (solo, sequential, or concurrent work), and **authorization** (which actions need approval). Direct operator assignment must remain possible. Any automated decision must be inspectable and overridable; it must not grant tool permissions or final publication authority. The Windows control plane holds the assignment record; the selected guest and harness execute the work. [ODAF task flow](task-flow.md), [authority matrix](agent-stack-and-authority.md)

The simplest candidate for five fixed specialists is explicit capability and permission filtering, followed by operator-visible recommendation. Policy-based auto-assignment for bounded, reversible work and dynamic supervision are later options to evaluate, not requirements. A router can classify once; a supervisor can make new delegation decisions during a multi-step run. Direct handoff gives the specialist control of the turn, whereas manager-as-tool keeps synthesis and the user-facing result with the lead. [LangChain router](https://docs.langchain.com/oss/python/langchain/multi-agent/router), [LangChain subagents](https://docs.langchain.com/oss/python/langchain/multi-agent/subagents), [OpenAI agent orchestration](https://openai.github.io/openai-agents-python/multi_agent/)

## User Stories

1. As the operator, I want to choose a specialist directly, so that the dashboard never prevents an intentional assignment.
2. As the operator, I want to submit a prepared work item without naming a specialist, so that ODAF can help find a suitable owner.
3. As the operator, I want to see the proposed owner, alternatives, and reason, so that I can correct a poor recommendation before dispatch.
4. As the operator, I want ODAF to say that no agent is eligible, so that missing capabilities or permissions are not hidden by a forced choice.
5. As the operator, I want to override a proposed or automatic assignment, so that I retain control when context changes.
6. As the operator, I want assignment and guest startup shown separately, so that I know whether work is assigned, queued, or actually running.
7. As the operator, I want only the needed VM subset started, so that idle guests do not consume host capacity.
8. As the operator, I want the work item's allowed actions and approval needs visible before execution, so that routing does not silently expand authority.
9. As the operator, I want one accountable owner and one review location for each work item, so that multiple contributions do not obscure responsibility.
10. As the operator, I want a solo path for tightly coupled work, so that unnecessary coordination does not slow it down.
11. As the operator, I want optional sequential delegation for dependent steps, so that each specialist receives the previous step's verified result.
12. As the operator, I want independent research or review tasks to run concurrently when worthwhile, so that I can gain time or perspectives without shared edits.
13. As the operator, I want bounded concurrent implementation tasks in isolated branches and clones, so that conflicting work can be identified and integrated deliberately.
14. As the operator, I want to inspect validation, integration, cost, and review evidence for the entire work item, so that a worker's completion is not mistaken for an accepted outcome.
15. As the operator, I want interruption, stale observations, retries, and duplicate submissions identified, so that a lost connection does not create duplicate or falsely completed work.
16. As a specialist agent, I want a bounded brief with expected artifacts and permitted scope, so that I can work without guessing the other agents' responsibilities.
17. As a lead agent, I want to receive each worker's artifact and validation result, so that I can integrate and hand off one coherent result.
18. As a future implementer, I want routing and topology evaluated against solo/manual baselines, so that complexity is added only when it demonstrably helps ODAF.

## Implementation Decisions

### Existing accepted boundaries

- The Windows control plane and host-local dashboard own fleet and assignment records. The command interface and dashboard share one observable contract; guests run agent instances, not the fleet control plane. [ADR 0001](adr/0001-control-plane-and-runtime.md)
- A work item may originate with the operator alone or in collaboration with an agent. Direct assignment and submission for routing both remain available. An assignment is not a VM start, run completion, validation result, or approval. [ODAF language](../CONTEXT.md), [task flow](task-flow.md)
- The operator retains consequential approval. Profiles declare specialist capabilities and permissions; credentials are not baked into the common image or broadened by a routing decision. [Authority matrix](agent-stack-and-authority.md)
- Each guest has its own Git clone. A Git worktree is attached to **one repository**; worktrees on separate VM clones do not form a shared worktree. Cross-VM changes need a deliberate branch/commit and fetch, push, or patch integration path. [Git worktree documentation](https://git-scm.com/docs/git-worktree)

### Q8.2 researched options (v1 selection in ADR 0007)

| Dimension | Options | Trade-off or trigger |
|---|---|---|
| Operator control | Direct; recommendation with confirmation; policy auto-assign for eligible classes; continuously supervising agent | Increasing flexibility also increases audit, failure, and cost burden. Keep direct override in every mode. |
| Selector | Explicit rules; weighted capability/availability score; LLM classification; retrieval/embedding match; supervised multi-label model; state-aware/adaptive router | Five fixed profiles favor a small declared registry first. Learned routing needs representative outcome labels, not merely prompt similarity. |
| Cardinality | One owner; one owner plus consultants; several deliberately selected contributors; abstain/no match | A work item may span specialties, but selecting too many raises cost. Multi-label research supports measuring both coverage and over-selection, not assuming either is always better. [Set-valued routing paper](https://arxiv.org/abs/2606.28925) |
| Scheduling | Start eligible stopped guest; queue until ready; decline; escalate to operator | Host RAM/CPU, guest resource profile, provider budget, and readiness must be checked separately from skill fit. |
| Delegation control | Host policy chooses; lead agent requests a bounded worker; specialist handoff | Manager-as-tool, handoff, and code-driven orchestration have different state and accountability semantics. [OpenAI orchestration](https://openai.github.io/openai-agents-python/multi_agent/) |
| External interoperability | Fixed local registry; optional A2A-style discovery/protocol later | A2A describes discoverable capabilities and task operations, but is not required merely because five known VMs exist. [A2A specification](https://a2a-protocol.org/latest/specification/) |

An eligible candidate must pass hard checks before ranking: required capability and toolset, profile permission, correct project/repository access, trustworthy guest/harness readiness, available capacity, risk policy, and task constraints. If any required fact is missing or stale, the decision must abstain or ask the operator; an LLM score is not authorization. A work-item contract should retain identifier and source, brief/spec and acceptance criteria, repository/base revision, required capabilities, allowed actions, risk, resource and time budgets, candidate/owner, policy version, rationale, override, and required artifacts. These are contract fields to decide, not an implemented schema.

The route-to-agent boundary should be checked *before* execution. SDK-level input guardrails can run concurrently with the agent and therefore may block only after the agent has already used tools; agent-level guardrails also do not automatically cover every handoff or tool call. ODAF needs its own pre-dispatch policy and action-level approval boundary regardless of harness. [OpenAI guardrails](https://openai.github.io/openai-agents-python/guardrails/), [human-in-the-loop](https://openai.github.io/openai-agents-python/human_in_the_loop/)

### Q8.4 researched options (v1 selection in ADR 0007)

| Topology | Appropriate when | Main cost or failure mode |
|---|---|---|
| Solo | One coherent change or a task with tight shared context | May miss a useful second perspective. |
| Sequential pipeline or handoff | One step depends on another's output | Longer wall-clock time, but simple integration. |
| Lead with parallel read-only consultants | Independent research, architecture checks, or reviews | Synthesis and provider cost. |
| Lead with independent implementation workers | Clearly separable deliverables and merge plan | Edit overlap, stale bases, integration and test time. |
| Redundant attempts or voting | Alternative solutions or independent verification are valuable | Cost and reviewer work; agreement is not proof. |
| Implementer plus independent reviewer/evaluator | Risk or quality warrants another perspective | Extra latency; verifier quality limits value. |
| Dynamic supervisor or decentralized swarm | Subtasks emerge during a long task and simpler topologies fail | Harder budget, state, retry, and accountability management. |

These patterns are documented by [Anthropic's workflow taxonomy](https://www.anthropic.com/engineering/building-effective-agents), [OpenAI's orchestration guide](https://openai.github.io/openai-agents-python/multi_agent/), and [Microsoft's orchestration patterns](https://learn.microsoft.com/en-us/agent-framework/workflows/orchestrations/). They are options, not evidence that all should be implemented.

One accountable lead per work item is accepted v1 policy. The lead owns decomposition, dependency order, bounded briefs, integration, validation evidence, and handoff; the operator owns final acceptance and publication. Worker scopes should not overlap accidentally. A task claim/lease prevents unintended duplicate work; intentionally redundant attempts must be named as such. Runs need explicit cancellation, timeout, retry, and uncertain/stale-state behavior. No worker should write directly to an uncoordinated shared main branch. The practical elapsed-time comparison must include setup, the slowest worker, integration, tests, and operator review, not just worker completion.

The research supports caution: a controlled study of 260 configurations found multi-agent performance highly task-dependent, including gains on decomposable reasoning and losses on sequential planning. [Kim et al.](https://arxiv.org/abs/2512.08296) A software-engineering study found gains from dependency-aware centralized delegation, isolated workspaces, and test-based integration on its benchmarks; a separate study cautions that isolation can defer rather than prevent merge conflicts. Neither predicts ODAF's results without a local pilot. [Geng and Neubig](https://arxiv.org/abs/2603.21489), [STORM](https://arxiv.org/abs/2605.20563) Anthropic's research-system account reports much higher token use for its multi-agent design and notes that coding is often less parallelizable than research. Its compiler-agent case study used separate clones and task locks but still encountered frequent merge conflicts. [Research system](https://www.anthropic.com/engineering/multi-agent-research-system), [compiler case study](https://www.anthropic.com/engineering/building-c-compiler)

## Testing Decisions

The highest existing proposed test seam is the Windows control-plane command/status contract, observed through dry-run plans and machine-readable results; the dashboard should use the same behavior rather than have a second decision engine. No executable tests or prior routing tests exist yet. Real Hyper-V/SSH/guest interaction remains a separate representative-guest and five-guest smoke test. [ADR 0004](adr/0004-fleet-lifecycle-and-validation.md)

- Test external behavior: the same work-item input and declared fleet state produce an explainable candidate set, selection or abstention, and no dispatch to an ineligible target. Avoid tests that assert internal scorer implementation.
- Test direct assignment, recommendation/override, no match, unavailable and stale guests, missing permissions, conflicting constraints, and multi-specialty work.
- Test that assignment, VM start/queue, run state, validation, and approval remain distinct; a connection loss must not assert task failure or completion without evidence.
- Test duplicate submission, interrupted dispatch, retry, cancellation, timeout, and late worker result without double execution or lost artifacts.
- Test solo, sequential, read-only parallel, and isolated implementation cases at the control/status boundary; verify owner, provenance, worker outputs, integration and approval gates.
- Test security invariants before dispatch and before sensitive tools: routing may not widen credentials, allowed files, network scope, or approval rights.
- Smoke-test cross-VM Git artifact transfer and integration on actual guests; a successful worker run does not count until final tests and operator review are recorded.

For an ODAF pilot, collect representative real work items by type and risk. First compare router proposals in **shadow mode**, with the operator making the real assignment. Evaluate eligibility, abstention, accepted outcome, rework, elapsed time, cost, and override burden; operator agreement alone is not quality. Then compare bounded auto-routing only on reversible low-risk classes if the operator accepts its policy. For Q8.4, compare matched tasks under solo, sequential, and bounded parallel modes; include final integration and review in wall-clock time, provider cost, guest/host peak load, merge conflicts, validation failures, and reopens. Stratify by whether subtasks were genuinely independent. No benchmark result supplies a universal ODAF promotion threshold.

The earlier research note's suggested 50/90%/25% routing and 10/20%/10% concurrency gates are **historical pilot hypotheses, not adopted policy or evidence-based universal thresholds**. Set numerical gates only after observing ODAF baselines and selecting acceptable risk. Safety invariants (no unauthorized action, no out-of-scope dispatch, no duplicate consequential operation) should not be traded for speed.

## Out of Scope

- Implementing a router, queue, dashboard, agent supervisor, Git synchronization service, or VM lifecycle action in this document.
- Automatically launching all five VMs for every work item, dynamic multi-host scheduling, or mandatory A2A/LLM routing.
- Treating a routing recommendation as permission to access secrets, publish, merge, or perform destructive actions.
- Treating the researched alternatives as already implemented or silently promoting them over the accepted v1 policy.

## Further Notes

**Deferred choices:** the concrete UI stack, harnesses for three non-implementation roles, numerical budget defaults, promotion criteria for automatic routing, and an isolated parallel-code-writing or optional Orca pilot. Direct assignment, approved recommendation, one lead, the v1 collaboration modes and evidence gate are fixed in ADR 0007. This document records research options rather than claiming implementation.

All external sources linked above are first-party documentation or original research papers, accessed 2026-09-27. Vendor examples and benchmark scores are not ODAF field measurements. Live documentation can change; implementation should pin relevant versions. The earlier [Q8 source ledger](research-q8-dashboard-routing-accountability.md#dated-source-ledger) covers the Q8.1/Herdr/Orca context.
