# Project O.D.A.F.

ODAF is a local, reproducible fleet for five specialized AI-development agents running in Ubuntu Hyper-V guests on a Windows host.

## Status

This repository contains an accepted design for the Windows control plane, common-image fleet, agent identities, runtime authority and v1 work policy. The executable control plane, dashboard, router, queue and guest supervisor have not been implemented. The [unified implementation plan](docs/implementation-plan.md) is the single delivery roadmap; the [decision status index](docs/decision-status.md) separates accepted decisions from open ones. [ADR 0007](docs/adr/0007-work-routing-and-evidence.md) defines the accepted work policy and [agent domain context](docs/agents/domain.md) names the five specialist roles.

[GitHub Issue #1](https://github.com/BillBalint-SM/project-O.D.A.F./issues/1) and [Issue #2](https://github.com/BillBalint-SM/project-O.D.A.F./issues/2) remain source specifications. The unified plan reconciles both; later accepted ADRs take precedence where their early suggestions differ.

## Goal

From one dashboard on the Windows host, the operator should be able to:

- create or reconcile the five VM instances from one golden image;
- start or stop all agents or any selected subset with one action;
- see all five VMs and agents in one view, with distinct VM, SSH, guest, harness and task states;
- activate the correct specialized profile for each agent;
- accept work prepared by the operator alone or with an agent, then assign it directly or submit it for control-plane routing;
- inspect the selected agent, progress, validation evidence and result before consequential approval;
- recover from drift or a failed guest without rebuilding the fleet manually.

## Architecture

~~~text
Windows host
└── ODAF control plane
    ├── local operator dashboard and command interface
    ├── one control/status contract and fleet registry
    ├── Hyper-V lifecycle and checkpoints
    ├── SSH, guest and harness readiness
    └── work assignment and review state
        │
        ├── Ubuntu VM: agent-01 ── guest supervisor ── profile-01
        ├── Ubuntu VM: agent-02 ── guest supervisor ── profile-02
        ├── Ubuntu VM: agent-03 ── guest supervisor ── profile-03
        ├── Ubuntu VM: agent-04 ── guest supervisor ── profile-04
        └── Ubuntu VM: agent-05 ── guest supervisor ── profile-05
~~~

The host manages VMs and presents the unified operator view. The guest supplies the environment. A profile describes agent specialization. The image provides common capabilities. Herdr and optional Orca add specialized terminal/worktree views without becoming the fleet dashboard. See the [agent stack and authority matrix](docs/agent-stack-and-authority.md).

## Working technical baseline

| Area | Decision |
|---|---|
| Host control plane | Windows PowerShell 5.1 plus native Hyper-V cmdlets |
| Operator dashboard | Required host-local interface backed by the ODAF control/status contract; UI technology not yet selected |
| Guest OS | Ubuntu Server LTS |
| VM baseline | Generation 2, two vCPUs, four GiB fixed RAM, 70 GiB disk |
| Guest supervisor | Small Python service managed by systemd |
| Common shell | Bash and tmux |
| Core source tools | Git, Git LFS, GitHub CLI, curl, wget, rsync, jq, yq, ripgrep, fd, fzf |
| Native build tools | build-essential, GCC/Clang, CMake, Ninja, pkg-config, ccache |
| JavaScript | Node.js LTS, npm, pnpm, TypeScript project tooling |
| Python | Python, uv, ruff, pytest, pyright |
| Version policy | mise or an equivalent explicit per-project tool-version manager |
| AI CLIs | Codex CLI, Pi, Claude Code, Gemini CLI and Aider available without credentials |
| Hermes | NousResearch Hermes Agent is an additional harness candidate |
| Terminal sessions | Herdr server on each guest, accessed from the host over SSH |
| Task/worktree orchestration | [stablyai/orca](https://github.com/stablyai/orca) is an optional integration |
| Browser | Playwright with Chromium as the baseline; browser MCP and other engines are profile capabilities |
| Containers | Docker CLI, Buildx and Compose v2; daemon activation is opt-in |
| Infrastructure | kubectl, Helm, k9s, Terraform/OpenTofu and Ansible |
| Local models | Not part of the common image; opt-in profile capability |
| Validation | Local-first; GitHub Actions manual or release-scoped only |

The four-GiB baseline is for one lightweight agent workload. Heavy browser use, large builds, Docker workloads, multiple simultaneous CLIs or local inference need a larger resource profile.

## Repository shape

~~~text
/
├── AGENTS.md
├── README.md
├── CONTEXT.md
├── .gitignore
├── config/
│   └── odaf.example.json
├── scripts/
│   └── README.md
├── guest/
│   └── README.md
├── profiles/
│   └── README.md
├── docs/
│   ├── agents/
│   │   ├── domain.md
│   │   └── issue-tracker.md
│   ├── agent-stack-and-authority.md
│   ├── decision-status.md
│   ├── implementation-plan.md
│   ├── q8-routing-and-parallelism-spec.md
│   ├── research-q8-dashboard-routing-accountability.md
│   ├── task-flow.md
│   └── adr/
│       ├── 0001-control-plane-and-runtime.md
│       ├── 0002-layered-image-and-personalization.md
│       ├── 0003-identity-and-secrets.md
│       ├── 0004-fleet-lifecycle-and-validation.md
│       ├── 0005-ai-coding-language-and-agent-identity.md
│       ├── 0006-component-authority-and-optional-orchestration.md
│       └── 0007-work-routing-and-evidence.md
└── .scratch/                 # local-only, ignored, not canonical
~~~

The repository intentionally does not contain empty five-fold profile folders, a fake CI workflow, a local Kubernetes cluster, a model runtime or a copied VM disk. Those become real artifacts only when the first implementation needs them. `.git/` is local Git metadata, not repository content; `.scratch/` is ignored operator-local work.

## Intended operator surface

The first usable release has a unified local dashboard on the operator's Windows PC. It lists the five declared agents and their VM placements, separates observed readiness from active task state, and shows the time of the last observation. A closed dashboard does not stop guests or supervised tasks; on reopening, it reconciles displayed state with the host and guests. It supports selecting any one-to-five-agent subset, one-action start/stop, direct work assignment or submission for routing, inspection of the selected target, progress and validation review, and operator approval. Stopping a guest with active work requires an explicit impact warning and confirmation. A failed or unreachable layer is shown explicitly rather than flattened into one green/red status.

The dashboard and command interface call the same control/status operations. Herdr or Orca may be opened for detailed terminal, worktree or diff work; they are not required to understand fleet readiness or make an assignment. The local dashboard does not imply a cloud service or an internet-facing multi-user system.

These are target contracts, not implemented commands yet:

~~~text
odaf validate
odaf plan --agents all
odaf start --agents all
odaf start --agents agent-01,agent-03
odaf stop --agents agent-02
odaf status
odaf reconcile
~~~

The command surface must support dry-run output, deterministic selection, idempotent reconciliation, machine-readable status and clear refusal when a requested capability exceeds the selected resource profile.

The start operation reaches agent readiness: each selected VM, guest, assigned profile and primary harness must be ready for work. Assigning a work item is a separate operation. The [task flow](docs/task-flow.md) records direct assignment, operator-approved recommendation, queueing and evidence review. A stopped assigned VM may start after approval; a busy or unavailable specialist queues rather than silently rerouting. The two implementation agents use Pi with TDD-first and BDD-first defaults; model and reasoning choice is per work item.

## Intended audience and first supported environment

ODAF is intended to be reproducible by another developer from a clean machine. The reusable core and example configuration belong in the public repository; operator-specific profiles and credentials may remain private. The first supported environment is one trusted operator on one Windows/Hyper-V host. Multi-host scheduling and an internet-facing multi-tenant service are outside the first release.

Named agent VMs are durable. Tasks, worktrees and harness sessions can be short-lived inside them. The Windows command interface and machine-readable status form the control contract used by the required host-local dashboard. The control plane, golden image and profiles have separate versions and must declare compatibility before deployment.

English is the canonical language of the repository and agent documentation. A Hungarian operator guide can be added for user-facing setup without keeping two complete, drifting copies of the technical specification.

## Image and fleet lifecycle

1. Build and version the common Ubuntu development image.
2. Clone the image into five VM instances.
3. Personalize each clone with unique hostname, machine identity and SSH host keys.
4. Attach the declared network and assign the profile identity.
5. Start the selected subset.
6. Verify guest supervisor and agent health through the control-plane status seam.
7. Checkpoint before risky image or fleet changes.

Credentials are injected at runtime or through a controlled secret mechanism. They are never baked into the image or committed configuration.

## Development workflow

Local validation is the default. The future validation command should run PowerShell/Pester/PSScriptAnalyzer checks, shell checks, JSON/config validation and documentation consistency without GitHub Actions credentials or provider credentials.

GitHub Actions may be added later for manual validation, release artifacts or a trusted self-hosted Windows integration runner. Hosted GitHub runners are not assumed to provide Hyper-V integration.

## Implementation roadmap

Use the [unified implementation plan](docs/implementation-plan.md) for delivery order, work-package dependencies, release gates and ticket derivation. The summaries above are architecture context, not a separate implementation sequence.
