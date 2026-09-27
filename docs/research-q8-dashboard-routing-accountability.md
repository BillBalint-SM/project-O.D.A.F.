# ODAF open design questions Q8.1, Q8.2, Q8.4

**Research date:** 2026-09-27
**Scope:** Dashboard architecture, work-item routing authority, and accountable execution model for five specialist Ubuntu VMs on one Windows Hyper-V host. Assumptions include remote SSH worktrees, scarce GitHub Actions minutes, local-first checks, and operator review.
**Source policy:** Primary sources only: official project documentation/source, official vendor documentation, and research papers. Product claims below describe documented capabilities, not an independent field test.

**Accepted Q8.1 clarification (2026-09-27):** The operator's own Windows PC is the dashboard machine. A unified, host-local ODAF dashboard is required in the first usable release, not conditional on an Orca/Herdr pilot. The original CLI-plus-links-only option below is superseded. The Q8.2/Q8.4 alternatives below are research background; the accepted v1 policy is in [ADR 0007](adr/0007-work-routing-and-evidence.md). [ADR 0001](adr/0001-control-plane-and-runtime.md)

**Q8.2/Q8.4 follow-up:** The expanded [routing and parallelism specification](q8-routing-and-parallelism-spec.md) is the current inventory of options, proposed contracts, and evaluation approach. The numerical pilot gates below are retained as historical hypotheses only; they are not adopted thresholds. In particular, agreement with the operator's route is not a substitute for accepted task outcomes.

## Recommendation

Build the **minimum unified ODAF dashboard on the Windows host** for VM/fleet readiness, subset control, assignment intent, progress and operator decisions. Back it with the same Windows command/status contract; avoid a second state store in the UI. Use Herdr for persistent remote SSH terminal sessions and pane/agent observation. Pilot the installed Windows Orca desktop against one Ubuntu guest as an **SSH worktree target** for task worktrees, run visibility, and diff review. Do not make either product authoritative for Hyper-V state or ODAF assignment records. Pick one execution/terminal owner per run and link to it from ODAF.

Start with **operator-selected assignment and a visible recommendation-only router**. A later policy may auto-select an agent for low-risk, reversible work after the decision criteria below pass. Keep consequential approval (publishing, merging, external actions, destructive fleet operations) with the operator. “Autonomous routing” should mean selecting a destination under an explicit policy; it must not implicitly mean autonomous approval or release.

Use **one accountable lead per work item**, selected by the operator or router. Permit bounded parallel specialists only for independent subtasks in separate workspaces; the lead integrates and owns the result, and the operator remains the final approver. Each guest has its own Git clone, so cross-VM integration uses commits/branches rather than one shared worktree. Do not use an unstructured swarm as the default.

This fits the existing ODAF contract: direct assignment and routed assignment are both contemplated; the control surface must record the selected target; each guest owns its clone/worktree; execution, validation, and operator review are separate states; and final consequential approval belongs to the operator. ([task-flow.md](task-flow.md), [agent-stack-and-authority.md](agent-stack-and-authority.md))

## Decision summary

| Question | Recommended default | What would change the decision |
|---|---|---|
| **Q8.1 Dashboard: Orca vs Herdr vs custom ODAF** | Accepted: one Windows-hosted ODAF dashboard for all five agents, backed by the shared control/status contract. Herdr supplies detailed terminal sessions; pilot Windows Orca + SSH guest worktrees for run/review. | The dashboard's framework, packaging and detailed screens remain open; a pilot may shape them but does not make the unified dashboard optional. Avoid duplicating terminal or diff editors. |
| **Q8.2 Manual vs autonomous routing** | Proposed: manual selection and visible recommendation first; policy-based auto-assignment remains an option. | Evaluate shadow-mode outcomes and risk before choosing any auto-assignment gate; the numeric figures below are historical hypotheses. Never auto-approve consequential actions. |
| **Q8.4 Lead vs parallel/swarm** | Proposed: one accountable lead and optional delegated specialists for independent slices; isolated clones/branches and deliberate integration. | Compare complete task outcomes, including integration and review, before selecting a concurrency policy; the numeric figures below are historical hypotheses. |

## Q8.1 — Dashboard architecture

### Documented capabilities and limits

