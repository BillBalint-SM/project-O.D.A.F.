# ODAF work preparation and assignment

The operator may develop an idea, plan, document or spec alone or in conversation with an agent. That produces a work item. The operator can then assign the work item directly to a specialist agent or submit it to the ODAF control surface for routing and validation. Both paths must be available; the control surface does not replace human planning.

## Accepted flow

1. The operator prepares a work item alone or with an agent.
2. The operator chooses direct assignment to a named agent or submits it to the ODAF control surface.
3. For control-plane assignment, ODAF applies the eligibility checks and deterministic recommendation in [ADR 0007](adr/0007-work-routing-and-evidence.md). The operator inspects and approves the proposed target; no-match abstains.
4. ODAF records the assignment. If necessary it starts only that agent's stopped VM, waits for readiness, then dispatches. A busy or unavailable target queues; changing the target requires operator approval.
5. The selected agent runs in its own guest, using its profile and one primary harness. One VM handles at most one active work item in v1.
6. The guest keeps its own Git clone. Work is isolated in a task-specific worktree when worktrees are used; no shared writable host checkout is the source of truth.
7. The maker returns artifacts, tests, diff summary, known gaps and risk notes against stated acceptance criteria. A separate test/review agent checks them; the operator accepts, redirects or blocks. Push, PR creation, merge, deployment and destructive actions require separate approval.

GitHub Issues or a direct operator description can provide a work item. The ODAF task contract should preserve source attribution without requiring every exploratory idea to become a GitHub Issue first.

## Host-local operator dashboard

The operator works from one dashboard on the Windows PC, not from a guest VM. It shows the five declared agent/VM placements, selected subset, readiness by layer, assigned work, progress, validation evidence and the operator decision still needed. The operator can start or stop any selected subset and assign a prepared work item directly or through the control plane. Detailed Herdr terminal and optional Orca worktree/review views may be opened from the dashboard, but an assignment and fleet status must remain understandable without them. The dashboard and command interface use one ODAF control/status contract.

Closing the dashboard does not stop a guest or supervised task. On reopening, the interface reconciles observations from Hyper-V and the guests; it shows stale or unreachable observations explicitly rather than inventing a completed or stopped state.

## Separate states

Fleet readiness answers whether a selected agent can receive work. Assignment answers which agent should do a work item. Run state answers what happened during execution. Validation answers whether the result meets the work item's checks. The operator owns final consequential approval. A failure in one state must not be reported as a failure of every layer.

The agreed start contract reaches a ready profile and primary harness, not merely a powered-on VM. Status reports VM, SSH, guest, Herdr and harness readiness as separate fields. This does not assign a work item.

## Runtime authentication

Provider authentication belongs to the selected harness inside the guest. ODAF may report whether the profile is usable, but it does not collect or print provider credentials. The base image and public example configuration contain no live authentication material.

## Pilot and implementation details still to resolve

- Dashboard UI technology and packaging; the host-local dashboard requirement is already accepted.
- The primary harnesses for research, planning and review; the two implementation agents use Pi.
- Numerical agent/time/cost limits after a measured pilot, and exact retry/cancellation behavior after uncertain dispatch.
- The separate parallel-code-writing pilot and optional Orca integration. Solo, sequential and parallel read-only modes are accepted v1 policy.
