# Changelog

All notable changes to the GAAIM specification are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html) at the specification level (see [README — Versioning](./README.md#versioning)).

During the `-draft` phase (v0.x), breaking changes are permitted between minor versions and would be documented here with migration guidance.

---

## [Unreleased]

### Added

- Placeholder for changes in the next release

---

## [0.1-draft] — 2026-04-05

Initial publication of GAAIM — Core v0.1-draft and Engineering Profile v0.1-draft, released together.

### Added — Core (§1–§8, §10–§12)

- **§1 Introduction** — scope definition, requirements language (RFC 2119 / RFC 8174), document conventions
- **§2 Architecture: Core + Profiles** — the Core + Profiles pattern (FHIR/OpenID Connect precedent), cross-profile interoperability guarantees
- **§3 Core Event Model** — CloudEvents 1.0 profile, GAAIM extension attributes (`gaaimversion`, `profile`, `tenantid`, `traceid`, `correlationid`, `causationid`, `prev`, `signature`, `signaturekey`, `auditeligible`), the `data` object, attribution model with principal types (Human, AIModel, Tool, Service, Workflow, System, External) and roles (proposer, reviewer, approver, executor, observer), three principal-identifier forms (full, vendor-only, anonymized), ULID and UUIDv7 event identifier support
- **§4 Canonical Serialization** — RFC 8785 (JCS) with GAAIM-specific exclusions, absent-attribute rule (MUST NOT synthesize as null), number-serialization guidance, normative test vectors
- **§5 Signing & Provenance** — Ed25519 MTI per RFC 8032, algorithm registry with prefix-encoded signatures, URI-qualified HTTPS `signaturekey` binding, Key Registry HTTP API (`GET /v1/keys/{keyId}`, `/.well-known/gaaim-registry` discovery), prefix-encoded public keys, key rotation and revocation, chain continuity verification with gap-detection semantics, reserved prefixes for post-quantum algorithms (`ml-dsa:`, `slh-dsa:`, `fn-dsa:`, `hybrid:`), migration planning for long-horizon integrity
- **§6 Core Event Taxonomy** — `gaaim.core.key.rotated`, `gaaim.core.key.revoked`, `gaaim.core.principal.registered`, `gaaim.core.agent.handoff`
- **§7 Profile Architecture** — profile specification requirements, profile registry, four-stage lifecycle (Proposed, Draft, Candidate, Ratified), tenant custom extensions under `gaaim.<tenant-slug>.*`, namespace registration authority, governance posture
- **§8 Compliance Levels** — L0 Development, L1 Emitter, L2 Sink, L3 Processor, orthogonal to profile conformance, conformance manifest requirements
- **§10 Security Considerations** — threat model (six adversaries), key compromise handling, PII handling across three principal-identifier forms, tenant isolation, registry origin allowlisting (§10.4.1), registry trust, common verification pitfalls
- **§11 IANA Considerations** — `application/gaaim-event+json` media type, CloudEvents extension attribute registry entries, `expertise://` URI scheme
- **§12 References** — normative and informative references, related-work comparison matrix against in-toto, SLSA, and C2PA

### Added — Engineering Profile (§9)

- Profile slug `eng`, version v0.1-draft, fifteen event types across five categories:
  - **Version-control lifecycle:** `gaaim.eng.commit.authored`, `gaaim.eng.commit.reviewed`, `gaaim.eng.branch.merged`, `gaaim.eng.pr.opened`, `gaaim.eng.pr.merged`, `gaaim.eng.pr.closed`
  - **Review:** `gaaim.eng.review.completed`
  - **Build and deploy:** `gaaim.eng.build.completed`, `gaaim.eng.test.executed`, `gaaim.eng.deploy.initiated`, `gaaim.eng.deploy.completed`
  - **AI workflow:** `gaaim.eng.ai.prompt.submitted`, `gaaim.eng.ai.response.received`, `gaaim.eng.ai.suggestion.accepted`, `gaaim.eng.ai.suggestion.rejected`, `gaaim.eng.codegen.applied`
  - **Supply chain:** `gaaim.eng.dependency.updated`, `gaaim.eng.security.scanned`
- Reference format normalization rules (§9.4.0) — prefix-encoded commit hashes, normalized repository URIs, prefix-encoded artifact references
- Engineering-specific attribution roles extending Core base-set
- Expertise vocabulary with `expertise://` identifiers for engineering domains

### Added — Appendices

- **Appendix A** — Worked examples with signature computation, canonicalization walkthroughs, and normative test vectors
- **Appendix B** — JSON Schemas (referenced normatively)
- **Appendix C** — Profile Author Guide (reference framework for hypothetical future stewardship body)
- **Appendix D** — Glossary
- **Appendix E** — Change log (this file's predecessor; spec-internal change record)

### Added — Repository

- Dual-license structure: CC-BY-SA 4.0 (specification text) and Apache 2.0 (code and schemas)
- Published-reference governance posture documented in [GOVERNANCE.md](./GOVERNANCE.md) and §7.6 of the specification
- Contribution guidance in [CONTRIBUTING.md](./CONTRIBUTING.md), Code of Conduct in [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md), security disclosure policy in [SECURITY.md](./SECURITY.md)

### License

- Specification text: CC-BY-SA 4.0
- Code, schemas, and examples: Apache 2.0

### Publisher

Edson Technologies, Inc. — first implementer via [ForgeTrack](https://forgetrack.io). See [GOVERNANCE.md](./GOVERNANCE.md) for the published-reference posture.

---

[Unreleased]: https://github.com/GAAIM-standard/spec/compare/v0.1-draft...HEAD
[0.1-draft]: https://github.com/GAAIM-standard/spec/releases/tag/v0.1-draft
