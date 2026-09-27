# Host control-plane boundary

This area will contain the small Windows PowerShell control plane once implementation begins.

The control plane must expose these behaviors:

- validate the registry and profile references;
- render a dry-run plan;
- create or reconcile the five VM instances;
- personalize clones without reusing unique identity material;
- start or stop all agents or a selected subset;
- report VM, SSH, supervisor and harness health;
- create checkpoints before risky mutations.

Keep Hyper-V cmdlets behind host-side helpers and keep guest operations behind an SSH boundary. The first tests should exercise the observable command surface with deterministic adapters rather than requiring a live fleet.
