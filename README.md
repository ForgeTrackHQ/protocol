<!--
  File: README.md
  Version: 0.1.003 | Path: ForgeTrackHQ/protocol/README.md
-->

# ForgeTrack Protocol

**A canonical, signed, vendor-neutral event format for representing knowledge work produced by humans, AI systems, and the workflows that combine them.**

> ## ⚠️ REQUEST FOR COMMENT — v0.1-draft
>
> This specification is a **draft**. It is published to solicit public critique, attract reference implementers, and establish a public timestamp for its development. Breaking changes between v0.x minor versions should be expected. The protocol will reach **v1.0 (stable)** only after the RFC period concludes and at least two independent conforming implementations exist.
>
> Feedback is actively wanted. See [RELEASE-NOTES-v0.1-draft.md](./RELEASE-NOTES-v0.1-draft.md) for what we're specifically asking reviewers to weigh in on, and [CONTRIBUTING.md](./CONTRIBUTING.md) for how to contribute.

---

## What This Is

The ForgeTrack Protocol specifies how work events — commits, deployments, AI inferences, document edits, approvals, conversations, reviews — can be emitted, signed, stored, attributed, and federated across organizations in a vendor-neutral format.

It is designed to serve four audiences whose needs have historically been met by separate, incompatible tools:

- **Engineering leaders** who want to measure output across humans and AI without committing to a single vendor's telemetry format
- **Compliance auditors** (SOC 2, ISO 27001, HIPAA, SOX, GDPR, EU AI Act) who need tamper-evident records of who did what, when, and with what authorization
- **Tax professionals** preparing R&D tax credit documentation (IRS Section 41, SBIR/STTR, grant reporting) that must survive substantiation scrutiny
- **Investors and board members** who want comparable, auditable reporting on engineering productivity, AI adoption, and contribution attribution

The specification defines a canonical event model, a cryptographic signing and provenance scheme, an authentication plane, adapter interfaces for common tools (IDEs, Git platforms, MCP servers, OpenTelemetry, CI/CD, productivity suites), and a data model for storing, enriching, and querying the resulting event stream.

