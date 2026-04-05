# GAAIM

**Generally Accepted AI Metrics** — a canonical, signed, vendor-neutral event format for representing knowledge work produced by humans, AI systems, and the workflows that combine them.

- **Website:** [gaaim.org](https://gaaim.org)
- **Specification:** [Read v0.1-draft](./gaaim-core-and-engineering-profile-v0_1.md)
- **License:** [CC-BY-SA 4.0](./LICENSE-SPEC) (specification text) · [Apache 2.0](./LICENSE-CODE) (schemas, fixtures, example code)
- **Status:** Published Reference Specification — not actively stewarded
- **First implementation:** [ForgeTrack](https://forgetrack.com)

---

## What This Is

GAAIM (pronounced *"game"*) specifies how work events — commits, deployments, AI inferences, document edits, approvals, conversations, reviews — can be emitted, signed, stored, attributed, and federated across organizations in a vendor-neutral format.

It is designed to serve four audiences whose needs have historically been met by separate, incompatible tools:

- **Engineering leaders** who want to measure output across humans and AI without committing to a single vendor's telemetry format
- **Compliance auditors** (SOC 2, ISO 27001, HIPAA, SOX, GDPR, EU AI Act) who need tamper-evident records of who did what, when, and with what authorization
- **Tax professionals** preparing R&D tax credit documentation (IRS Section 41, SBIR/STTR, grant reporting) that must survive substantiation scrutiny
- **Investors and board members** who want comparable, auditable reporting on engineering productivity, AI adoption, and contribution attribution

The specification defines a canonical event envelope (built as a CloudEvents 1.0 profile), a first-class attribution model, a cryptographic signing protocol using Ed25519 and RFC 8785 canonicalization, and a compliance-level framework. Together these let any tool in a knowledge-work stack emit audit-defensible evidence of what was produced, by whom, and with what AI assistance.

This repository contains **the specification text only**. Reference implementations (SDKs, adapters, conformance test suites) are planned in separate repositories under the [GAAIM-standard](https://github.com/GAAIM-standard) organization.

---

## Who Should Read What

The specification is long. Different audiences should enter at different chapters rather than reading linearly.

| If you are a…                                                | Start here                                          | Then read                                                    | Skip unless relevant             |
| ------------------------------------------------------------ | --------------------------------------------------- | ------------------------------------------------------------ | -------------------------------- |
| **Product or engineering leader** evaluating the specification | §1 (Introduction), §2 (Architecture)                | §7 (Profile Architecture), §9 (Engineering Profile)          | §4, §10, §11                     |
| **Architect or protocol designer**                           | §3 (Core Event Model), §4 (Canonical Serialization) | §5 (Signing), §10 (Security Considerations)                  | §9                               |
| **Implementer** building an emitter or sink                  | §3 (Core Event Model), §5 (Signing & Provenance)    | §4 (Canonical Serialization), §8 (Compliance Levels), Appendix A (Worked Examples) | §12                              |
| **Security architect or auditor**                            | §5 (Signing), §10 (Security Considerations)         | §3 (Core Event Model), §8 (Compliance Levels)                | §9 (unless engineering-specific) |
| **Compliance officer or R&D tax professional**               | §1 (Introduction), §3.5 (Attribution Model)         | §8 (Compliance Levels), §9 (Engineering Profile)             | §4, §5, §11                      |
| **Enterprise IT / DevOps** considering adoption              | §2 (Architecture), §8 (Compliance Levels)           | §5 (Signing), §7 (Profile Architecture)                      | Appendices B, C                  |

Appendix D (Glossary) is useful regardless of entry point — definitions of terms used throughout the specification.

---

## Repository Contents

```
spec/
├── README.md                                    ← you are here
├── LICENSE-SPEC                                 ← CC-BY-SA 4.0 (specification text)
├── LICENSE-CODE                                 ← Apache 2.0 (code, schemas, examples)
├── NOTICE                                       ← attribution and acknowledgments
├── GOVERNANCE.md                                ← governance posture
├── CONTRIBUTING.md                              ← how to contribute, expectations
├── CODE_OF_CONDUCT.md                           ← Contributor Covenant v2.1
├── SECURITY.md                                  ← vulnerability disclosure policy
├── CHANGELOG.md                                 ← version history
├── RELEASE-NOTES-v0.1-draft.md                  ← v0.1-draft summary
└── gaaim-core-and-engineering-profile-v0_1.md   ← the specification itself
```

Related repositories under [github.com/GAAIM-standard](https://github.com/GAAIM-standard) are envisioned to host:

| Repository          | Purpose                                                      | Status    |
| ------------------- | ------------------------------------------------------------ | --------- |
| `spec`              | This repo — specification text                               | Published |
| `canonicalize`      | Reference canonicalization libraries (TypeScript, C#, Python, Go, Rust) | Planned   |
| `verifier`          | Reference signature verifiers                                | Planned   |
| `conformance-tests` | Conformance test fixtures and vectors                        | Planned   |
| `schemas`           | JSON Schemas for envelope and profile events                 | Planned   |
| `profiles`          | Additional vertical profile specifications                   | Planned   |

Until the planned repositories are populated, implementers should treat the specification text as the authoritative source and derive schemas and test vectors directly from it.

---

## Versioning

The specification uses [Semantic Versioning](https://semver.org/):

- **Major version (1.0, 2.0)** — breaking changes that require conforming implementations to change their code
- **Minor version (1.1, 1.2)** — additive changes that do not break conformance
- **Patch version (1.1.1)** — editorial changes only, no normative impact

During the `-draft` phase (v0.x), breaking changes are permitted between minor versions and would be documented in the CHANGELOG with migration guidance. This relaxation ends at v1.0.

The v0.1-draft specification is stable as published. It will not be revised except to correct errors; substantive changes would be made in a future v0.x or v1.0 revision.

---

## License

This repository uses a **dual-license structure**:

| Content                | License                        | Applies to                                                   |
| ---------------------- | ------------------------------ | ------------------------------------------------------------ |
| **Specification text** | [CC-BY-SA 4.0](./LICENSE-SPEC) | The specification document, the README, the CHANGELOG, release notes, governance documents, and all prose in this repository |
| **Code and schemas**   | [Apache 2.0](./LICENSE-CODE)   | JSON schemas, DDL, code examples, CI configuration, and any executable or structured-data content |

**Why dual-license?** Specification text and code have different sharing expectations. CC-BY-SA 4.0 on the specification text ensures that derivative specifications carry forward the same open license (copyleft), preventing anyone from fencing off an evolved version behind proprietary licensing. Apache 2.0 on code allows implementers to freely incorporate schemas and example code into products under any license, without the copyleft burden that would make CC-BY-SA problematic for production use.

This structure follows the precedent set by W3C specifications (which use CC-BY for text and W3C Software License for code) and by the CNCF's approach to open standards.

Any contribution to this repository is licensed under the license that applies to the content being contributed.

---

## Contributing

Contributions are welcome — typo fixes, clarifications, implementation reports, and substantive technical feedback.

See [CONTRIBUTING.md](./CONTRIBUTING.md) for:

- How to open Discussions, Issues, and Pull Requests
- What kinds of changes are most likely to be merged
- Expectations about response times

**Short version:** editorial PRs (typos, clarifications, broken links) are welcome and get addressed on a best-effort basis. Normative changes are evaluated against the current specification direction but may not be merged under the current published-reference posture. Opening a Discussion first is usually the right move for substantive proposals.

---

## Governance

GAAIM is published as a **reference specification**. Edson Technologies, Inc. is the publisher and first implementer (via [ForgeTrack](https://forgetrack.com)). No formal working group is currently operating, and there is no RFC process in flight.

If independent implementer interest emerges — multiple parties shipping GAAIM-conformant producers or sinks, interop requests, or demand from regulators and auditors — the specification is intended to be contributed to a neutral standards body at that time. Edson Technologies does not intend to operate a long-term independent standards process.

See [GOVERNANCE.md](./GOVERNANCE.md) and [§7.6 of the specification](./gaaim-core-and-engineering-profile-v0_1.md#76-governance-posture) for the current posture in full.

The specification text is irrevocably licensed under CC-BY-SA 4.0 and code under Apache 2.0. Anyone may fork the specification at any time.

---

## Contact

- **Public questions and discussions:** [GitHub Discussions](https://github.com/GAAIM-standard/spec/discussions)
- **Concrete problems:** [GitHub Issues](https://github.com/GAAIM-standard/spec/issues)
- **Security vulnerabilities:** see [SECURITY.md](./SECURITY.md) — do not open public Issues
- **Code of conduct reports:** see [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)
- **Private correspondence:** `spec@gaaim.org`

---

*GAAIM is published by [Edson Technologies, Inc.](https://edson.tech) as the event architecture underlying [ForgeTrack](https://forgetrack.com). If it's useful, use it. If it isn't, fork it. The event model should outlive whoever first wrote it down.*
