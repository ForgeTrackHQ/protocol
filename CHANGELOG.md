# Changelog

All notable changes to the ForgeTrack Protocol specification are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html) at the specification level (see [README — Versioning](./README.md#versioning)).

During the `-draft` phase (v0.x), breaking changes may occur between minor versions and will be documented here with migration guidance.

---

## [Unreleased]

### Added
- Placeholder for changes in the next release

---

## [0.1-draft] — 2026-04-04

Initial public release. Request for Comment.

### Added
- Complete v0.1-draft specification (74 chapters, 10 appendices) published under CC-BY-SA 4.0
- Part I — Vision & Category (chapters 1–4): problem framing, category definition (Work Telemetry & Attribution Infrastructure), positioning, AADM/AWAM open standard introduction
- Part II — Architectural Foundations (chapters 5–9): architectural principles, six-layer architecture, canonical event model, event types & catalog, signing & provenance
- Part III — Layer-by-Layer Deep Dive (chapters 10–15): adapter surfaces, unified ingestion API, canonical event store, intelligence & enrichment, analytics & compliance, ecosystem
- Part IV — Authentication & Authorization Plane (chapters 16–23): Azure managed identity, API keys, workload identity federation, OAuth/OIDC, mTLS, SAML/SCIM, scopes & roles
- Part V — Data Model (chapters 24–31): core entities, events table, agents & attributions, tools & service principals, API keys & audit trail, workflows & provenance chains, projections, migration strategy
- Part VI — Adapters (chapters 32–40): MCP reference implementation, IDE adapters, Git platform adapters, communication platform adapters, productivity suite adapters, OpenTelemetry integration, CI/CD adapters, custom SDKs, CLI & terminal
- Part VII — Intelligence & Analytics (chapters 41+): AI-powered enrichment, attribution inference
- Part VIII — Product Surfaces
- Part IX — Enterprise Features (chapters 57–61): white-label, SSO, on-prem/air-gapped, federated networks
- Part X — Compliance & Reporting (chapters 62–69): IRS R&D tax credits, SOC 2, ISO 27001/27701, GDPR & EU AI Act, HIPAA, SOX, SBIR/STTR, investor-grade reporting
- Part XI — Phased Roadmap (chapters 70–74)
- Appendix A — Complete Event Type Catalog
- Appendix B — CloudEvents 1.0 Mapping
- Appendix C — Example Event Payloads
- Appendix D — DDL for New Tables
- Appendix E — API Endpoint Reference
- Appendix F — SDK Examples
- Appendix G — Threat Model
- Appendix H — Glossary & Terminology
- Appendix I — AADM/AWAM Specification v0.1
- Appendix J — Migration Guide
- Repository governance model (Phase 1 Stewardship, with documented phase transition criteria)
- RFC process for substantive specification changes
- Dual-license structure (CC-BY-SA 4.0 for specification text, Apache 2.0 for code and schemas)

### Known Open Questions
Areas where reviewer feedback is especially welcome during the RFC period:

- **Attribution confidence representation** — the spec describes confidence as a 0.0–1.0 float per contributor-role pairing; feedback on whether this is sufficient granularity is welcome
- **Signing algorithm negotiation** — v0.1 defaults to Ed25519; multi-algorithm support semantics may need refinement
- **Event chain verification across tenant boundaries** — federated verification is specified but not fully prototyped
- **Breaking change policy between v0.x releases** — the 30-day review window may need adjustment based on implementer feedback
- **Conformance testing requirements** — formal conformance criteria for v1.0 eligibility are not yet defined

### Not Yet Included
Deferred to v0.2 or later:

- JSON schemas for event types (planned for v0.2, referenced from Appendix A)
- Example event payloads in JSON form (planned for v0.2, referenced from Appendix C)
- `rfcs/` folder and first RFCs
- `CONTRIBUTORS.md`
- Formal conformance test suite
- Reference SDK releases (TypeScript, C#, Python, Go)
- Reference adapter implementations

### License
- Specification text: CC-BY-SA 4.0
- Code, schemas, and examples: Apache 2.0

### Steward
Edson Technologies Inc. (Phase 1 Stewardship per [GOVERNANCE.md](./GOVERNANCE.md))

---

[Unreleased]: https://github.com/ForgeTrackHQ/protocol/compare/v0.1-draft...HEAD
[0.1-draft]: https://github.com/ForgeTrackHQ/protocol/releases/tag/v0.1-draft