This repository contains **the specification text only**. Reference implementations (SDKs, adapters, conformance test suites) live in separate repositories under the [ForgeTrackHQ](https://github.com/ForgeTrackHQ) organization.

---

## Who Should Read What

The specification is long. Different audiences should enter at different chapters rather than reading linearly.

| If you are a… | Start here | Then read | Skip unless relevant |
|---|---|---|---|
| **Product or engineering leader** evaluating the protocol | Part I (Vision & Category) | Part VIII (Product Surfaces), Part XI (Roadmap) | Parts IV, V, VI |
| **Architect or protocol designer** | Parts II–III (Architectural Foundations, Layer Deep Dive) | Part V (Data Model), Appendix G (Threat Model) | Part X |
| **Implementer** building an adapter or SDK | Part VI (Adapters) | Part III (Layer-by-Layer), Appendices A–F | Part X |
| **Security architect or auditor** | Part IV (Auth Plane), Appendix G (Threat Model) | Parts II–III, §9 (Signing & Provenance) | Part VIII |
| **Compliance officer or R&D tax professional** | Part X (Compliance & Reporting) | Part V (Data Model), Appendix I (AADM/AWAM) | Parts IV, VI |
| **Enterprise IT / DevOps** considering adoption | Part IX (Enterprise & Scale) | Part IV (Auth Plane), Part XI (Roadmap) | Appendices B, F |

Appendix H (Glossary) is useful regardless of entry point — definitions of terms used throughout the specification.

---

## Repository Contents

```
protocol/
├── README.md                         ← you are here
├── LICENSE-SPEC                      ← CC-BY-SA 4.0 (specification text)
├── LICENSE-CODE                      ← Apache 2.0 (code, schemas, examples)
├── NOTICE                            ← attribution and acknowledgments
├── GOVERNANCE.md                     ← stewardship model, phase transitions
├── CONTRIBUTING.md                   ← how to contribute, RFC process, style guide
├── CODE_OF_CONDUCT.md                ← Contributor Covenant v2.1
├── SECURITY.md                       ← vulnerability disclosure policy
├── CHANGELOG.md                      ← version history
├── RELEASE-NOTES-v0.1-draft.md       ← this release, what we're asking for
└── spec/
    └── forgetrack-protocol-v0.1-draft.md  ← the specification itself
```

Additional files will appear as the project matures:

- `MAINTAINERS.md` — when the second Maintainer is appointed
- `IMPLEMENTERS.md` — when the first external Reference Implementer declares
- `CONTRIBUTORS.md` — with the first external contribution
- `rfcs/` — with the first RFC

---

## Versioning

The specification uses [Semantic Versioning](https://semver.org/) at the specification level:

- **Major version (1.0, 2.0)** — breaking changes that require conforming implementations to change their code
- **Minor version (1.1, 1.2)** — additive changes that do not break conformance
- **Patch version (1.1.1)** — editorial changes only, no normative impact

During the `-draft` phase (v0.x), the normal semver rules are relaxed: breaking changes are expected between v0.x minor versions and are documented in the CHANGELOG with migration guidance. This relaxation ends at v1.0.

### Criteria for v1.0 (Stable)

The specification transitions from `-draft` to `v1.0` only when **all** of the following are true:

- The public RFC period has concluded with no open breaking-change RFCs
- **At least two independent conforming implementations exist** (where "independent" means not controlled by or financially dependent on the Steward)
- A public conformance test suite has been published
- Formal review by declared Reference Implementers has occurred

This criterion is borrowed from W3C and IETF practice: a specification does not become "stable" because its authors declare it stable. It becomes stable because the real world has proven it implementable.

Until v1.0, implementers should track the `main` branch and expect periodic breaking changes. After v1.0, breaking changes require a Major version bump and extended deprecation periods.

---

## License

This repository uses a **dual-license structure**:

| Content | License | Applies to |
|---|---|---|
| **Specification text** | [CC-BY-SA 4.0](./LICENSE-SPEC) | The protocol specification in `spec/`, the README, the CHANGELOG, release notes, governance documents, and all prose in this repository |
| **Code and schemas** | [Apache 2.0](./LICENSE-CODE) | JSON schemas, DDL, code examples, CI configuration, and any executable or structured-data content |

**Why dual-license?** Specification text and code have different sharing expectations. CC-BY-SA 4.0 on the specification text ensures that derivative specifications carry forward the same open license (copyleft), preventing anyone from fencing off an evolved version behind proprietary licensing. Apache 2.0 on code allows implementers to freely incorporate schemas and example code into products under any license, without the copyleft burden that would make CC-BY-SA problematic for production use.

This structure follows the precedent set by W3C specifications (which use CC-BY for text and W3C Software License for code) and by the CNCF's approach to open standards.

Any contribution to this repository is licensed under the license that applies to the content being contributed. Sign-off on your commits (see [CONTRIBUTING.md §6.2](./CONTRIBUTING.md#62-developer-certificate-of-origin-dco)) certifies that you have the right to contribute under these terms.

---

## Contributing

Contributions of all kinds are welcome — typo fixes, substantive objections, alternative designs, implementation reports, compliance-use-case feedback.

See [CONTRIBUTING.md](./CONTRIBUTING.md) for:

- How to open Discussions, Issues, PRs, and RFCs
- The full RFC process, template, and lifecycle
- The specification text style guide (normative language, voice, examples)
- Commit conventions and DCO sign-off requirements

New contributors should start by opening a Discussion. It's better to talk through scope for five minutes than to write a full RFC that turns out to be solving the wrong problem.

---

## Governance

The ForgeTrack Protocol is currently in **Phase 1: Single-Organization Stewardship**, with Edson Technologies Inc. serving as Steward. Phase transition criteria toward community governance (Phase 2) and foundation governance (Phase 3) are documented in [GOVERNANCE.md](./GOVERNANCE.md).

The specification text is irrevocably licensed under CC-BY-SA 4.0 and code under Apache 2.0. Anyone may fork the specification at any time. The Steward's authority exists only insofar as the community recognizes this repository as canonical.

---

## Contact

- **Public questions and discussions:** [GitHub Discussions](https://github.com/ForgeTrackHQ/protocol/discussions)
- **Concrete problems:** [GitHub Issues](https://github.com/ForgeTrackHQ/protocol/issues)
- **Security vulnerabilities:** see [SECURITY.md](./SECURITY.md) — do not open public Issues
- **Code of conduct reports:** see [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)
- **Private correspondence:** `protocol@forgetrack.org`

---

*The ForgeTrack Protocol is stewarded by [Edson Technologies Inc.](https://edsontechnologies.com) with the stated intent to transition to neutral foundation governance as adoption warrants.*
