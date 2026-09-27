# ODAF unified implementation vision and delivery plan

Status: implementation plan; the described software is not yet implemented or verified. This is the single delivery roadmap for the project. [Issue #1](https://github.com/BillBalint-SM/project-O.D.A.F./issues/1) supplies the original runtime, image and validation requirements; [Issue #2](https://github.com/BillBalint-SM/project-O.D.A.F./issues/2) adds the accepted operator dashboard, assignment and evidence workflow. The [decision index](decision-status.md) and its accepted ADRs govern wherever an earlier suggestion conflicts with a later decision. Research documents describe alternatives, not automatically approved features.

Tickets must be derived from the work packages in this document. They are delivery slices, not a second product plan. A work package is complete only when its behavior is observable through the shared command/status contract, the host-local dashboard where operator-facing, local checks, and an appropriate real-environment demonstration. Do not mark a design idea as implemented merely because it is documented.

## 1. Vision and release outcome

ODAF makes one Windows PC the operator's control station for five dedicated AI-development specialists in Ubuntu Hyper-V guests. The operator can build or reconcile the fleet from one known image; start, stop and inspect all five or any selected subset with one action; prepare and assign bounded work; observe execution and evidence; and accept, redirect or block the result. Another developer should be able to reproduce this on a compatible host without receiving the original operator's disks, addresses or credentials.

The five VM placements and the five agent definitions are related but different. A VM supplies a guest environment; a profile gives an agent its role, tools, skills and permissions; a harness runs the model/tool loop; a work item is assigned to one accountable agent. A Linux login named `agent1`, `agent` or anything else is not the human operator or an agent identity by itself. The operator can use the Windows dashboard without opening five guest consoles or relying on Herdr/Orca as a fleet dashboard.

The first supported environment is one trusted operator, one Windows Hyper-V host, five Ubuntu Server guests, and local-first validation. The default per-placement resource contract is Generation 2, two vCPUs, four GiB fixed RAM and a 70-GiB virtual disk. A requested start is admitted only when the host can support the selected set; this default is not a guarantee that five heavy browser or container workloads fit at once. The release does not promise cloud scheduling, multi-host high availability, multi-tenant access, GPU inference, or automatic deployment.

Success is demonstrated in increasing scope: a safe one-placement command/status/dashboard path; a verified common image and independently identified clone; a working Pi/Herdr agent pilot; any selected subset; then five distinct profiles with bounded assignment, independent review and a real five-VM smoke test. Each stage must be usable and locally testable before the next relies on it.

## 2. One architecture, clear state authority

| Layer | Owns | Must report, not infer |
|---|---|---|
| Operator on Windows | Work intent, approvals and final verdict | The decision and its scope |
| ODAF host control plane | Fleet registry, plans, Hyper-V lifecycle, assignment and durable coordination state | Desired versus observed state; selected targets; failures and uncertainty |
| Host-local dashboard and command interface | Two views of the same ODAF operations | The same status and approval boundaries, with observation time |
| Hyper-V | VM registration, power, hardware and checkpoint state | VM facts, not agent readiness |
| Ubuntu guest and authenticated SSH boundary | Guest/network reachability and local workspace | Reachability and guest identity, not task success |
| Guest supervisor | Profile preparation and readiness | Profile version, capabilities, health and errors |
| Herdr in the first interactive pilot | Pane and Pi process lifecycle | Session/process presence, reconnect and exit state |
| Selected harness, initially Pi for implementers | Inner model, tool and context loop | Run outputs, tool results and evidence |
| Optional Orca integration | A separately approved worktree/run mode | Only the state it actually owns; never a second owner for one process |

The first host implementation uses Windows PowerShell 5.1 and native Hyper-V cmdlets. The guest supervisor is intended as a small systemd-managed Python service, subject to the thin interface and pilot evidence; it must not compete with Herdr for ownership of the first interactive Pi process. ODAF must not reimplement the Pi, Hermes or other harness inner loop. Hermes Agent is a candidate harness, not a fleet authority. The first release must work without Orca.

One versioned declarative fleet registry maps agent definition, VM placement, guest identity, Linux login and profile explicitly. The example configuration is not a source of live values. The host exposes validate, plan, reconcile, start, stop and status operations, plus bounded work-item operations. Machine-readable output is the primary verification seam. It separates VM, SSH, guest, supervisor, Herdr, harness, assignment, run, validation and operator-verdict state. Every observation is timestamped; stale or unavailable information remains unknown rather than becoming “idle,” “ready,” or “complete.” Operations need correlation identifiers and outcomes that distinguish planned, active, completed, failed and uncertain work. Retrying a request must reconcile observations before taking another action.

The host-local dashboard is required in v1 but its UI framework and packaging are still open. Its first slice chooses the smallest maintainable approach with an operator-reviewed rationale. The dashboard never keeps a competing fleet truth, bypasses a CLI guard, or stops a guest when its window closes. Reopening it refreshes observed state. A stop request affecting active work requires an impact warning and explicit confirmation.

## 3. Image, development environment and identity

The common image is a versioned product artifact, not an arbitrary copy of a running guest. A rebuild records its source, package/tool versions, capability checks and compatibility with personalization and profile versions. Capture or export must be validated before it becomes the clone source. A Hyper-V checkpoint is a recovery point, not by itself a complete independent image or backup; do not assume a checkpoint-linked disk can safely be copied as a standalone clone. Microsoft documents [checkpoint behavior](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/checkpoints) and [copy import with a new VM ID](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/deploy/export-and-import-virtual-machines). An implementation ticket must choose and test the actual capture/clone path against the local host, including rollback without overwriting an existing VM.

The shared environment should make serious terminal-based AI development practical without five divergent operating-system images:

| Capability | Common-image intent | Boundary |
|---|---|---|
| Shell and repository work | Bash, tmux, SSH, Git, Git LFS, GitHub CLI, curl/wget, rsync, jq/yq, ripgrep/fd/fzf and diagnostic utilities | Verify actual installed versions and commands; no personal Git auth in the image |
| JS/TS | Supported Node.js LTS, npm, pnpm and TypeScript project tooling | Project-local versions come from an explicit version policy |
| Python | Supported Python with uv, ruff, pytest and pyright | Avoid relying on accidental system-package state |
| Native builds | C/C++ compilers, build-essential, CMake, Ninja, pkg-config, ccache and support for Go/Rust projects where needed | Report resource requirements instead of assuming every build fits in four GiB |
| AI coding CLIs | Codex CLI and Pi as known starting points; Claude Code, Gemini CLI, Aider and Hermes as supported/candidate capabilities to validate | No provider credentials in the image; one primary harness per agent instance |
| Browser | Playwright and Chromium baseline, usable by a profile that needs it | Other engines, browser MCP and concurrent graphical workloads are profile choices |
| Containers/infrastructure | Docker CLI/Buildx/Compose and relevant kubectl, Helm, k9s, Terraform/OpenTofu, Ansible capabilities where useful | Docker Engine, clusters and local model runtimes are opt-in; their memory/security cost is explicit |

The exact tool versions, version manager and compatibility matrix are implementation decisions to record before freezing an image; the list above is a capability target, not permission to install every candidate blindly. A capability report must state what is installed, usable and optional/disabled. Normal test runs must not require provider credentials or public internet; image download/build checks may be separately marked as network-dependent.

The image has no personal hostname, static live address, provider token, SSH private key or operator-specific profile. Provision a clone in a non-destructive, initially stopped state; then perform repeatable personalization before treating it as ready: assign unique VM and guest identifiers, hostname, machine identity, SSH host keys, network settings, declared Linux login/workspace and profile. The same login name may appear inside isolated guests, but it is never used as an agent ID. Compare declared and observed identity before dispatch. Provider authentication is performed inside the chosen guest harness using controlled runtime secret delivery; ODAF may report usability but must not display or persist the credential itself. Rotation and failure handling must be testable without exposing secret values.

Checkpoints or other recoverable snapshots precede risky image/fleet changes, with the exact rollback action requiring operator authorization. Partial provisioning is recorded and reconciled; conflicts with unrelated VMs or disks block rather than overwrite. Never reset the whole fleet because one placement fails.

## 4. Five specialist profiles and the work loop

| Role | Accepted default | What remains to decide at its pilot |
|---|---|---|
| Research/specification | Owns evidence-gathering and specification output | Primary harness, versioned skill/tool/permission profile and VM mapping |
| Architecture/planning | Owns design options, constraints and execution plans | Primary harness, versioned skill/tool/permission profile and VM mapping |
| Implementation A | Pi; TDD-first default | Exact tested Pi/Herdr integration and profile version |
| Implementation B | Pi; behavior/acceptance-first (BDD) default | Exact tested Pi/Herdr integration and profile version |
| Test/review/integration | Independent checker of maker evidence | Primary harness, review permissions, profile version and VM mapping |

TDD-first and BDD-first are working defaults, not exclusive skills. Luna, Sol, Astra or any other model family and reasoning effort belong to the work item and available provider capabilities, not permanent VM or agent identity. The three non-implementation harness selections happen after the first pilot, with an operator-reviewed choice; a role name alone is not a runnable profile. Each guest has its own repository clone. Task-specific worktrees may be used for isolation, but no shared writable host checkout is the hidden source of truth.

The operator may prepare an idea, document or spec alone or with an agent. A dispatchable work item states its primary deliverable, acceptance criteria, permitted scope/actions, expected evidence, agent-count limit, elapsed-time limit and provider-cost limit. Until measured defaults exist, the operator supplies these limits. Exceeding one pauses and asks; the system does not silently switch provider/model, increase cost or recruit another agent.

The operator can assign directly or request an ODAF recommendation. A recommendation first filters by capability, permission, project access, capacity and known VM startability; then uses a simple deterministic ranking with visible reasons, or abstains. Assignment requires approval. Only the selected stopped VM may start after approval; actual guest/harness readiness is checked before dispatch. A busy or unavailable specialist queues with a visible reason. One accountable lead owns each work item, and a VM handles at most one active item. Reassignment needs operator approval. Automatic routing remains observation-only shadow mode until a later explicit decision based on eligibility, abstention, overrides, quality, rework, time and cost.

V1 permits solo work, sequential handoff and parallel read-only research/review. A lead can delegate bounded, logged, non-writing investigation within the already approved scope and budget; delegation cannot add authority or shift accountability. Parallel code writing needs a separate isolated pilot and integration policy. Local scoped edits, tests and commits may occur under an approved work item. Push, PR creation, merge, deployment and destructive actions each require separate operator approval; technical tool access is not approval.

The maker returns artifacts, diff summary, test results, logs where relevant, gaps and risks against the work item's criteria. A separate test/review agent independently evaluates them. ODAF records report, validation and operator decision as distinct states. Only the operator accepts, blocks or redirects and owns consequential publication. An agent's self-report does not become a verdict automatically.

## 5. Safety, recovery and validation

Every boundary fails closed when identity, authorization or state freshness is uncertain. The host reads Hyper-V and guest state before retrying a failed or ambiguous operation; it does not infer that a timed-out dispatch did or did not execute. Cancellation, reconnect, restart, stop and crash recovery are pilot behaviors to measure and then specify. Durable coordination data must survive closing the dashboard and restarting the host command process, without storing provider tokens or private keys. The implementation chooses a small local state store only when the first real workflow needs one; its format is not predetermined here.

The primary test seam is the externally visible Windows command/status contract. Local checks cover PowerShell syntax/behavior (Pester and PSScriptAnalyzer where applicable), guest shell behavior (ShellCheck and disposable Ubuntu checks where practical), JSON/configuration and documentation consistency. Deterministic fake Hyper-V/SSH/guest responses test validation, idempotency, failure classification, selected subsets, stale status, approval guards and retries without five real VMs or provider credentials. Dashboard tests prove it invokes that same guarded contract. Real lab smoke testing advances from one VM to two simultaneously personalized guests and then the selected-subset/five-VM flow. No ordinary local change should require hosted GitHub Actions minutes. A manual or release-scoped workflow may be added only when useful; hosted runners are not assumed to offer Hyper-V integration.

Each delivery ticket has these common acceptance rules:

1. Show one complete observable user/developer path, not only a library, schema, UI mock or future scaffold. Include the dashboard path whenever the behavior is operator-facing.
2. Exercise the public contract locally with a failure case and prove no unrelated VM, disk, credential or work item was changed.
3. Report actual tests and real-VM checks performed; do not claim unrun hardware validation.
4. Update capability/version/decision documentation only for real behavior. Preserve public examples free of personal addresses, hostnames and secrets.
5. Record a newly resolved open decision in the appropriate ADR/decision index before relying on it; do not silently turn a candidate into a product promise.

## 6. Delivery map — the source of implementation tickets

IDs below are stable work-package references; GitHub issue numbers are not. A blocker is a genuine prerequisite, not merely an earlier line in this table. A ticket may be worked when all of its blockers are complete. Each row must be expanded into one issue with a demonstration and failure-oriented acceptance criteria; avoid layer-only tickets.

| ID | Demoable outcome | Blocked by |
|---|---|---|
| W01 | Inspect one existing placement through machine-readable status and a minimal host-local dashboard; unknown/stale layers remain explicit. Select and record the dashboard approach. | None |
| W02 | Run one local validation command for the first host/dashboard slice, with repeatable PowerShell/config/doc checks and no GitHub Actions dependency. | W01 |
| W03 | Validate one placement and show a no-side-effect plan with identity, capacity, path and network conflicts visible in both operator surfaces. | W01, W02 |
| W04 | Build/verify one candidate guest with common shell, repository, JS/TS, Python and native-build capabilities, reporting actual versions. | W01, W02 |
| W05 | Use Pi/Codex and a Playwright/Chromium browser check on the candidate without embedded provider credentials; classify other AI/infra tools as verified or optional. | W04 |
| W06 | Capture a versioned, independently usable golden image with manifest, compatibility check and tested recovery path before replacing a candidate. | W05 |
| W07 | Start/stop one already registered VM with plan/impact guard, idempotent status and an active-work stop warning. | W03 |
| W08 | Create one powered-off clone from the verified image, preserving source and blocking conflicting VM/disk targets. | W03, W06 |
| W09 | Personalize and boot the clone with unique guest/network/SSH identity, authenticated host access and an observed identity check. | W07, W08 |
| W10 | Activate and check guest-side provider credentials at runtime without copying secrets into the image, registry or normal logs; demonstrate failed and rotated credentials. | W09 |
| W11 | Start/stop any selected subset of one to five declared placements with admission checks and idempotence; report missing profile/harness readiness as unready until W15–W18 supply those profiles. | W09 |
| W12 | Activate Implementation A's Pi/TDD-first profile through the guest supervisor and show distinct supervisor, Herdr and harness readiness. | W04, W07, W10 |
| W13 | Run one bounded interactive Pi work item under Herdr process ownership, with per-item model/reasoning choice and evidence returned to ODAF. | W12 |
| W14 | Reconnect, stop, time out and recover an interrupted Pi pilot without double-dispatch or an invented success state. | W13 |
| W15 | Activate Implementation B's Pi/BDD-first profile on its own placement and demonstrate that both implementation defaults remain configurable per work item. | W09, W13 |
| W16 | Choose, approve, activate and demonstrate the research/specification harness and versioned profile on its own placement. | W09, W13 |
| W17 | Choose, approve, activate and demonstrate the architecture/planning harness and versioned profile on its own placement. | W09, W13 |
| W18 | Choose, approve, activate and demonstrate the independent test/review/integration harness and versioned profile. | W09, W13 |
| W19 | Directly approve and dispatch one bounded work item to one ready lead, enforcing scope, budget, action authorization and one active item per VM. | W13 |
| W20 | Queue unavailable work, persist assignment/run state across restart and require approval for rerouting; show uncertainty without duplicating execution. | W11, W14, W19 |
| W21 | Have the independent reviewer check maker evidence and present an operator accept/block/redirect verdict without automatic shipping. | W18, W19 |
| W22 | Recommend or abstain using visible eligibility/ranking, obtain approval before starting only the chosen VM, and log shadow routing outcomes. | W20 |
| W23 | Delegate a bounded, logged read-only investigation to another specialist in an isolated workspace without changing the accountable lead. | W16, W19 |
| W24 | Demonstrate one-, selected-subset- and five-VM flows in the real lab and publish a reproducible adopter guide with known limits and unverified optional features. | W11, W14, W15, W16, W17, W18, W20, W21, W22, W23 |

The independent early frontier is W01. After it, W02 and subsequent slices open only according to their actual blockers. W04–W06 deliberately mature the image before W08 clones it. W07 validates an existing VM without waiting for image work. W16–W18 may proceed in parallel after the first Pi pilot. W24 is a release demonstration, not a place to implement missing features.

## 7. Open decisions and explicit release gates

The following are not pre-approved implementation facts: dashboard framework/packaging; exact versions and version manager; primary harness and permissions for the three non-implementer roles; final role-to-VM mapping; Herdr reconnect/cancel/timeout/uncertain-dispatch semantics; durable run-state storage; numerical default budgets and auto-routing thresholds; parallel-writing integration; optional Orca process ownership. Their triggering work packages gather evidence, propose the smallest maintainable choice and record operator/ADR approval before making that choice a dependency for later work. If a gate cannot be resolved, the ticket reports the blocker instead of inventing a default.

The first usable release need not contain an auto-router, Orca integration, local models, always-on containers or a cloud service. It does need the host-local dashboard, safe subset fleet control, five actually runnable specialist profiles, at least the accepted manual/direct work path, independent review and operator verdict. Shadow recommendations are useful as a later v1 slice but never grant autonomous assignment. The final W24 smoke result distinguishes complete, partial and unverified capabilities rather than hiding deferred items.

## 8. Source reconciliation

| Source requirement | Unified treatment |
|---|---|
| Issue #1: explicit CLI/language/native/browser/AI candidates, version reporting and local checks | Preserved as image capability targets and W02, W04–W06; optional infra CLIs are validated where useful, heavy services remain opt-in. |
| Issue #1: checkpoint, safe clone, separate Linux workspace, authenticated SSH and runtime secrets | Preserved in the image/identity/security model and W06, W08–W10. |
| Issue #1: guest supervisor invokes the harness and owns all process behavior | Superseded for the first interactive Pi pilot: supervisor prepares/reports; Herdr owns pane/process; Pi owns the inner loop (ADR 0006). |
| Issue #1: GUI dashboard out of scope | Superseded: a host-local dashboard sharing the command/status contract is required (ADR 0001 and Issue #2). |
| Issue #1: repository is empty | Historical statement only; inspect and preserve the current documentation-and-contract foundation. |
| Issue #2: five roles, approved routing, bounded collaboration, maker/checker/operator evidence | Preserved in the profile and work-loop model and W15–W23 (ADR 0007). |
| Both issues: public reusability, secret safety, on-demand fleet and local-first testing | Preserved as acceptance rules across every work package and in W24. |

This document is the roadmap; the issues remain provenance and searchable discussion. If an accepted ADR changes, update this plan and affected tickets together. Avoid copying the entire plan into every ticket.
