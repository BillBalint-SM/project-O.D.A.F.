# ODAF agent stack and authority matrix

This is the accepted division of responsibilities. The optional Orca integration targets [stablyai/orca](https://github.com/stablyai/orca). Its concrete integration contract remains an open design question.

| Layer | State authority | Product or component | Responsibility |
|---|---|---|---|
| Work intake and assignment | Operator or ODAF control plane | Host-local ODAF dashboard and shared command interface | Direct assignment or approved, deterministic recommendation under [ADR 0007](adr/0007-work-routing-and-evidence.md) |
| Fleet and VM | Hyper-V, observed by ODAF | ODAF control plane on Windows | VM placement, create/reconcile, start/stop, checkpoints and selected fleet status |
| Guest readiness | Guest operating system, reported by ODAF | ODAF guest supervisor | Profile activation and readiness report; no competing owner for the interactive Pi process |
| Terminal and pane | Herdr server on each guest | [Herdr](https://herdr.dev/docs/agent-automation/) | Persistent terminal sessions and interactive Pi process lifecycle in the first pilot, agent detection and remote operator attachment |
| Run and worktree | Orca when enabled | [stablyai/orca](https://github.com/stablyai/orca), optional | Worktree isolation, agent runs and review support after assignment |
| Agent loop and session | The selected harness | [Pi](https://github.com/badlogic/pi-mono), [Codex CLI](https://github.com/openai/codex), [Hermes Agent](https://github.com/NousResearch/hermes-agent), or another approved harness | Model requests, context, tool calls, permissions and harness session |
| Inference | Model provider | Provider selected by the profile and runtime credential | Model availability, requests, rate limits and provider-side failures |
| Approval and publishing | Operator | Human review | Consequential approval, final review and shipping |

Herdr is a terminal/session layer. Its agent state reports what it detects inside a pane; ODAF must still report VM and supervisor state separately. The intended placement is one Herdr server per active guest with the Windows host connecting over SSH.

The operator's Windows PC is the dashboard machine. The first usable ODAF release must present one local view of all five declared agents, their distinct readiness and task states, and actions for any selected subset. The dashboard and command interface share the ODAF control/status contract; neither Herdr nor Orca is the fleet authority. Closing the dashboard leaves guests and supervised work running, and reopening it refreshes observed state. The dashboard's UI framework and packaging are still open.

Orca is the optional [stablyai/orca](https://github.com/stablyai/orca) run/worktree layer. ODAF lifecycle and direct assignment must work without it. ODAF records the intended assignment; Orca reports execution state when enabled. Orca also offers a desktop interface, remote SSH worktrees and terminals, so its optional use overlaps with Herdr's terminal role. The first interactive Pi pilot uses Herdr as process owner; any later Orca-managed run needs a separately defined, single process owner.

Hermes means the [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) project. It is a harness candidate, including its own CLI, gateway, memory and scheduling features. If a profile selects Hermes, ODAF should integrate with the required harness behavior without duplicating Hermes's internal loop or treating its gateway as the fleet authority.

An agent definition selects one primary harness. Both implementation roles select Pi; the other three harness selections follow the pilot. An agent instance runs one primary harness at a time; a fallback may be selected by a later explicit policy. ODAF's harness-facing contract should be thin: launch, identify, read readiness, observe exit and stop. In the first interactive Pi pilot Herdr launches and owns the pane/process, while the guest supervisor prepares the profile and reports readiness. Reconnection, stop and uncertain-dispatch semantics still need verification. ODAF does not implement another model/tool loop.

An ODAF start request succeeds only after the selected guests report the assigned profiles and primary harnesses ready for work. The status result must show VM, SSH, guest, Herdr and harness readiness separately. Starting a fleet and assigning a work item are distinct operations.

## Example state disagreement

If Hyper-V says a VM is running, SSH responds, Herdr sees an idle Codex pane and Orca marks a task failed, these reports are all valid at their own layers. ODAF status should retain each field and explain that the task failed while the guest and harness remain available. A single green/red flag would hide the cause.

If Herdr loses its client connection, the guest VM and running harness may continue. If the guest supervisor fails, a persistent terminal pane does not by itself make the agent ready.
