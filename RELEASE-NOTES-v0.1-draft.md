<!--
  File: RELEASE-NOTES-v0.1-draft.md
  Version: 0.1.001 | Path: ForgeTrackHQ/protocol/RELEASE-NOTES-v0.1-draft.md
-->

# Release Notes — v0.1-draft

**Release date:** April 4, 2026
**Status:** Request for Comment
**License:** CC-BY-SA 4.0 (specification text), Apache 2.0 (code and schemas)

---

## Overview

This is the initial public release of the **ForgeTrack Protocol** — a canonical, signed, vendor-neutral event format for representing knowledge work produced by humans, AI systems, and the workflows that combine them.

This release is explicitly a **Request for Comment.** The specification is at v0.1-draft, and we are actively soliciting public critique from specifications engineers, compliance auditors, security architects, CPAs who prepare R&D tax credits, engineering leaders, and anyone else who will eventually need to emit, consume, or audit work events.

Breaking changes between v0.x minor versions are expected during the RFC period. The protocol will reach v1.0 (stable) after the RFC period concludes and at least two independent conforming implementations exist.

## What's In v0.1-draft

The specification is 12 Parts and 10 Appendices, organized so that different audiences can enter at the most relevant chapter rather than reading linearly:

- **Part I — Vision & Category:** the problem this solves, the category definition (Work Telemetry & Attribution Infrastructure), and the positioning of ForgeTrack as a standard, a verb, and a gateway.
- **Parts II–III — Architecture:** the six-layer architecture, the canonical event model, the event catalog, and signing/provenance/cryptographic-integrity primitives.
- **Part IV — Auth Plane:** Azure managed identity, API keys, workload identity federation, OAuth/OIDC, mTLS, SAML/SCIM, and scope/role definitions.
- **Part V — Data Model:** core entities, the events ledger, agent and attribution models, the identity registry, and provenance chains.
- **Part VI — Adapters:** MCP reference implementation, IDE adapters, Git platform adapters, communication platform adapters, productivity suite adapters, OpenTelemetry integration, CI/CD adapters, custom SDKs, and CLI integration.
- **Part VII — Intelligence:** AI-powered event enrichment and attribution inference.
- **Part VIII — Product Surfaces**
- **Part IX — Enterprise:** white-label, SSO, on-premises/air-gapped deployments, federated networks.
- **Part X — Compliance:** IRS R&D tax credits (Section 41 four-part test), SOC 2 Type II, ISO 27001 & 27701, GDPR & EU AI Act, HIPAA, SOX, SBIR/STTR, and investor-grade reporting.
- **Part XI — Roadmap:** the four-phase delivery plan and success metrics.
- **Part XII — Appendices:** complete event type catalog, CloudEvents 1.0 mapping, example payloads, DDL, API reference, SDK examples, threat model, glossary, AADM/AWAM specification, migration guide.

See the [specification table of contents](./spec/forgetrack-protocol-v0.1-draft.md#table-of-contents) for full navigation.

## What We're Asking For

Feedback that would be most valuable at this stage:

### From specifications reviewers

- Is the normative language (MUST/SHOULD/MAY) used consistently and correctly?
- Are there ambiguities or contradictions between chapters?
- Is the scope appropriately bounded, or too broad?
- Does the architecture hold together as a cohesive design, or does it read as bolted-together parts?
- How does it compare to adjacent standards (CloudEvents, OpenTelemetry, SLSA, in-toto)?

### From implementers

- Is the specification implementable as written, or are there gaps?
- Which parts would be hardest to implement correctly?
- Are the conformance expectations clear?
- What would you need to build an adapter for your tooling?

### From compliance and audit professionals

- Would IRS R&D tax credit documentation built on this specification survive Section 41 scrutiny?
- Are the SOC 2, ISO 27001, GDPR, and HIPAA chapters aligned with current auditor expectations?
- What's missing for EU AI Act compliance?

### From security architects

- Is the signing scheme sufficient?
- Are there attack surfaces in the auth plane that aren't addressed in the threat model?
- Does the federation model hold up under adversarial conditions?

### From engineering leaders and founders

- Would the output of this protocol actually help you make decisions?
- Would investors accept reports built on this specification?
- What dashboards would you need?

## Known Open Questions

The CHANGELOG lists open questions where we specifically want feedback. Highlights:

- **Attribution confidence representation** — is 0.0–1.0 float per contributor-role pairing the right granularity?
- **Signing algorithm negotiation** — v0.1 defaults to Ed25519; multi-algorithm support semantics need refinement.
- **Event chain verification across tenant boundaries** — federated verification is specified but not yet prototyped.
- **Breaking change policy between v0.x releases** — the 30-day review window may need adjustment.
- **Conformance testing requirements** — formal v1.0 conformance criteria are not yet defined.

## What's Not In v0.1-draft

Deferred to v0.2 or later:

- JSON schemas for event types (planned v0.2)
- Example event payloads in JSON form (planned v0.2)
- `rfcs/` folder and first formal RFCs
- Reference SDK releases (TypeScript, C#, Python, Go)
- Reference adapter implementations (MCP, IDE, Git platform, Slack, etc.)
- Formal conformance test suite
- PGP key for security reports

## How to Engage

- **Read the spec** at [`spec/forgetrack-protocol-v0.1-draft.md`](./spec/forgetrack-protocol-v0.1-draft.md)
- **Open a [Discussion](https://github.com/ForgeTrackHQ/protocol/discussions)** for open-ended critique and use-case reports
- **Open an [Issue](https://github.com/ForgeTrackHQ/protocol/issues)** for concrete bugs, ambiguities, or small change proposals
- **File an RFC** for substantive proposals (see [CONTRIBUTING.md](./CONTRIBUTING.md))
- **Email protocol@forgetrack.org** for private inquiries

We commit to engaging publicly with all substantive feedback. We will not silently ignore issues or discussions, even ones we disagree with — declining a proposal is a decision, and declined decisions will be documented with reasoning.

## Timeline

| Milestone | Target |
|---|---|
| v0.1-draft published (this release) | ✓ April 2026 |
| v0.2-draft (schemas, examples, first RFCs) | Summer 2026 |
| v0.3-draft (incorporating RFC-period feedback) | Fall 2026 |
| v1.0-rc1 (release candidate) | Winter 2026 / 2027 |
| v1.0 stable (requires ≥2 independent conforming implementations) | 2027 |

These dates are targets, not commitments. The RFC period concludes when the specification is stable enough for implementers to commit, not on a calendar deadline.

## Governance Note

The ForgeTrack Protocol is currently at **Phase 1 — Stewardship**, stewarded by [Edson Technologies Inc.](https://edsontechnologies.com). We have publicly committed to transitioning to neutral, multi-stakeholder governance (Phase 2) on objective criteria documented in [GOVERNANCE.md](./GOVERNANCE.md). The specification is licensed under CC-BY-SA 4.0 specifically so that if the Steward ever acts against the community's interest, the specification can be forked and continued independently.

## Acknowledgements

This initial release was drafted by the team at Edson Technologies Inc., building on prior work from the open specifications community. See [NOTICE](./NOTICE) for attributions.

---

*Thank you for reading. The best way to help right now is to disagree with us in public.*

— The ForgeTrack Protocol Maintainers
