# ODAF decision status

- Updated: 2026-09-27
- Scope: design decisions, not a claim that the fleet is implemented.

This is an index. The linked ADR or domain document is authoritative for the rule; [Q8 research](q8-routing-and-parallelism-spec.md) preserves alternatives and evidence rather than changing accepted v1 policy. [Issue #1](https://github.com/BillBalint-SM/project-O.D.A.F./issues/1) is an earlier foundation spec; where it differs, the later accepted ADRs below govern the intended design.

## Accepted

| Area | Fixed decision | Source |
|---|---|---|
| Control | The operator's Windows PC is the Hyper-V control plane and the required host-local dashboard. PowerShell 5.1/Hyper-V cmdlets and the dashboard share one command/status contract; agent development runs in Ubuntu guests. | [ADR 0001](adr/0001-control-plane-and-runtime.md) |
| Fleet | Five dedicated specialist placements can be started or stopped as all five or any selected subset with one action. The baseline per VM is Generation 2, two vCPUs, four GiB fixed RAM and 70 GiB disk. Registry-driven operations are declarative, idempotent and support plans/dry runs. | [ADR 0004](adr/0004-fleet-lifecycle-and-validation.md), [README](../README.md) |
| Image | One versioned common image; each clone gets distinct machine/network/SSH identity and activates its versioned specialization after cloning. VM, guest, Linux login, agent and provider identities remain separate; credentials never enter the public repo or image. | [ADR 0002](adr/0002-layered-image-and-personalization.md), [ADR 0003](adr/0003-identity-and-secrets.md) |
| Development environment | The common image targets modern CLI, browser, JS/TS, Python and native build capabilities. Docker/infrastructure CLIs are available where useful; heavy daemons, clusters and local models are opt-in. Validation is local-first; hosted GitHub Actions is optional, manual or release-scoped. | [README technical baseline](../README.md#working-technical-baseline), [ADR 0004](adr/0004-fleet-lifecycle-and-validation.md) |
| Vocabulary | Operator, agent definition, agent instance, profile, guest, harness and session are separate concepts. General AI-coding language follows the linked dictionary. The operator is not the guest's Linux user. | [CONTEXT](../CONTEXT.md), [ADR 0005](adr/0005-ai-coding-language-and-agent-identity.md) |
| Five roles | Research/specification; architecture/planning; implementation A; implementation B; test/review/integration. Both implementers use Pi: A defaults to TDD-first, B to BDD-first; neither method is exclusive. Model and reasoning strength are chosen per work item, not baked into agent identity. | [Agent domain](agents/domain.md) |
| Runtime authority | ODAF owns fleet/assignment state, the guest supervisor prepares profiles and reports readiness, Pi owns its inner model/tool loop, and the operator owns the consequential verdict. The first interactive Pi pilot gives Herdr the pane/process lifecycle. Hermes is a candidate harness; Orca is an optional separate integration, not a prerequisite. | [ADR 0006](adr/0006-component-authority-and-optional-orchestration.md), [authority matrix](agent-stack-and-authority.md) |
| Assignment | Direct operator assignment or deterministic, eligibility-filtered recommendation with operator approval. One accountable lead per work item; one active item per VM. Only the selected stopped VM starts after approval. Busy or unavailable specialists queue; reassignment needs approval. | [ADR 0007](adr/0007-work-routing-and-evidence.md) |
| Collaboration | V1 supports solo, sequential and parallel read-only research/review. A lead may delegate logged, bounded, non-writing investigation inside the approved scope and budget. Parallel code writing is not a default mode. | [ADR 0007](adr/0007-work-routing-and-evidence.md) |
| Evidence and authority | Work states acceptance criteria and per-item resource limits. Pre-dispatch eligibility and action authorization precede work. Local scoped edits, tests and commits may proceed; push, PR, merge, deploy and destructive actions need separate approval. An independent test/review agent checks maker evidence before the operator accepts, redirects or blocks. | [ADR 0007](adr/0007-work-routing-and-evidence.md), [task flow](task-flow.md) |
| Evaluation | Automatic routing remains shadow-mode research until observed outcomes justify a separate explicit promotion decision. No numerical promotion thresholds are accepted. | [ADR 0007](adr/0007-work-routing-and-evidence.md) |
| Operator-plane direction | Evaluate Paperclip first for the Windows-local web work/agent surface. Pause overlapping custom dashboard and work-control implementation until a one-VM fit/gap pilot establishes the authority boundary. This is not a claim of successful integration or final platform adoption. | [ADR 0008](adr/0008-paperclip-first-operator-plane.md) |

## Open or deferred

| Area | Decision still needed / trigger |
|---|---|
| Dashboard delivery | Test Paperclip's Windows-local installation and one-VM operator flow before selecting a final UI, packaging and integration route. The local dashboard requirement itself is fixed. |
| Three non-implementation agents | Select their primary harnesses after the first pilot; define actual versioned skill/tool/permission profiles and final agent-to-VM mappings. Their five role names alone are not runnable configurations. |
| Pi/Herdr pilot | Verify restart, reconnection, stop, timeout and uncertain-dispatch behavior on a guest before relying on interactive Pi runs. Define any distinct headless mode only if the pilot needs one. |
| Budgets and routing promotion | Derive numerical agent/time/cost defaults, eligible auto-routing classes and promotion criteria from measured local work. Until then the operator supplies per-work-item limits and approves assignments. |
| Parallel code and Orca | Decide whether an isolated parallel-writing pilot is worthwhile, its Git integration/review path and whether/how optional Orca owns any run. One run must have one process owner. |
| Exact implementation | Pin image/tool versions and version-manager choice; define durable run-state storage and safe cancellation/retry after ambiguous dispatch; implement and test the command/status contract. These are implementation choices, not permission to silently change accepted boundaries. |

The repository is a documentation-and-contract foundation. Its dashboard, router, queue, image builder and guest supervisor do not exist as verified executable components yet.
