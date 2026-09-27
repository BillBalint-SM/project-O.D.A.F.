# Domain context for ODAF agents

## Project

ODAF is a local fleet-control and runtime project for five specialized AI-development agents. The host is Windows with Hyper-V. The guests are Ubuntu Server VMs. The repository is public, so secrets and workstation-specific identity must stay local.

## Core goal

Make a reproducible agent fleet that can be created from one common image and started as a whole or as any selected subset. The operator's Windows PC is the unified dashboard and control plane; each guest owns its agent process and workspace.

## Main constraints

- The initial VM contract is two vCPUs, four GiB fixed RAM and 70 GiB disk.
- The host control plane is PowerShell 5.1 plus native Hyper-V cmdlets.
- The first usable release needs a host-local dashboard for viewing all five agents, controlling any subset and assigning work. It shares one control/status contract with the command interface; Herdr and optional Orca are specialist views, not the fleet dashboard.
- The guest common runtime includes Node.js/npm, Python, Git, Codex CLI and Pi, with a broader AI-development toolchain defined in the repository README and ADRs.
- Browser, CLI, container and infrastructure tools are baseline capabilities where useful; persistent daemons and local models remain profile opt-ins.
- GitHub Actions must not be the only validation path because hosted-runner budget is limited.

## Vocabulary

Use these terms precisely: control plane, guest supervisor, golden image, personalization, fleet registry, agent profile, harness, capability, provider credential, health and reconcile. Do not use agent1 as a synonym for the operator; it is a guest login or runtime identity only when the profile contract says so.

## Five specialist roles

The v1 role matrix is research/specification, architecture/planning, implementation A, implementation B, and test/review/integration. These are agent definitions, not Linux users or VM names. Skills, tools, permissions and resource needs belong in each agent profile.

Both implementation agents use Pi as their primary harness. A defaults to TDD-first; B defaults to acceptance/behavior-first (BDD). These are complementary working defaults, not exclusive permissions: either agent may use both methods when the work warrants it. Select model and reasoning strength per work item within the profile's allowed capabilities and budget; do not encode Luna, Sol or Astra as permanent agent identities. Select the primary harness for the other three roles after the first pilot; until then, their role definition does not imply a runnable configured agent.

## Success signal

The project is successful when the operator can use one Windows dashboard to inspect all five agents, start or stop any selected subset and assign work, while the same stable control-plane seam supports validation, planning and reconciliation. Each guest reports the correct profile and supervisor health; closing the dashboard does not stop supervised work.
