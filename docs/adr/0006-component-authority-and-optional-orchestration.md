# ADR 0006: Give each runtime layer one state authority

- Status: Accepted (authority boundaries; integration contracts require a pilot)
- Date: 2026-09-27

ODAF owns fleet lifecycle and assignment state. Its guest supervisor prepares the selected profile and reports readiness. For the first interactive Pi pilot, Herdr owns the terminal pane and Pi process lifecycle; Pi owns its model/tool loop. An optional stablyai/orca integration may later own a separate run/worktree mode, but it is not required for core operation. The operator owns consequential approval. Each layer reports its own state; a running VM, live pane and successful task are not interchangeable. The accepted Hermes candidate is NousResearch/hermes-agent, not the fleet authority. See the [agent stack and authority matrix](../agent-stack-and-authority.md) and [routing decision](0007-work-routing-and-evidence.md). Reconnection, stop and uncertain-dispatch behavior must be verified in the pilot before relying on Herdr-managed runs.
