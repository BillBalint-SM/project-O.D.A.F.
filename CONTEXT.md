# ODAF language

ODAF uses [Matt Pocock's AI Coding Dictionary](https://github.com/mattpocock/dictionary-of-ai-coding) for general AI-coding terms such as model, model provider, harness, environment, session, tool, skill, spec and handoff artifact. The terms below are specific to ODAF.

## Language

**Operator**:
The human who selects work, controls the fleet and reviews consequential results.
_Avoid_: agent1, Linux user

**Agent definition**:
A durable description of one specialist role and its assigned profile.
_Avoid_: VM, Linux user, running process

**Agent instance**:
One activated agent definition in a guest environment, using one primary harness at a time.
_Avoid_: VM, model, session

**Agent profile**:
The versioned specialization of an agent definition: instructions, skills, tools, permissions, harness choice and resource expectations.
_Avoid_: golden image, Linux account

**Work item**:
A bounded piece of work prepared by the operator alone or with an agent, then submitted for execution.
_Avoid_: VM, agent session

**Assignment**:
The decision that directs a work item to a particular agent definition or instance, either by the operator or through the control plane.
_Avoid_: starting a VM

**Fleet**:
The set of agent definitions and their declared guest placements, whether their VMs are running or stopped.
_Avoid_: collection of open terminal panes

**Fleet registry**:
The desired mapping between agent definitions, profiles and guest placements.
_Avoid_: observed VM state

**Control plane**:
The Windows-hosted ODAF component that plans and manages fleet placement, VM lifecycle, work assignment and their combined status.
_Avoid_: agent harness

**Operator dashboard**:
The unified interface on the operator's Windows PC for seeing and controlling the fleet, assigning work and reviewing results. It uses the control plane's contract rather than keeping separate state.
_Avoid_: guest desktop, Herdr pane, Orca worktree

**Guest**:
An Ubuntu VM that provides an environment in which an agent instance can run.
_Avoid_: agent

**Guest supervisor**:
The guest component that prepares an assigned profile and reports readiness for agent operation.
_Avoid_: model provider, fleet control plane

**Golden image**:
A versioned common guest image from which VM instances are created.
_Avoid_: personalized clone, agent profile

**Personalization**:
The process that gives a cloned guest its unique machine, network and SSH identity and activates its assigned profile.
_Avoid_: common image build

**Fleet health**:
The reported readiness of the VM, guest, supervisor and agent instance as distinct states.
_Avoid_: a single ambiguous running flag

**Reconcile**:
An idempotent operation that brings observed fleet state toward the declared fleet registry.
_Avoid_: recreate, reset
