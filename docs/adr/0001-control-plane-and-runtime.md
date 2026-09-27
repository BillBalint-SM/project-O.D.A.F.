# ADR 0001: Separate the Windows control plane from the Ubuntu agent runtime

- Status: Accepted (placement and host-local dashboard; implementation technology open)
- Date: 2026-09-26

## Context

The project needs five independently specialized agents on one Windows workstation. The operator wants to see, control and instruct any subset of them from that PC, without using a guest VM as the control station. Hyper-V is available on the host, while development tools and AI harnesses belong inside Linux guests. Mixing those responsibilities makes lifecycle operations, testing and recovery difficult.

## Decision

The Windows host is the ODAF control plane **and the operator's dashboard machine**. The first usable release includes a host-local, unified interface for observing the five VMs and agents, selecting any subset, starting and stopping it, assigning work, and reviewing progress and results. The dashboard uses the same ODAF control/status contract as the command line; neither surface maintains a separate fleet truth.

The host uses PowerShell 5.1 and native Hyper-V cmdlets for VM lifecycle, checkpoints, network attachment, selection and fleet-level health. Each Ubuntu guest runs the agent supervisor, workspace, harness and development tools. The host reaches guests through authenticated SSH and does not run an agent's normal development workload. Herdr and optional Orca may supply terminal, worktree and review views, but ODAF remains the unified fleet and assignment surface. Closing the dashboard does not stop guests or their supervised work.

## Consequences

- Host automation can be tested without giving guest processes Hyper-V privileges.
- The graphical dashboard is a required operator surface, not a future optional replacement for the CLI. Its framework and packaging remain implementation choices.
- Dashboard actions must use the same authority, validation and status model as command-line actions. A disconnected or stale guest is reported as unknown rather than silently treated as stopped.
- Guest behavior can be rebuilt or personalized without changing host lifecycle code.
- SSH health and identity become explicit contracts.
- A small amount of cross-platform contract testing is required.