**stablyai/orca.** Orca documents itself as a desktop/mobile/remote-runtime agent development environment. Its first-party repository lists parallel Git worktrees, terminals, agent support, review/diff annotation, remote SSH worktrees, and a CLI for creating/inspecting worktrees and driving terminals. [Orca repository](https://github.com/stablyai/orca), [Orca CLI overview](https://github.com/stablyai/orca/blob/main/docs/site/content/docs/cli/overview.mdx). Its CLI exposes JSON commands for worktree status/create, terminal read/send/wait, and terminal creation in a worktree. [Orca CLI guide](https://github.com/stablyai/orca/blob/main/skill-guides/orca-cli.md)

Orca documents **SSH worktrees** driven from the desktop application, with the Git worktree and agent running on the remote host. This is the appropriate first pilot path for the already installed Windows Orca client and the Ubuntu guests. [Orca SSH worktrees](https://www.onorca.dev/docs/ssh), [remote-worktree recipe](https://www.onorca.dev/docs/recipes/remote-worktrees). Separately, Orca documents a **headless Linux runtime** (`orca serve`) for Ubuntu 20.04/22.04/24.04 and Debian, with Xvfb and Electron library prerequisites. **Ubuntu 26.04.1, the ODAF guest release, is not listed**; do not assume `orca serve` is supported there without a compatibility test or maintainer confirmation. [Headless Linux Server](https://github.com/stablyai/orca/blob/main/docs/reference/headless-linux-server.md). Orca's SSH execution guidance states that the execution host owns process state and that lost contact means “unverifiable,” not proof the process exited. [SSH execution boundary](https://github.com/stablyai/orca/blob/main/docs/reference/ssh-execution-boundary.md)

**Herdr.** Herdr’s own docs describe it as a persistent terminal/session and agent-observation layer. It can connect from a local client to remote Linux machines over ordinary SSH; each remote machine retains its own server, sessions, and processes. [Connecting machines](https://herdr.dev/docs/connecting-machines/). The automation API/CLI can create workspaces/panes, start recognized agents in existing panes, prompt them, read output, and wait on lifecycle states. It distinguishes pane existence from recognized agent state and documents `unknown` where status cannot be classified confidently. [Agent automation](https://herdr.dev/docs/agent-automation/), [CLI reference](https://herdr.dev/docs/cli-reference/)

Herdr does provide Git worktree creation and workspace-oriented visibility, but its documented responsibility is terminal/workspace/agent control; it does not claim to manage Hyper-V, ODAF fleet registry, or operator assignment policy. Its remote CLI addresses one machine at a time and is not a combined fleet router. [CLI reference](https://herdr.dev/docs/cli-reference/)

**Custom ODAF interface.** The project defines a Windows command surface with `validate`, `plan`, `start`, `stop`, `status`, and `reconcile`, machine-readable status, dry-run planning, and distinct VM/SSH/guest/Herdr/harness readiness. At the time of the initial research, the graphical surface was open; the subsequent operator clarification made a host-local dashboard required while retaining that command/status seam. ([README.md](../README.md), [ADR 0001](adr/0001-control-plane-and-runtime.md))

### Evidence-based comparison

| Capability relevant to ODAF | Orca | Herdr | Custom ODAF |
|---|---|---|---|
| Hyper-V lifecycle/fleet registry | No documented ODAF/Hyper-V authority; integrate rather than delegate it. [Orca repo](https://github.com/stablyai/orca) | No documented fleet/Hyper-V authority. [Herdr connecting machines](https://herdr.dev/docs/connecting-machines/) | Correct place for ODAF’s Windows/Hyper-V contract, but a visual dashboard is new implementation. |
| SSH Ubuntu access | Desktop-driven SSH worktrees are documented. Headless `orca serve` support for Ubuntu 26.04.1 is unverified. [Orca SSH docs](https://www.onorca.dev/docs/ssh), [headless server](https://github.com/stablyai/orca/blob/main/docs/reference/headless-linux-server.md) | SSH remote-machine sessions are a first-class documented capability. [Connecting machines](https://herdr.dev/docs/connecting-machines/) | Can show fleet and links/status; recreating terminal transport is unnecessary. |
| Worktree/run workflow | Strong documented fit: create/list worktrees, agent terminals, run state, diff review. [CLI overview](https://github.com/stablyai/orca/blob/main/docs/site/content/docs/cli/overview.mdx) | Can create worktrees and drive agent sessions, but emphasis is persistent terminal and pane control. [CLI reference](https://herdr.dev/docs/cli-reference/), [automation](https://herdr.dev/docs/agent-automation/) | Must define policy and integration contract; no need to implement a second worktree engine. |
| Unified view across five guests | Orca supports remote runtime/SSH use, but evidence reviewed does not establish a native ODAF five-VM fleet readiness view. Treat as a pilot question. | Multi-machine UI lists remote agents/workspaces, while CLI targeting is one machine at a time. It does not replace ODAF aggregate fleet status. [Connecting machines](https://herdr.dev/docs/connecting-machines/), [CLI reference](https://herdr.dev/docs/cli-reference/) | Can aggregate ODAF-defined state, but a GUI duplicates established terminal and run UIs. |
| Operator review | Agent diffs/annotations and worktree state are documented. [Orca repo](https://github.com/stablyai/orca) | Pane state and terminal interaction support supervision; it is not a merge/release approval system. [Automation](https://herdr.dev/docs/agent-automation/) | Can enforce ODAF validation and approval gates; should link to diffs rather than build an editor/reviewer. |
| Cost/maintenance | Existing product reduces custom run/worktree UI work; SSH remote worktrees still add setup, compatibility, and resource costs to measure. Headless runtime would add more and is not the v1 assumption. | Existing product reduces terminal/pane UI work; requires a Herdr server per guest plus remote connection setup. | Highest ongoing implementation and maintenance cost; only justified by unmet ODAF-specific aggregate status/policy workflows. |

### Inference for this deployment

1. **No candidate alone is the right authority for all state.** ODAF must remain the one place that answers whether a VM/profile is ready and which agent was assigned. Orca and Herdr observe different execution layers; network or process state disagreement must remain representable rather than collapsed to one green/red badge.
2. **Herdr is the lower-friction default terminal layer** for five SSH-accessible Ubuntu guests because remote SSH and persistent per-machine sessions are central documented functions. Orca is the stronger candidate for a work-item-centric worktree/run/review experience. These roles overlap, so a run must declare which product owns its terminal/process lifecycle; dual ownership would create ambiguous start/stop/retry semantics.
3. **A full custom agent platform is premature; a focused dashboard is required.** At five guests, the Windows interface must cover fleet overview, arbitrary subset start/stop, assignment, progress and operator review. It should read and act through the same CLI/JSON-compatible control contract, while linking to Herdr and Orca for detailed terminal and diff work instead of reimplementing them.
4. **GitHub Actions scarcity strengthens local-first validation, not dashboard choice.** GitHub’s docs state hosted-runner quotas vary by plan and self-hosted runner minutes are free; self-hosted execution still consumes the operator’s VM/host capacity and requires maintenance. ODAF’s existing local-first validation policy should remain primary; CI should be selective (manual/release or quick hosted checks), not the runtime control plane. [Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions), [self-hosted runners](https://docs.github.com/en/actions/concepts/runners/self-hosted-runners)

### Testable Q8.1 decision criteria

Pilot on one representative guest, then repeat with five guests before freezing the integration boundary:

- **Placement/compatibility:** Use the Windows Orca client with one guest as an SSH target. Start/stop the Herdr server through its documented guest-level lifecycle. Verify reconnection after SSH interruption without incorrectly marking a still-running task exited. Test guest-hosted headless `orca serve` separately only after Ubuntu 26.04 compatibility is established.
- **Resource fit:** Measure idle and active CPU/RAM on the guest and host for Herdr-only versus Windows Orca + remote SSH worktree + Herdr. Accept the combined setup only if peak use stays within the declared VM profile with adequate headroom and no material impact on the agent task. A 20% headroom target may be used as an initial ODAF pilot hypothesis, not a vendor claim.
- **Five-guest observability:** From the Windows operator surface, identify each VM’s Hyper-V state, SSH reachability, guest/supervisor readiness, active work item, agent/run status, and last evidence timestamp. A disconnected UI must not mutate execution state.
- **Authority:** Start, stop, retry, and task assignment have one named owner in every supported path. ODAF task IDs map deterministically to Orca worktree/run IDs and Herdr machine/workspace/pane IDs. Duplicate submissions are idempotent or rejected clearly.
- **Review:** A human can inspect the exact task brief, selected profile/agent, diff, local validation summary, and approval status before publication. Orca/Herdr statuses must link back to the ODAF task record.
- **Dashboard gate:** The first usable release must show all five declared agents, distinguish VM/SSH/guest/harness/task states and stale observations, start or stop any selected subset with one action, record the chosen assignment, and expose progress, validation and operator approval. Closing and reopening the UI must preserve supervised work and refresh displayed state. The Orca/Herdr pilot determines integration links and process ownership, not whether this surface exists.

## Q8.2 — Manual or human-approved routing vs autonomous routing

### Evidence

ODAF already accepts both direct operator assignment and control-plane routing, and requires the selected target to be inspectable. This is compatible with a staged policy: manual assignment first, recommendations second, narrowly scoped automatic assignment later. ([task-flow.md](task-flow.md))

Herdr documents a useful human-control precedent: remote setup that may install/replace software requires interactive approval, defaults to No for replacing a running server, and background connections do not answer prompts or install/update/restart servers. [Connecting machines](https://herdr.dev/docs/connecting-machines/). OpenAI’s Agents SDK documents an approval pause: a sensitive tool call waits for a human approve/reject decision before the run resumes. This is a design precedent, not an assertion that ODAF uses that SDK. [Human-in-the-loop guide](https://openai.github.io/openai-agents-python/human_in_the_loop/)

Routing is distinct from authorization. A system can automatically select the most suitable specialist while still requiring human approval for tool use, external side effects, merge, or release. The OpenAI multi-agent documentation recommends clearly scoped delegated tasks and expected outputs; this is relevant to routing contracts but does not establish that any particular routing accuracy is safe for ODAF. [Multi-agent guide](https://developers.openai.com/api/docs/guides/agents-api/multi-agent)

### Recommendation and inference

Use three explicit modes, persisted per work item:

1. **Direct:** Operator names an agent. No router guess is involved.
2. **Recommend:** ODAF proposes one agent (or asks for clarification), shows a short rationale based on declared capabilities, resource needs, and current availability, and waits for operator confirmation.
3. **Policy auto-assign:** ODAF assigns only within a declared low-risk class; it records policy/version, candidate set, rationale/features, confidence/abstention, and override path. Work remains subject to normal validation and review.

For the first operational release, implement Direct and Recommend. They are simpler to audit and provide labeled outcomes that can evaluate the future router. Do not train/use routing from hidden prompts or opaque agent preference. Route from explicit profile capability metadata plus operator-provided work-item labels; when the task is ambiguous or spans specialties, ask or recommend the accountable lead.

### Testable Q8.2 promotion criteria

**Historical proposal, superseded by the evaluation approach in the [Q8.2/Q8.4 follow-up](q8-routing-and-parallelism-spec.md#testing-decisions).** The figures below were illustrative, not calibrated to ODAF data.

Run recommendation-only mode for at least **50 representative work items** across the five specialties (proposed minimum, not a statistically universal guarantee). Record the operator’s intended choice, router proposal, overrides, task outcome, validation/rework, and assignment latency.

Promote only if all are met:

- At least **90% top-choice agreement** with the operator’s chosen specialist on tasks the router does not abstain from.
- **100% correct abstention/escalation** for missing required capability, unavailable agent, conflicting constraints, and tasks tagged high-risk or cross-specialty; test these cases explicitly.
- At least **25% reduction** in median time from ready work item to assignment, without higher failure/rework rate than manual baseline.
- Every decision is inspectable and replayable from stored policy/version and input fields; operator override takes one action and does not lose the original recommendation.
- No automated routing bypasses task-level permission boundaries, validation, or operator release approval. Sensitive/destructive tasks remain approval-gated regardless of route mode.

If the sample is too small or the result is mixed, retain recommendation mode. A wrong route wastes guest capacity and operator time; at five specialists, assignment itself is a low-frequency decision, so expected time savings may not justify model calls or a complex classifier. That is an ODAF-specific inference to verify in the pilot.

## Q8.4 — One accountable lead vs parallel/swarm assignment

### Evidence

Anthropic’s first-party engineering account describes a lead agent planning research and launching parallel subagents for independent directions; the lead then compresses and synthesizes their findings. The reported 90.2% improvement is for Anthropic’s internal research evaluation and a breadth-first research use case; it is not a general software-development guarantee. [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)

The 2026 paper *Effective Strategies for Asynchronous Software Engineering Agents* identifies coordination, dependencies, concurrent edit interference, and integrating partial work as key difficulties. Its CAID design uses centralized, dependency-aware delegation, isolated workspaces, and structured integration with executable verification. The authors report gains over single-agent baselines on PaperBench and Commit0; these are paper-specific results and not a direct prediction for ODAF’s five-VM workload. [Geng & Neubig, arXiv:2603.21489](https://arxiv.org/abs/2603.21489)

Orca’s documented parallel-worktree flow creates isolated worktrees for parallel attempts, while its CLI makes those worktrees and agent terminals inspectable. [Orca repository](https://github.com/stablyai/orca), [Orca CLI guide](https://github.com/stablyai/orca/blob/main/skill-guides/orca-cli.md). Git worktrees provide separate working directories attached to one repository, which supports isolation but does not solve semantic integration or ownership by itself. [Git worktree documentation](https://git-scm.com/docs/git-worktree)

### Recommendation and inference

Make **one lead accountable for every work item**, independent of how many agents contribute. The lead owns scope, dependencies, integration, validation evidence, and the final handoff. The operator owns final acceptance/publication. Specialists can work in parallel only when the lead can define non-overlapping deliverables and separate worktrees; shared-file or tightly coupled work defaults to sequential delegation.

For ODAF, “swarm” should not mean every available VM receives the same prompt or edits the same checkout. That duplicates provider usage, consumes scarce guest resources, creates merge/review work, and obscures ownership. Use fan-out for research, independent reviews, test plans, or alternative implementations when the outputs are explicitly comparable; cap concurrency to available resource headroom and keep GitHub Actions out of the inner loop unless a task specifically needs it.

### Testable Q8.4 criteria

**Historical proposal, superseded by the evaluation approach in the [Q8.2/Q8.4 follow-up](q8-routing-and-parallelism-spec.md#testing-decisions).** The figures below were illustrative, not calibrated to ODAF data.

For the initial pilot, compare single-lead execution with lead-plus-two-specialist execution on **10 matched, decomposable tasks** (proposed small operational sample; results guide policy, not broad scientific claims). Capture wall-clock completion, provider/token spend where available, guest peak RAM/CPU, merge conflicts, lead integration time, validation failures, operator review time, and reopened/reworked tasks.

Allow default parallel delegation only if it delivers at least **20% median end-to-end time reduction**, with no increase in failed validation/rework, no more than **10% increase in operator review time**, and no resource-profile breach. Otherwise allow parallelism only by explicit operator choice. One lead and one task record remain mandatory at every concurrency level.

## Historical decision proposals (accepted v1 policy in ADR 0007)

> ODAF’s Windows control plane owns fleet state, assignment records and their unified host-local dashboard. The dashboard and command interface share one control/status contract. Herdr supplies persistent SSH terminal access, and a limited Windows Orca + Ubuntu SSH-worktree pilot may supply task worktrees and diff review. Guest-hosted headless Orca is deferred until Ubuntu 26.04 compatibility is verified. Each run declares one execution/terminal owner. Proposed for further decision: assignment begins as direct or operator-approved recommendation, with later measured low-risk auto-assignment; each work item has one accountable lead and bounded parallel specialists. The operator retains consequential approval and publication authority.

## Dated source ledger

Accessed **2026-09-27**. Live documentation and repository `main`/`master` links can change; pin commit hashes/release versions when turning pilot outcomes into implementation requirements.

| Source | Type and why it is primary | Date/status noted |
|---|---|---|
| [ODAF task flow](task-flow.md) | Project-owned contract for assignment, run, validation, and review | Repository state read 2026-09-27 |
| [ODAF agent stack and authority](agent-stack-and-authority.md) | Project-owned authority boundaries for ODAF, Herdr, Orca, harness, operator | Repository state read 2026-09-27 |
| [ODAF README](../README.md) | Project-owned deployment and status constraints | Repository state read 2026-09-27 |
| [stablyai/orca repository](https://github.com/stablyai/orca) | Product owner’s repository/feature statement | Live `main`, accessed 2026-09-27 |
| [Orca CLI overview](https://github.com/stablyai/orca/blob/main/docs/site/content/docs/cli/overview.mdx) | First-party CLI contract | Live `main`, accessed 2026-09-27 |
| [Orca CLI guide](https://github.com/stablyai/orca/blob/main/skill-guides/orca-cli.md) | First-party command/use guidance | Live `main`, accessed 2026-09-27 |
| [Orca SSH worktrees](https://www.onorca.dev/docs/ssh) | First-party desktop-to-remote worktree instructions | Live docs, accessed 2026-09-27 |
| [Orca remote-worktree recipe](https://www.onorca.dev/docs/recipes/remote-worktrees) | First-party remote worktree workflow | Live docs, accessed 2026-09-27 |
| [Orca headless Linux server](https://github.com/stablyai/orca/blob/main/docs/reference/headless-linux-server.md) | First-party deployment instructions and readiness protocol | Live `main`, accessed 2026-09-27 |
| [Orca SSH execution boundary](https://github.com/stablyai/orca/blob/main/docs/reference/ssh-execution-boundary.md) | First-party statement of remote execution ownership and uncertainty semantics | Live `main`, accessed 2026-09-27 |
| [Herdr connecting machines](https://herdr.dev/docs/connecting-machines/) | Maintainer documentation for SSH remote machine behavior | Live docs, accessed 2026-09-27 |
| [Herdr agent automation](https://herdr.dev/docs/agent-automation/) | Maintainer API/CLI behavior and agent state semantics | Live docs, accessed 2026-09-27 |
| [Herdr CLI reference](https://herdr.dev/docs/cli-reference/) | Maintainer CLI/remote-command contract | Live docs, accessed 2026-09-27 |
| [OpenAI Agents SDK human-in-the-loop](https://openai.github.io/openai-agents-python/human_in_the_loop/) | First-party implementation guidance for approval pauses | Live docs, accessed 2026-09-27 |
| [OpenAI multi-agent guide](https://developers.openai.com/api/docs/guides/agents-api/multi-agent) | First-party task delegation guidance | Live docs, accessed 2026-09-27 |
| [Anthropic multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) | First-party account of deployed lead/subagent architecture and evaluation context | Published 2025-06-13; accessed 2026-09-27 |
| [Geng & Neubig, *Effective Strategies for Asynchronous Software Engineering Agents*](https://arxiv.org/abs/2603.21489) | Research paper/preprint on centralized asynchronous isolated delegation | Submitted 2026-03-23; accessed 2026-09-27 |
| [Git worktree documentation](https://git-scm.com/docs/git-worktree) | Official Git documentation for worktree behavior | Live docs, accessed 2026-09-27 |
| [GitHub Actions billing and usage](https://docs.github.com/en/billing/concepts/product-billing/github-actions) | GitHub’s primary source for included hosted minutes and billing | Live docs, accessed 2026-09-27 |
| [GitHub self-hosted runners](https://docs.github.com/en/actions/concepts/runners/self-hosted-runners) | GitHub’s primary source for self-hosted runner operation | Live docs, accessed 2026-09-27 |

## Limits of this research

- No ODAF implementation exists yet, so the recommendations are architecture choices to validate, not measured ODAF results.
- Vendor feature statements are not independent reliability or security audits. This report does not claim Orca/Herdr feature parity, comparative uptime, or total resource consumption.
- Orca's headless-server guide does not list the ODAF guests' Ubuntu 26.04.1 release. Desktop-to-guest SSH worktrees are the recommended pilot; headless guest hosting remains a separate compatibility question.
- The proposed numeric thresholds (20% resource headroom, 50 routing samples, routing accuracy and time-saving targets, 10 parallel-task comparisons, and time/review cutoffs) are decision gates for the ODAF pilot, not published benchmark findings.
- GitHub Actions quota depends on account/repository plan. Confirm the actual account quota before setting CI cadence; preserve local validation regardless.
