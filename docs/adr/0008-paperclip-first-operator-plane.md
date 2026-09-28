# ADR 0008: Evaluate Paperclip before building an ODAF work control plane

- Status: Accepted direction (integration unverified; final adoption gated by a pilot)
- Date: 2026-09-27

## Context

ODAF needs a Windows-local web interface for work intake, assignment, agent activity, evidence and operator decisions. The earlier plan assumed that ODAF would implement this entire surface. [Paperclip](https://github.com/paperclipai/paperclip) already provides a local web control plane with tasks, agent coordination, approvals and activity. Its documentation lists [Pi and Hermes adapters](https://docs.paperclip.ing/guides/org/agent-adapters/), [experimental SSH execution environments](https://docs.paperclip.ing/experimental/environments/) and a [remote HTTP adapter](https://docs.paperclip.ing/reference/adapters/http/). These are integration possibilities, not proof that the ODAF fleet works with Paperclip. No operator-owned Paperclip fork has been verified; record its URL only if one is confirmed.

## Decision

Take a **Paperclip-first** path for the work/agent operator plane. Pause custom ODAF dashboard, task-board, dispatcher, queue and approval-store implementation where Paperclip might meet the same need. Do not yet declare Paperclip the production authority or close existing delivery tickets. The next decision gate is a one-VM fit/gap pilot, not a five-VM migration.

Keep ODAF responsible for the Windows Hyper-V fleet: golden image, safe cloning and personalization, declared one-primary-agent-per-VM placement, selected-subset start/stop, guest identity and readiness. The operator remains the final authority for consequential actions. Inside a guest, the primary agent may use local sub-agents; these are not extra fleet placements. The pilot must establish one owner for assignment and run state so Paperclip and ODAF cannot dispatch the same work independently. Existing ADR 0007 safety and evidence rules remain requirements to test against Paperclip, not presumed Paperclip features.

The pilot uses a Paperclip instance reachable through the Windows-local browser and one existing Ubuntu/Pi guest. It verifies: local installation and restart on the intended host; authenticated connection to the guest; direct, bounded assignment; observable VM versus agent/run state; result/evidence and human approval; failed, interrupted and repeated dispatch without duplicate work; and whether the selected VM can remain stopped until its approved work requires it. Compare SSH environment, HTTP adapter or a minimal host bridge only as needed. Do not expose the dashboard to the public network or store provider credentials in the VM image or repository. Record actual behavior, incompatibilities and operator verdict before changing the full delivery plan.

## Consequences

- Issue #29 becomes the Paperclip fit/gap decision gate; W01's custom-dashboard implementation path is paused pending that verdict.
- If the pilot passes, amend ADRs 0001/0006/0007, the unified plan and affected tickets to give Paperclip explicit ownership of work state, with ODAF providing a thin Hyper-V/guest boundary. Avoid a second ODAF task database or router.
- If the pilot fails a necessary requirement, document the failed seam and choose the smallest alternative; the earlier host-local dashboard plan remains available rather than being silently discarded.
- Paperclip's organization, budget and automation features are optional unless a demonstrated ODAF requirement needs them. No VM lifecycle or Hyper-V support is assumed from Paperclip's agent-adapter documentation.
