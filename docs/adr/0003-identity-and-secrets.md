# ADR 0003: Keep identity layers and credentials separate

- Status: Accepted (design; implementation pending)
- Date: 2026-09-26

## Context

The VM name, hostname, Linux login, agent role and provider account are different concepts. Cloning can accidentally duplicate machine identity or expose credentials if they are treated as one value.

## Decision

Represent VM identity, guest identity, Linux login, agent profile and provider credential as separate fields. The fleet registry maps them explicitly. Provider credentials and private keys enter at runtime or through a controlled secret mechanism. They are never stored in the image, example configuration or public repository.

## Consequences

- Status output must show enough identity to diagnose a mismatch without printing secrets.
- Personalization is a required lifecycle stage, not an optional convenience.
- Public examples use placeholders and documentation-only values.
- Credential injection and rotation remain separate implementation work.
