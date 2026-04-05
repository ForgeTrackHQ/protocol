# Release Notes — v0.1-draft

**Release date:** April 5, 2026
**Status:** Published Reference Specification
**License:** CC-BY-SA 4.0 (specification text), Apache 2.0 (code and schemas)

---

## Overview

This is the initial public release of **GAAIM** (Generally Accepted AI Metrics, pronounced *"game"*) — a canonical, signed, vendor-neutral event format for representing knowledge work produced by humans, AI systems, and the workflows that combine them.

GAAIM is published as a **reference specification**. It defines the event model underlying [ForgeTrack](https://forgetrack.io), Edson Technologies' product for tracking AI-amplified engineering work. Publishing the specification openly lets ForgeTrack's provenance claims be independently verified, and gives implementers a reference if they want to emit compatible events from their own tools.

See [GOVERNANCE.md](./GOVERNANCE.md) for the current governance posture. No formal working group is currently operating; the specification is intended to be contributed to a neutral standards body if independent implementer interest emerges.

---

## What's In v0.1-draft

GAAIM v0.1-draft consists of two parts published together:

### GAAIM Core v0.1-draft

The vertical-neutral foundation, covering:

- **§1 Introduction** — scope, requirements language, document conventions
- **§2 Architecture: Core + Profiles** — the Core + Profiles pattern, its precedent, and cross-profile interoperability
- **§3 Core Event Model** — the CloudEvents 1.0 profile, CloudEvents attributes, GAAIM-specific extension attributes, the `data` object, the attribution model, principal types and roles, and event identifiers
- **§4 Canonical Serialization** — JSON Canonicalization Scheme (RFC 8785) with GAAIM-specific exclusions, absent-attribute handling, number serialization, and normative test vectors
- **§5 Signing & Provenance** — Ed25519-based origin signing, the algorithm registry, the Key Registry HTTP API, URI-qualified key identifiers, key rotation and revocation, chain continuity, and the deferred transparency-log direction
- **§6 Core Event Taxonomy** — the core administrative event types (key lifecycle, principal lifecycle, agent handoff)
- **§7 Profile Architecture** — how profiles extend Core, the profile registry, the profile lifecycle, tenant custom extensions, namespace rules, and the current governance posture
- **§8 Compliance Levels** — L0 Development through L3 Processor, orthogonal to profile support
- **§10 Security Considerations** — threat model, key compromise, PII handling, tenant isolation, registry origin allowlisting, and registry trust
- **§11 IANA Considerations** — media type registration and CloudEvents extension attribute registry entries
- **§12 References** — normative and informative references, plus related-work positioning against in-toto, SLSA, and C2PA

### GAAIM Engineering Profile v0.1-draft

The first vertical profile, covering software-engineering knowledge work:

- **§9 Engineering Profile** — profile metadata, fifteen event types covering version-control lifecycle, review, build, deploy, AI workflow, and supply-chain events; engineering-specific attribution conventions; and an expertise vocabulary

### Appendices

- **Appendix A** — Worked examples, including full signature computation and canonicalization walkthroughs with test vectors
- **Appendix B** — JSON Schemas (normative, referenced from the specification)
- **Appendix C** — Profile Author Guide (describes the framework under which new profiles would be proposed if a stewardship body were operating)
- **Appendix D** — Glossary of terms used throughout the specification
- **Appendix E** — Change log

---

## Not Yet Included

The following are referenced in the specification but not yet published in separate repositories:

- Reference canonicalization libraries in the five MTI languages (TypeScript, C#, Python, Go, Rust) — planned for [canonicalize](https://github.com/GAAIM-standard/canonicalize)
- Reference signature verifiers — planned for [verifier](https://github.com/GAAIM-standard/verifier)
- Machine-readable JSON Schemas at a stable URL — planned for [schemas](https://github.com/GAAIM-standard/schemas)
- Conformance test vectors as standalone fixtures — planned for [conformance-tests](https://github.com/GAAIM-standard/conformance-tests)
- Reference Key Registry implementation
- Additional vertical profiles beyond Engineering (Legal, Healthcare, Financial Services, etc.)

Until these are published, implementers should treat the specification text as the authoritative source and derive schemas, fixtures, and verification logic directly from it.

---

## How to Engage

- **Read the spec:** [`gaaim-core-and-engineering-profile-v0_1.md`](./gaaim-core-and-engineering-profile-v0_1.md)
- **Discussions:** [GitHub Discussions](https://github.com/GAAIM-standard/spec/discussions) for open-ended feedback, implementation reports, and questions
- **Issues:** [GitHub Issues](https://github.com/GAAIM-standard/spec/issues) for concrete problems — typos, broken links, ambiguities
- **Pull Requests:** editorial PRs welcome; see [CONTRIBUTING.md](./CONTRIBUTING.md)
- **Private inquiries:** `spec@gaaim.org`
- **Security:** see [SECURITY.md](./SECURITY.md)

Response times are best-effort under the current published-reference posture.

---

## Licensing

- Specification text: [CC-BY-SA 4.0](./LICENSE-SPEC)
- Code, schemas, and examples: [Apache 2.0](./LICENSE-CODE)
- Both licenses are permanent and irrevocable. Anyone may fork the specification at any time.

---

## Publisher

GAAIM v0.1-draft is published by [Edson Technologies, Inc.](https://edson.tech), developer of [ForgeTrack](https://forgetrack.io) (the first production implementation). See [GOVERNANCE.md](./GOVERNANCE.md) for the full posture.

---

*The best way to help this specification become useful is to implement it. If you ship a conformant implementation, tell us — open a Discussion and we'll track it.*
