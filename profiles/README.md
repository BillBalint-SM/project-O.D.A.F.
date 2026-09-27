# Agent profiles

Profiles are the specialization layer above the common image. A profile should declare:

- stable agent identity and human purpose;
- primary AI CLI and harness;
- enabled skills and tools;
- browser, Docker, Kubernetes and model-runtime permissions;
- workspace policy;
- resource expectations;
- health checks and failure policy.

The five profile directories do not exist yet because no profile contract has been implemented. Add a profile when its first real harness or capability needs versioned files; do not copy a generic placeholder five times.
