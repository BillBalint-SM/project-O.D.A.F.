# ADR 0002: Use a common golden image with repeatable personalization

- Status: Accepted (design; implementation pending)
- Date: 2026-09-26

## Context

Five manually maintained VMs drift quickly. A fully unique image per agent duplicates maintenance and makes upgrades expensive. A single image containing every identity and credential would be unsafe.

## Decision

Build one versioned common Ubuntu AI-development image. After cloning, run an idempotent personalization step that assigns unique hostname, machine identity, SSH host keys, network identity and profile. Activate specialization through profile data and guest setup rather than through five divergent operating-system images.

## Consequences

- Common packages and security updates have one maintenance path.
- Clone operations must regenerate unique identity material.
- Profile activation must be observable and repeatable.
- Image version, personalization version and profile version are separate compatibility values.
