# ADR 0007: Route bounded work with one owner and evidence-based review

- Status: Accepted (design; implementation and pilot pending)
- Date: 2026-09-27

## Context

The operator prepares work on the Windows PC and needs to assign it to one of five specialist agents without confusing assignment, VM startup, execution, verification and approval. Routing and parallel execution must not silently widen permissions, duplicate work or turn an agent's self-report into an accepted result. [Q8 research and alternatives](../q8-routing-and-parallelism-spec.md) remain background, not v1 policy.

## Decision

- The operator may assign directly or request a recommendation. In v1 the router applies hard capability, permission, project-access, capacity and known VM-startability checks, then a simple deterministic ranking. Actual guest/harness readiness is checked after any required start and before dispatch. The router explains candidates and may abstain. The operator approves a recommendation before dispatch; automatic assignment is evaluated in shadow mode only.
- One specialist is the accountable lead for each work item, selected for its primary deliverable. The lead may delegate bounded, logged, non-writing investigation within the approved scope and budget. Delegation does not add permissions or change the accountable owner.
- Approved assignment may start only the selected stopped VM, then waits for profile/harness readiness. One VM handles at most one active work item in v1. If its designated specialist is busy or unavailable, the item queues with the reason visible; rerouting needs operator approval. Unknown or stale state is not reported as idle or completed.
- V1 supports solo work, sequential handoff, and parallel read-only research or review where useful. Parallel code writing is a separate, isolated-workspace pilot with an explicit integration plan, not a default mode. Each guest keeps its own repository clone.
- Each work item states acceptance criteria, scope, permitted actions, required evidence and a limit on agent count, elapsed time and provider cost. The operator sets per-work-item limits until pilot measurements justify numerical defaults. Exhaustion pauses work and asks the operator for direction; it does not silently escalate resources or switch provider/model.
- Local scoped edits, tests and commits may proceed under an approved work item. Push, PR creation, merge, deployment and destructive actions require separate operator approval. Pre-dispatch eligibility and action-level authorization precede execution; completed work receives post-run validation.
- The maker tests and reports its work. A separate test/review agent checks the change and evidence before operator acceptance. The handoff includes artifacts, test results, diff summary, known gaps and risk notes against the stated criteria. The operator decides whether to accept, redirect or block, and owns consequential publication.
- Compare recommendations to actual operator assignments and outcomes in shadow mode. Measure eligibility, abstention, overrides, quality, rework, time and cost before proposing auto-routing. Any promotion requires a separate explicit decision; no numerical threshold is accepted yet.

The five role assignments and the two Pi implementation defaults are recorded in [agent domain context](../agents/domain.md). Model and reasoning strength are selected per work item within the profile's allowed capabilities and budget, not permanently attached to an agent identity. The first interactive Pi pilot uses the Herdr process boundary in [ADR 0006](0006-component-authority-and-optional-orchestration.md).

## Consequences

The dashboard and command interface must show assignment, queue, VM/guest/harness readiness, run, validation and operator verdict as distinct states. The host control plane records the assignment and evidence; the selected guest executes. Safety checks must be testable locally through the shared command/status contract before a real five-VM smoke test. This ADR specifies intended behavior, not an existing router, queue or review implementation.
