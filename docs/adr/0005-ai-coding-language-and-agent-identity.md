# ADR 0005: Use dictionary-aligned AI-coding language and separate agent identity

- Status: Accepted
- Date: 2026-09-27

ODAF adopts Matt Pocock's AI Coding Dictionary as the external vocabulary for model, model provider, harness, environment, session, turn, tool, skill and handoff artifact. Within ODAF, agent definition describes a stable role, agent instance describes one activated runtime, and VM, Linux login and provider credential remain separate concepts. This avoids using agent as a vague synonym for a machine, user or CLI and keeps ODAF terminology compatible with the wider AI-coding community.
