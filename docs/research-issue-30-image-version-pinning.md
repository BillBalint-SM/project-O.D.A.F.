# Issue #30: Common image version and tool pinning

**Research date:** 2026-09-27  
**Question:** Which version-manager and pinning approach fits ODAF’s Ubuntu 26.04 common development image, and how should image-level pins differ from project pins?  
**Source policy:** Primary sources only. Product capabilities below are documented claims, not independent tests.

## Recommendation

Use **mise as the common guest tool-version manager**, with exact versions recorded for the image’s Node.js and supported CLI tools. Pin the mise release used to assemble the image and record its provenance. Use mise’s `mise.lock` where suitable to freeze resolved tool downloads, but do not treat a project lockfile as the golden-image manifest. For Python projects, retain `uv` for dependency resolution and `uv.lock`; use one chosen manager consistently for the Python interpreter rather than letting system Python, mise, and uv compete implicitly. ODAF already names mise or an equivalent tool manager and uv in its technical baseline. ([ODAF README](../README.md#working-technical-baseline), [Issue #30](https://github.com/BillBalint-SM/project-O.D.A.F./issues/30))

Keep three version records with distinct owners:

| Scope | Record and pin | Reason |
|---|---|---|
| Golden image | Versioned image manifest: Ubuntu release and image identity; apt package names and installed versions; mise release; exact common tool versions; source/checksum or provenance evidence; build and validation result. | A VM image contains OS packages, libraries, tools and configuration that project files cannot capture. The project’s ADR separates common image, personalization and profile versions. ([ADR 0002](adr/0002-layered-image-and-personalization.md)) |
| Project runtimes | Committed `mise.toml` plus `mise.lock` for project tools where available. Keep supported-version ranges in project metadata when the project supports a range; exact runtime selection is a separate decision. | mise documents that the project config declares tools and its lockfile resolves version requests to consistent pinned tool downloads. `locked` mode fails if the current platform lacks a pre-resolved URL. ([mise configuration](https://mise.jdx.dev/configuration.html), [mise lockfile settings](https://mise.jdx.dev/configuration/settings.html#lockfile)) |
| Project Python dependencies | Committed `pyproject.toml` and `uv.lock`, with `requires-python` expressing compatibility. | uv distinguishes a project’s supported Python range from its dependency lock and supports managed or system interpreters. ([uv project configuration](https://docs.astral.sh/uv/concepts/projects/config/), [uv Python versions](https://docs.astral.sh/uv/concepts/python-versions/), [uv lockfile](https://docs.astral.sh/uv/concepts/projects/sync/)) |

The image manifest should capture actual installed versions after an image build and validation; do not guess versions into it during this research. Rebuilds should be explicit image releases with a reviewed update cadence and compatibility checks, rather than unattended changes to a supposedly fixed image. Ubuntu 26.04 LTS is released and supported as an LTS; this confirms the OS target exists, not that every third-party tool or installer has been validated on it. ([Ubuntu 26.04 release notes](https://documentation.ubuntu.com/release-notes/26.04/), [ODAF implementation plan](implementation-plan.md))

## Candidate comparison

| Candidate | Fit | Limit |
|---|---|---|
| **mise** | One config can declare multiple tools and versions; official docs provide Linux installation and a lockfile with resolved versions/download URLs. Its official installer supports Linux x64 and arm64 binaries. ([mise install](https://mise.jdx.dev/installing-mise.html), [mise lockfile](https://mise.jdx.dev/configuration/settings.html#lockfile)) | Tool coverage comes from multiple backends/registries, so verify the selected backend and exact artifact for each required CLI. The manager lock records tool downloads; it does not capture apt packages or the complete VM image. ([mise backend overview](https://mise.jdx.dev/dev-tools/)) |
| **nvm** | Maintained Node version manager supports `.nvmrc`, and `nvm install` resolves and installs its requested Node version. ([nvm README](https://github.com/nvm-sh/nvm/blob/master/README.md)) | Node-only, so it leaves Python and the other common CLIs to separate mechanisms. `.nvmrc` may contain moving aliases such as `lts/*`; use a concrete version when exact reproducibility is required. |
| **uv alone** | Manages Python versions, supports `.python-version`, and provides project dependency locking. ([uv Python versions](https://docs.astral.sh/uv/concepts/python-versions/), [uv project configuration](https://docs.astral.sh/uv/concepts/projects/config/)) | Python-specific; does not provide a unified pin for Node and the common command-line tools. uv’s managed CPython builds come from Astral’s `python-build-standalone`, not from CPython distributing official binaries. ([uv Python versions](https://docs.astral.sh/uv/concepts/python-versions/)) |

The simplest fit is therefore mise for shared runtime and CLI selection, plus uv for Python project dependencies. Avoid adding nvm alongside mise for Node or using uv and mise both to select project Python unless a concrete compatibility need requires it.

## Browser and CLI boundary

Do not pin Playwright’s browser binary independently of the Playwright package that consumes it. Playwright documents that each release expects specific browser binaries, and updating Playwright can require reinstalling the matching browsers. Keep project Playwright and its dependencies under the project lock; the common image can provide OS libraries and the browser installation capability. ([Playwright browser versioning](https://playwright.dev/docs/browsers))

Pin image-provided AI CLIs to explicit releases where their official sources expose immutable versions and verifiable artifacts. Keep provider credentials outside the image and inject them at runtime, consistent with ODAF’s stated image lifecycle. If a CLI’s official distribution only offers a moving installer or channel, record that limitation and its resolved version in the manifest; do not claim byte-for-byte reproducibility from a version label alone. ([ODAF README: image and fleet lifecycle](../README.md#image-and-fleet-lifecycle))

## Update and validation policy

- Pin tool versions for each image build. Update versions deliberately, rebuild, and run ODAF’s planned W04–W06 capability and image checks before issuing a new image version. ([ODAF implementation plan](implementation-plan.md), [ADR 0002](adr/0002-layered-image-and-personalization.md))
- Treat the apt/native layer separately from mise. Record installed package versions and repositories in the manifest; a reproducible rebuild may additionally require a fixed package-repository snapshot. Validate package availability against Ubuntu 26.04 during the actual image build.
- For projects, commit lockfiles and use locked/frozen installation in automation where supported. Update project locks independently of changing the common image.
- Verify checksums/signatures or upstream attestations where the supplier publishes them. If none are supplied, record the source and state that artifact integrity was not independently established.
- Record compatibility separately for image, personalization and profile, matching ADR 0002. Keep secrets and local machine paths out of the manifest and public documentation. ([ADR 0002](adr/0002-layered-image-and-personalization.md), [ADR 0003](adr/0003-identity-and-secrets.md))

## Limits

This is a source-based recommendation, not a compatibility test of mise, uv, Node, AI CLIs, browser binaries, or apt packages on an ODAF Ubuntu 26.04 guest. The actual exact versions, backend choices, apt snapshot policy, manifest schema, update cadence and image build checks remain decisions for W04–W06 and must be validated during implementation. No tools were installed and no image was built for this research.
