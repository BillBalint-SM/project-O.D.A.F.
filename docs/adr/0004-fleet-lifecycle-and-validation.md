# ADR 0004: Make fleet operations declarative, idempotent and local-first

- Status: Accepted (design; implementation pending)
- Date: 2026-09-26

## Context

The operator needs one-command start/stop for all five VMs or any subset. GitHub Actions hosted-runner budget is limited, and GitHub-hosted runners are not assumed to provide Hyper-V integration.

## Decision

Use a declarative fleet registry and a small command surface for validate, plan, reconcile, start, stop and status. Selection is explicit and deterministic. Operations are idempotent and support dry-run output. The required Windows dashboard uses the same control/status contract; the command surface remains the primary test seam, not the only operator interface. Normal validation runs locally; GitHub Actions is manual or release-scoped. Real Hyper-V integration remains a trusted lab smoke test or later self-hosted runner job.

## Consequences

- The control-plane command surface is the primary test seam.
- Local developers can validate without provider credentials or hosted-runner minutes.
- A machine-readable status contract is required.
- Integration tests need a real lab environment outside ordinary unit/contract validation.
