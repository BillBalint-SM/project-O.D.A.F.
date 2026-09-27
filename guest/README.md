# Guest runtime boundary

This area will contain the Ubuntu bootstrap, clone-personalization and guest supervisor implementation.

The guest contract is intentionally small:

- install or verify common image capabilities;
- set unique hostname, machine identity and SSH host keys after cloning;
- activate one agent profile;
- prepare an isolated workspace;
- prepare the selected harness and report its readiness; for the first interactive Pi pilot, Herdr owns its pane/process lifecycle;
- expose machine-readable health and capability information;
- restart safely without losing the declared identity.

The guest must not require Hyper-V permissions. Credentials arrive at runtime and are never built into the image.
