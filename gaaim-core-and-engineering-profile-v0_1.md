# GAAIM — Generally Accepted AI Metrics

## Core Specification v0.1-draft + Engineering Profile v0.1-draft

---

**Status:** Published Reference Specification — v0.1-draft
**Published:** April 5, 2026
**Canonical URL:** https://gaaim.org/spec/v0.1-draft
**Repository:** https://github.com/GAAIM-standard/spec
**Publisher:** Edson Technologies, Inc.
**License:** Specification text is CC-BY-SA 4.0. Reference schemas, test fixtures, and example code are Apache 2.0.

---

## Abstract

GAAIM (Generally Accepted AI Metrics, pronounced *"game"*) is an open, vendor-neutral specification for signed, attributed records of knowledge work produced by humans, AI systems, and the workflows that combine them. It defines a canonical event envelope (built as a CloudEvents 1.0 profile), a first-class attribution model, a cryptographic signing protocol, and a compliance-level framework that together let any tool in a knowledge-work stack emit audit-defensible evidence of what was produced, by whom, and with what AI assistance.

This document specifies **GAAIM Core v0.1-draft** (the vertical-neutral foundation) and **GAAIM Engineering Profile v0.1-draft** (the first vertical profile, defining engineering-specific event types, attribution roles, and expertise vocabulary). The two are published together; subsequent profiles (Legal, Healthcare, Financial, Consulting, and others) will follow on independent release tracks under the profile lifecycle defined in §7.3.

This is a **published reference specification**. It defines the event model underlying [ForgeTrack](https://forgetrack.com) and is published openly for implementer reference, auditor review, and ecosystem optionality. Edson Technologies does not currently operate a formal standards working group; if independent implementer interest emerges, the specification is intended to be contributed to a neutral standards body at that time. See §7.6 for the current governance posture.

---

## A Note on the Name

GAAIM expands to *Generally Accepted AI Metrics*, but the standard's subject is **AI-augmented work**: human labor amplified by AI, not AI output in isolation. The name carries the same linguistic fossil as "horsepower" — the original horses have been gone from the measurement for over a century, but the unit persists because it was the right name at the moment the category crystallized. GAAIM measures what humans and AI produce together; the attribution model (§3.5) is where that joint authorship is made explicit in the wire format. Every event in a GAAIM ledger is, by construction, a record of collaboration.

The short form is the one that should appear in specifications, code, and documentation. The long form ("Generally Accepted AI-augmented work Metrics") is the semantic gloss that clarifies scope when the bare acronym is ambiguous.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Architecture: Core + Profiles](#2-architecture-core--profiles)
3. [Core Event Model](#3-core-event-model)
4. [Canonical Serialization](#4-canonical-serialization)
5. [Signing & Provenance](#5-signing--provenance)
6. [Core Event Taxonomy](#6-core-event-taxonomy)
7. [Profile Architecture](#7-profile-architecture)
8. [Compliance Levels](#8-compliance-levels)
9. [Engineering Profile v0.1-draft](#9-engineering-profile-v01-draft)
10. [Security Considerations](#10-security-considerations)
11. [IANA Considerations](#11-iana-considerations)
12. [References](#12-references)

**Appendices:** A. Worked Examples · B. JSON Schemas (normative) · C. Profile Author Guide · D. Glossary · E. Change Log

---

# 1. Introduction

## 1.1 What GAAIM Is

GAAIM is a specification for capturing knowledge work — the things humans and AI systems produce together — as **signed, attributed, structured events** that are portable across tools, verifiable after the fact, and aggregable into defensible claims about who did what.

It is four things in one document:

1. **An event envelope.** A CloudEvents 1.0 profile that adds the fields knowledge-work evidence requires but which generic event formats do not carry: attribution, provenance linkage, expertise tags, and compliance metadata.
2. **An attribution model.** A first-class representation of *who* contributed to each event — humans, AI models, tools, services — along with their role (proposer, reviewer, approver, executor, observer), confidence, and provenance pointers.
3. **A signing protocol.** An Ed25519-based origin-signing scheme using the JSON Canonicalization Scheme (RFC 8785), a public key registry format, and a chained-hash construction that links events into tamper-evident sequences.
4. **A compliance and conformance framework.** Four compliance levels (L0 Compatible through L3 Processor) that describe what an implementation does with events, declared independently from which vertical profiles it supports.

An implementation that emits GAAIM events produces a record that any GAAIM-aware consumer — an auditor's tool, a regulator's reader, a customer's data warehouse, a competing analytics platform — can verify, parse, and reason about without needing to trust the emitter's internal systems.

## 1.2 What GAAIM Is Not

GAAIM is deliberately scoped. It is not:

- **An identity system.** GAAIM events reference principals by stable identifiers and carry signatures over public keys, but GAAIM does not specify how those identifiers are issued or how keys are initially trusted. Implementations MAY use OIDC, Entra ID, DID-based identity, or any other identity substrate. The specification is compatible with all of them.
- **A storage format.** GAAIM defines the wire format for events in flight and a canonical serialization for signing. How events are stored at rest (relational, columnar, object storage, event-sourcing engine) is an implementation concern.
- **A transport protocol.** GAAIM events MAY be delivered over HTTPS, Kafka, NATS, AMQP, WebSocket, or any other channel. CloudEvents transport bindings apply unchanged.
- **A metric definition catalog.** GAAIM events carry structured data from which metrics are derived; GAAIM does not mandate what those metrics should be or how they should be computed. Profile-level benchmark metrics (§7.1) are optional extensions, not core requirements.
- **A replacement for CloudEvents, Sigstore, in-toto, OpenTelemetry, or SPIFFE.** GAAIM builds on these standards where they fit and references them normatively.

## 1.3 Requirements Language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [BCP 14](https://www.rfc-editor.org/info/bcp14) (RFC 2119, RFC 8174) when, and only when, they appear in all capitals, as shown here.

## 1.4 Document Conventions

- **Code fences** containing JSON represent GAAIM events or fragments thereof. Line breaks and indentation inside code fences are for readability; canonical serialization (§4) removes them.
- **Normative tables** (marked as such in the caption) define required fields and their semantics. Tables without normative markings are explanatory.
- **Event type identifiers** follow reverse-DNS notation rooted at `gaaim.` (e.g., `gaaim.core.artifact.created`, `gaaim.eng.commit.authored`). The `gaaim.` prefix is reserved.
- **Wire-format field names** are case-sensitive and match the CloudEvents convention of lowercase alphanumeric (`gaaimversion`, `attribution`, `prev`). Profile payloads MAY use their own naming conventions inside the `data` object.
- **Version identifiers** follow semantic versioning (MAJOR.MINOR.PATCH). The `gaaimversion` field on events carries the Core version; profile versions are declared separately in the `profile` attribute (§3.3).

---

# 2. Architecture: Core + Profiles

## 2.1 Precedent

GAAIM is published as a single standard with a **Core specification** plus versioned **Profiles** that layer vertical-specific event catalogs, attribution roles, and expertise vocabularies on top. There is one standard, not a family of competing standards. Engineering is the first profile — not a distinct specification that gets superseded later.

This follows the governance pattern that has produced the healthiest cross-vertical standards in adjacent domains:

- **HL7 FHIR** publishes one Core resource model. US Core, International Patient Summary, AU Core, and specialty profiles (Genomics, Radiology, Oncology) layer on top. Implementers build against the Core; jurisdictions and specialties evolve their profiles independently.
- **OpenID Connect** publishes one Core specification. Basic, Implicit, Hybrid, and FAPI are deployment profiles. The core identity contract is stable; profiles adapt it to operating environments.
- **CloudEvents** — GAAIM's own transport layer — publishes one Core event model. HTTP, Kafka, AMQP, and NATS are transport bindings, not parallel standards.
- **SOC 2** publishes one Trust Services Criteria framework. Security, Availability, Processing Integrity, Confidentiality, and Privacy are selectable categories per engagement, not separate audit standards.

The pattern wins because it gives adopters a stable core to implement against while letting vertical communities evolve domain vocabularies at their own pace. A tool built against GAAIM Core can emit signed, attributed, audit-defensible events before any profile exists; profile conformance is layered on as the tool gains vertical-specific capabilities.

## 2.2 The Two Dimensions

A GAAIM implementation is characterized by two orthogonal dimensions:

**Compliance Level** describes *what the implementation does with events*. Levels are cumulative: L3 requires L2, which requires L1, which requires L0. They are defined formally in §8:

- **L0 — Development.** Emits GAAIM-structured events without signatures. Not audit-eligible.
- **L1 — Emitter.** Emits signed, attributed events conforming to Core. Audit-eligible.
- **L2 — Sink.** L1, plus receives events from other emitters, stores them, preserves signatures, and exposes a standard read API.
- **L3 — Processor.** L2, plus runs enrichment, correlation, or inference while preserving the provenance chain.

**Profile Conformance** describes *which vertical vocabularies the implementation correctly handles*. An implementation declares conformance for each profile it supports (Engineering, Legal, Healthcare, Financial, Consulting, and so on). Profile conformance is independent of compliance level.

A certification is therefore stated as **level + profiles**:

| Example Implementation                       | Certification                | Meaning                                            |
| -------------------------------------------- | ---------------------------- | -------------------------------------------------- |
| An IDE adapter that signs engineering events | GAAIM L1 + eng               | Emits signed engineering events                    |
| A legal-ops platform that signs legal events | GAAIM L1 + legal             | Emits signed legal events                          |
| A customer data warehouse                    | GAAIM L2 + eng               | Sinks and stores engineering events; no enrichment |
| A full analytics platform                    | GAAIM L3 + eng, legal, fin   | Processes three profiles                           |
| An audit tool                                | GAAIM L3 + eng, fin, consult | Processes three profiles for its practice areas    |

This separation lets procurement specify what matters to each buyer. An enterprise can require that "all AI-augmented work tools in our supply chain must be GAAIM L1 Certified for the Legal Profile by end of FY." A vendor can market clearly: "GAAIM L1 + Engineering certified." An auditor can scope an engagement: "We accept GAAIM L3 + Engineering, Consulting tools as evidence sources."

## 2.3 Namespace Model

GAAIM event types use reverse-DNS notation rooted at the reserved `gaaim.` prefix:

| Namespace          | Scope                                  | Example Event Types                                          |
| ------------------ | -------------------------------------- | ------------------------------------------------------------ |
| `gaaim.core.*`     | Vertical-neutral infrastructure events | `gaaim.core.artifact.created`, `gaaim.core.decision.logged`, `gaaim.core.session.started` |
| `gaaim.eng.*`      | Engineering Profile events             | `gaaim.eng.commit.authored`, `gaaim.eng.review.completed`    |
| `gaaim.legal.*`    | Legal Profile events                   | `gaaim.legal.motion.filed`, `gaaim.legal.brief.reviewed`     |
| `gaaim.health.*`   | Healthcare Profile events              | `gaaim.health.note.signed`, `gaaim.health.order.placed`      |
| `gaaim.fin.*`      | Financial Services Profile events      | `gaaim.fin.memo.authored`, `gaaim.fin.model.validated`       |
| `gaaim.consult.*`  | Consulting Profile events              | `gaaim.consult.deliverable.submitted`                        |
| `gaaim.{tenant}.*` | Tenant-private extensions              | `gaaim.acme.deal.approved`                                   |

The `gaaim.` prefix is reserved. The `gaaim.core.*` and `gaaim.{profile}.*` namespaces are governed by the publisher (currently Edson Technologies) or a future stewardship body to which governance is contributed. Tenant namespaces (`gaaim.{tenant}.*`) require no registration and are outside the scope of profile certification.

Tenant-private event types that achieve cross-tenant adoption MAY be candidates for promotion to a profile namespace under a future stewardship body. The envisioned promotion path is documented in Appendix C for reference.

**Core events and profile events interoperate at the envelope level.** A consumer that speaks only GAAIM Core can ingest events from any profile as signed, attributed, correlatable records — it will simply not know the profile-specific semantics of the payload. This property is essential: it means a sink or processor can be Core-conformant today and gain profile conformance incrementally as vertical event catalogs are added.

---

# 3. Core Event Model

## 3.1 Base Format

All GAAIM events are **CloudEvents 1.0** [RFC-CE10] events with GAAIM-specific extension attributes. The canonical wire representation is CloudEvents **structured-mode JSON**: the event and its data are encoded as a single JSON object, with event attributes as top-level members.

Implementations MAY transport GAAIM events over any CloudEvents-supported binding (HTTP, Kafka, AMQP, NATS, WebSocket, MQTT). The signing protocol (§5) operates on the structured-mode JSON representation regardless of transport. A binary-mode GAAIM event (CloudEvents attributes in transport headers with the data payload as the body) MUST be converted to structured mode for signature verification.

An illustrative GAAIM event — a decision record emitted by an IDE plugin that orchestrated an AI-assisted architectural choice — is shown below. Every field and its semantics are specified in the sections that follow.

```json
{
  "specversion": "1.0",
  "id": "evt_01HQ5P3KJ6X8W2YQGMZB9N4T7R",
  "source": "ide-plugin://example.org/ide-adapter/instance-42",
  "type": "gaaim.core.decision.logged",
  "subject": "workitem/1247",
  "time": "2026-04-04T21:30:15.123Z",
  "datacontenttype": "application/json",
  "dataschema": "https://gaaim.org/schemas/v1/core/decision-logged.json",

  "gaaimversion": "1.0",
  "profile": "core",
  "tenantid": "acme-engineering",
  "traceid": "01HQ5P3KJ6X8W2YQGMZB9N4T7R",
  "correlationid": "session_01HQ5P3KJ0000000000000000",
  "causationid": "evt_01HQ5P3KJ6X8W2YQGMZB9N4T7P",
  "prev": "evt_01HQ5P3KJ6X8W2YQGMZB9N4T7P",
  "signature": "ed25519:MEUCIQDx...",
  "signaturekey": "ide-adapter-instance-42-2026q2",

  "data": {
    "decisionType": "Architecture",
    "summary": "Chose Ed25519 over RSA for event signing.",
    "rationale": "Smaller keys, smaller signatures, constant-time implementation, FIPS 186-5 approved.",
    "alternatives": [
      {"option": "RSA-2048", "reasonRejected": "Larger signatures (256B vs 64B); parameter-choice risk."}
    ],
    "impactScope": 4,
    "expertiseDomains": ["crypto", "standards"],
    "attributions": [
      {
        "principalId": "ai:example:model-v1",
        "principalType": "AIModel",
        "role": "proposer",
        "confidence": 0.95
      },
      {
        "principalId": "user:alice@acme.example",
        "principalType": "Human",
        "role": "approver",
        "confidence": 1.0
      }
    ],
    "references": {
      "sessionId": "session_01HQ5P3KJ0000000000000000",
      "workItemId": 1247
    },
    "tags": ["crypto", "signing", "architecture"]
  }
}
```

## 3.2 Required CloudEvents Context Attributes

GAAIM constrains CloudEvents' optional attributes more tightly than CloudEvents itself. The following table is **normative**.

| Attribute         | CloudEvents | GAAIM    | Semantics                                                    |
| ----------------- | ----------- | -------- | ------------------------------------------------------------ |
| `specversion`     | REQUIRED    | REQUIRED | MUST be the string `"1.0"`                                   |
| `id`              | REQUIRED    | REQUIRED | MUST be a ULID (§3.6). MUST be globally unique per producer  |
| `source`          | REQUIRED    | REQUIRED | URI identifying the producer (see §3.2.1)                    |
| `type`            | REQUIRED    | REQUIRED | Reverse-DNS event type rooted at `gaaim.` (§2.3)             |
| `subject`         | OPTIONAL    | REQUIRED | Logical entity the event concerns (see §3.2.2)               |
| `time`            | OPTIONAL    | REQUIRED | RFC 3339 timestamp in UTC with millisecond precision         |
| `datacontenttype` | OPTIONAL    | REQUIRED | MUST be `"application/json"` for structured-mode events      |
| `dataschema`      | OPTIONAL    | REQUIRED | URI resolving to the JSON Schema document for this event type |

### 3.2.1 The `source` Attribute

`source` MUST be a URI that uniquely identifies the producing instance. Producers SHOULD use a URI scheme that reflects their tool category, followed by an authority identifying the instance:

- `ide-plugin://vendor.example.org/product/instance-id`
- `ci://example.org/build-runner/runner-007`
- `sdk://example.org/lib-name@1.2.3/process-uuid`
- `webhook://example.org/receiver/endpoint-id`
- `agent://example.org/agent-runtime/session-id`

A receiver MUST be able to distinguish two producers of the same type at the same organization (for example, two instances of the same IDE plugin running on different developer workstations) by their `source` alone. `source` MUST NOT encode secret or sensitive information.

### 3.2.2 The `subject` Attribute

`subject` MUST be a string identifying the logical entity the event concerns, scoped relative to the producer. It is an opaque identifier to the transport layer but has profile-specific meaning. Typical values include `workitem/1247`, `commit/a1b2c3d4`, `document/acme-brief-v3`, `patient/anon-7281` (profile-dependent and subject to PII handling rules in §10.3).

## 3.3 GAAIM Extension Attributes

The following extension attributes are defined by GAAIM. Field names are lowercase alphanumeric per the CloudEvents naming convention. Column "Req." indicates whether the attribute is REQUIRED, RECOMMENDED, or OPTIONAL.

| Attribute       | Req.            | Type                    | Semantics                                                    |
| --------------- | --------------- | ----------------------- | ------------------------------------------------------------ |
| `gaaimversion`  | REQUIRED        | String (semver)         | Core version of this event's envelope. Current value: `"1.0"` |
| `profile`       | REQUIRED        | String                  | Profile the event belongs to: `"core"` or a registered profile slug (`"eng"`, `"legal"`, etc.). MUST match the second dotted segment of `type` |
| `tenantid`      | REQUIRED        | String                  | Tenant identifier. Drives isolation on any multi-tenant sink. MUST NOT leak across tenants |
| `auditeligible` | REQUIRED        | Boolean                 | `true` for L1+ events, `false` for L0 events. Consumers MUST NOT present `auditeligible: false` events as audit evidence |
| `traceid`       | OPTIONAL        | String                  | W3C Trace Context trace-id for cross-service correlation     |
| `correlationid` | OPTIONAL        | String                  | Groups related events at the workflow level (e.g., all events of a single session) |
| `causationid`   | OPTIONAL        | String (ULID)           | The `id` of the event that directly caused this event. MAY reference an event from a different producer. Not part of the chain hash (§5.6) |
| `prev`          | REQUIRED at L1+ | String (ULID) or `null` | The `id` of the immediately preceding event from the same producer. The first event in a producer's chain MUST emit `"prev": null` explicitly. Drives the signing chain (§5.6) |
| `signature`     | REQUIRED at L1+ | String                  | Base64url-encoded Ed25519 signature, prefixed with `"ed25519:"` |
| `signaturekey`  | REQUIRED at L1+ | String (HTTPS URI)      | URI-qualified key identifier resolvable via the Key Registry HTTP API (§5.3.2.1) |

**L0 events** MUST emit `"auditeligible": false` and MUST omit `signature` and `signaturekey`. L0 events MAY omit `prev`. **L1 and higher events** MUST emit `"auditeligible": true`, MUST populate `signature` and `signaturekey`, and MUST populate `prev` (using `null` only for the first event in a producer's chain).

Consumers MUST reject any event bearing an invalid or unverifiable signature. Consumers MUST reject any event where `auditeligible: true` and `prev` is absent. Consumers MUST NOT reject an L0 event solely because it lacks a signature, but MUST NOT treat it as evidence.

**Additional extension attributes** beyond those defined here MAY be added by producers, subject to CloudEvents extension attribute naming rules (lowercase alphanumeric). Attributes prefixed with `gaaim` are reserved for future versions of this specification. Implementations SHOULD namespace custom extensions with a short vendor prefix (for example, `x_acmebuild_runner`).

## 3.4 The `data` Payload Contract

The `data` member of a GAAIM event is a JSON object. Its shape is determined by the event's `type` and validated against the JSON Schema identified by `dataschema`.

Every GAAIM `data` payload MUST include an `attributions` array (§3.5) except for the narrow set of administrative core events enumerated in §6.2.

Every GAAIM `data` payload MAY include:

- A `references` object mapping named roles to entity identifiers (for example, `{"sessionId": "...", "workItemId": 1247}`). Keys are profile-specific; values are strings or integers.
- A `tags` array of free-form short strings (maximum 64 characters each, maximum 32 entries per event) for flexible search, filter, and rollup.

Event-type-specific fields are declared by the JSON Schema at `dataschema`. The Core schema registry (§6.3) hosts all normative Core schemas under `https://gaaim.org/schemas/v1/core/`.

## 3.5 The Attribution Model

The attribution model is the distinguishing contribution of GAAIM. Every non-administrative event declares, in its `data.attributions` array, the principals that participated in producing the event, the role each played, and a confidence score for the attribution claim itself.

### 3.5.1 Attribution Object Schema

```json
{
  "principalId": "ai:anthropic:claude-opus-4-6",
  "principalType": "AIModel",
  "role": "proposer",
  "confidence": 0.95,
  "context": {
    "tokenInput": 2341,
    "tokenOutput": 487,
    "latencyMs": 3420
  }
}
```

**Required fields:** `principalId`, `principalType`, `role`, `confidence`.
**Optional field:** `context` (principal-type-specific metadata; see §3.5.4).

### 3.5.2 Principal Types (Normative)

| Value      | Meaning                                                      |
| ---------- | ------------------------------------------------------------ |
| `Human`    | An identified natural person                                 |
| `AIModel`  | A specific AI model version (e.g., a language model, image model, code model) |
| `Tool`     | A software tool that acted on its own behalf (linter, formatter, transpiler) |
| `Workflow` | An orchestrated multi-step automation                        |
| `Service`  | A backend service acting via service credentials             |
| `System`   | The hosting platform itself (e.g., a scheduled job, automated retention) |
| `External` | An identified third party outside the tenant's boundary      |

### 3.5.3 Roles (Normative Base Set)

| Value       | Meaning                                                      |
| ----------- | ------------------------------------------------------------ |
| `proposer`  | Authored the first draft or initial suggestion               |
| `reviewer`  | Evaluated but did not commit                                 |
| `approver`  | Committed the action; accepts responsibility                 |
| `executor`  | Performed the mechanical operation (applied the patch, sent the message) |
| `observer`  | Was present but did not contribute                           |
| `delegator` | Assigned the task to another principal                       |
| `verifier`  | Checked after completion                                     |

Profiles MAY extend this set with vertical-specific roles (e.g., `attending_physician` in the Healthcare Profile, `signing_attorney` in the Legal Profile). Profile-defined roles MUST NOT collide with base-set names.

### 3.5.4 The `confidence` Field

`confidence` is a floating-point value in the closed interval [0.0, 1.0]. It represents confidence in the attribution claim itself — not the quality of the work product.

- A human who directly performs an approval: `1.0` (deterministic).
- An AI model whose participation is inferred from surrounding context rather than directly recorded: `0.5` – `0.9`.
- A tool whose invocation is mechanically logged: `1.0`.

Consumers computing aggregate metrics SHOULD weight attributions by confidence.

### 3.5.5 Principal Identifier Format

`principalId` is a string with a type-specific URI-like format:

- Humans: `user:<email-or-stable-id>` (e.g., `user:alice@acme.example`)
- AI models: `ai:<vendor>:<model-name-and-version>` (e.g., `ai:anthropic:claude-opus-4-6`)
- Tools: `tool:<name>:<version>` (e.g., `tool:eslint:9.14.0`)
- Services: `svc:<service-uri>` (e.g., `svc://billing.acme.example`)

Identifiers MUST be stable: an AI model version SHOULD NOT be rewritten across events, a user identifier SHOULD NOT change when a person's display name changes. Identifiers MAY contain personally identifying information and are subject to the PII handling rules in §10.3.

### 3.5.6 Principal Identifier Forms

GAAIM supports three forms of principal identifier, allowing producers to balance attribution detail against privacy requirements. Every `principalId` MUST conform to exactly one of the forms below.

**Full form.** Identifies the principal directly and unambiguously.

- Humans: `user:<email-or-stable-id>` (e.g., `user:alice@acme.example`)
- AI models: `ai:<vendor>:<model-name-and-version>` (e.g., `ai:anthropic:claude-opus-4-6`)
- Tools: `tool:<n>:<version>` (e.g., `tool:eslint:9.14.0`)
- Services: `svc:<service-uri>` (e.g., `svc://billing.acme.example`)

Full form is the default and MUST be used unless the producer has declared an alternative form in the Key Registry.

**Vendor-only form.** Identifies the principal's vendor or organization but redacts the specific identity.

- Humans: `user:<tenant-slug>:*` (e.g., `user:acme:*`)
- AI models: `ai:<vendor>:*` (e.g., `ai:anthropic:*`)
- Tools: `tool:<vendor>:*` (e.g., `tool:jetbrains:*`)
- Services: `svc:<domain>:*` (e.g., `svc:acme.example:*`)

Vendor-only form lets a producer attribute work to a class of principals without revealing individual identities. Useful for aggregate analytics, cross-tenant benchmarks, and AI labs that prefer not to expose per-model-version usage at the wire level.

**Anonymized form.** Uses a deterministic one-way hash of the full identifier.

- Format: `<type>:anon:<sha256-of-full-id-hex>`
- Humans: `user:anon:5e884898da28047151d0e56f8dc6292773603d0d...`
- AI models: `ai:anon:7d793037a0760186574b0282f2f435e7...`
- Tools: `tool:anon:<hash>`
- Services: `svc:anon:<hash>`

The hash is SHA-256 over the UTF-8 bytes of the full identifier, hex-encoded (lowercase, 64 characters). The same full identifier always hashes to the same anonymized form, enabling correlation across events without exposing the underlying identity.

Anonymized form is one-way. A verifier holding only the anonymized form cannot recover the full identifier. A verifier holding both forms can verify correspondence by hashing the full form and comparing.

**Form declaration.** A producer MUST declare which principal-identifier form it emits, by principal type, in its Key Registry record (§5.3.2.2). The record includes a `principalForms` field:

```json
{
  "principalForms": {
    "Human": "full",
    "AIModel": "vendor-only",
    "Tool": "full",
    "Service": "full"
  }
}
```

Consumers MUST accept all three forms from any producer. Sinks MUST NOT reject events solely because a principal is in anonymized or vendor-only form.

**Auditor requirements.** Auditors who require full-form attribution MAY contractually require producers to emit full form, or to provide a side-channel mapping from anonymized to full forms for audit scope. GAAIM does not mandate full-form emission at the wire level; this is a contractual matter between producer and auditor.

**Downgrade prohibition.** A producer that has emitted events in full form MUST NOT retroactively re-emit those events in anonymized form under the same `signaturekey`. Anonymization is a forward-only choice at the time of emission. Historical events remain in the form they were originally signed.

## 3.6 Event Identifiers

Event `id` values MUST be **time-ordered 128-bit identifiers**. Conformant producers MUST use one of the following formats:

- **ULID** [ULID-SPEC]: 128-bit, lexicographically sortable, Crockford-Base32-encoded (26 characters). ULIDs encode 48 bits of millisecond timestamp followed by 80 bits of cryptographic randomness, producing strings such as `01HQ5P3KJ6X8W2YQGMZB9N4T7R`.
- **UUIDv7** [RFC-9562 §5.7]: 128-bit time-ordered UUID, encoded as a 36-character dashed hex string such as `01897f48-fc1d-7890-abcd-ef0123456789`.

Both formats provide the time-prefix property needed for efficient range queries and chain traversal. Producers SHOULD choose one format consistently per `source`; mixing within a single producer's event stream is permitted but complicates id-based ordering.

Sinks MUST accept both formats. Query APIs (§8.4) MUST treat both formats as valid values for `id`, `prev`, and `causationid` fields.

The `id` field MUST be unique per producer. A collision, while statistically improbable, MUST be treated by producers as a fatal error: the event MUST be regenerated with a new id rather than silently retransmitted.

**Format choice guidance (informative):**

- Choose **UUIDv7** if your platform already has UUID infrastructure, your sink is built on a relational database with native UUID types, or you need RFC-standardized identifier generation.
- Choose **ULID** if you want the most compact textual representation, your primary storage is columnar or key-value, or you prefer Crockford-Base32's human-readable property (no ambiguous characters like `O`/`0`).

## 3.7 Event Versioning

GAAIM Core uses semantic versioning. Events declare their envelope version via `gaaimversion`:

- **PATCH increment** (1.0 → 1.0.1): editorial, documentation, clarification. No wire-format change.
- **MINOR increment** (1.0 → 1.1): added OPTIONAL attributes, added core event types, added principal types or roles. Consumers on version 1.0 MUST continue to parse 1.1 events successfully, ignoring unknown optional attributes.
- **MAJOR increment** (1.0 → 2.0): breaking change. Consumers MUST be upgraded.

The steward commits to maintaining MAJOR-version support for **at least five years** after a successor MAJOR version ships. Producers MUST NOT be forced to re-emit historical events in a new envelope version; sinks MUST accept historical events in the envelope version under which they were signed.

Profile versions are independent of Core versions. A producer MAY emit events where `gaaimversion="1.0"` and the profile (indicated by the `profile` attribute) is at version `2.3`.

---

# 4. Canonical Serialization

## 4.1 Why Canonicalization Is Required

A signature is a mathematical function of bytes. Two JSON encodings of the same logical event — one with keys in one order, one with whitespace, one with a different number representation — produce different byte sequences and therefore different signatures. Verifying a signature requires that the verifier reconstructs the exact byte sequence the signer signed.

GAAIM specifies a single canonical serialization so that any conforming producer and any conforming verifier agree, bit-for-bit, on what was signed.

## 4.2 Canonical JSON Rules

GAAIM canonical serialization follows the **JSON Canonicalization Scheme** [RFC-8785] with two GAAIM-specific exclusions.

The canonicalization procedure is:

1. **Remove signature attributes.** The `signature` and `signaturekey` top-level attributes MUST be removed from the event object before canonicalization. These are the attributes being computed; including them would be circular.
2. **Apply JCS (RFC 8785)** to the resulting object:
   - Object keys are sorted lexicographically by UTF-16 code unit, recursively.
   - No insignificant whitespace is emitted.
   - Numbers are serialized per ECMAScript's `ToString(Number)` as specified in RFC 8785 §3.2.2.3.
   - Strings are serialized as UTF-8 bytes with minimal JSON escaping per RFC 8785 §3.2.2.2.
   - The encoding is UTF-8 without a byte-order mark.
3. **The resulting byte sequence** is the canonical form signed in §5.

### 4.2.1 Absent Attributes (Normative)

Canonical serialization is applied to the event object **as emitted**. Attributes absent from the source object MUST NOT be added during canonicalization, regardless of whether they are REQUIRED, RECOMMENDED, or OPTIONAL in §3.3. An implementation that wishes an attribute to appear in the canonical form MUST include it explicitly in the source object, with an explicit value (including `null` where that is the intended value).

This rule exists to prevent the most common source of cross-implementation signature mismatches: one producer omitting an optional attribute while another inserts it with a default or null value. Two implementations applying this rule to identical source objects MUST produce bit-identical canonical byte sequences.

### 4.2.2 Number Serialization Note (Informative)

ECMAScript's `ToString(Number)` emits `1` for the value `1.0`, `0.92` for `0.92`, and `2847` for `2847`. Implementations MUST NOT preserve trailing zeros or decimal points from the source JSON. Producers that need to distinguish integer from fractional semantics MUST encode that distinction elsewhere (typically in a schema-defined string field), not in the numeric representation.

### 4.2.3 Test Vectors

Normative test vectors are published at `https://gaaim.org/conformance/canonical-vectors-v0.1.json` and mirrored in Appendix A.3. Every conformant implementation MUST produce canonical bytes whose SHA-256 matches the published vectors when given the source objects in the vector file. A test-vector pass is REQUIRED for conformance self-attestation at L1 or higher.

## 4.3 What Gets Signed, What Doesn't

| Attribute / Member                                           | In Canonical Form?             |
| ------------------------------------------------------------ | ------------------------------ |
| All CloudEvents context attributes (`specversion`, `id`, `source`, `type`, `subject`, `time`, `datacontenttype`, `dataschema`) | Yes                            |
| `gaaimversion`, `profile`, `tenantid`, `traceid`, `correlationid`, `causationid`, `prev` | Yes                            |
| `data` (including `attributions`, `references`, `tags`, payload fields) | Yes                            |
| `signature`, `signaturekey`                                  | **No** (excluded)              |
| Transport-layer metadata (HTTP headers, Kafka headers)       | **No** (not part of the event) |

Implementations MUST canonicalize the entire event minus the two excluded attributes; they MUST NOT include transport metadata in the canonical form.

## 4.4 Canonicalization Walkthrough

Given the following non-canonical event fragment:

```json
{
  "type": "gaaim.core.artifact.created",
  "id": "evt_01HQ...",
  "specversion": "1.0",
  "data": { "size": 1024, "name": "diagram.svg" }
}
```

The canonical form (after sorting keys, removing whitespace, excluding signature attributes) is:

```
{"data":{"name":"diagram.svg","size":1024},"id":"evt_01HQ...","specversion":"1.0","type":"gaaim.core.artifact.created"}
```

Full walkthroughs with signature computation against known test vectors are provided in **Appendix A.3**. Reference implementations in TypeScript, C#, Python, Go, and Rust are published under Apache 2.0 at `https://github.com/GAAIM-standard/canonicalize`.

---

# 5. Signing & Provenance

## 5.1 Why Signing Matters

A GAAIM event without a signature is a claim. A GAAIM event with a valid signature from a registered key is **evidence** — it attests that the named producer, holding the registered private key, emitted exactly these bytes at origin. Origin signing moves trust out of the storage layer: a sink, a forwarder, or an analytics platform may be compromised without invalidating historical events, because the signatures verify against keys that were registered before the events were created.

Three use cases make origin signing non-negotiable:

- **Audit defense.** A regulator, tax authority, or acquirer examines claims about who produced what work. Signatures let the examiner verify each event against a key registered independently of the claimant's storage.
- **Cross-organization data exchange.** A customer, auditor, or partner ingests events from another organization. Signatures let them verify without having to trust the sender's infrastructure.
- **Dispute resolution.** A contested attribution (for example, "this output was AI-generated without human review") is resolved by showing the signed attribution chain from the moment work was performed.

In all three cases, unsigned data is a self-attestation. Signed data is verifiable evidence.

## 5.2 Signature Algorithms

### 5.2.1 Algorithm Registry

GAAIM maintains a registered algorithm list governing which signature algorithms MAY appear in the `signature` attribute. Algorithms are identified by a lowercase prefix followed by a colon (for example, `"ed25519:"`). The registry is published at `https://gaaim.org/registry/signing-algorithms` and is versioned with this specification.

New algorithms are added to the registry through the publisher's revision process (or, if governance is contributed to a neutral body, through that body's process). Adding a new algorithm is a MINOR version change to GAAIM Core: verifiers that do not support the new algorithm continue to function correctly on events signed with previously registered algorithms, and MAY be upgraded on their own schedule to support new entries.

### 5.2.2 Mandatory-to-Implement Algorithm

As of GAAIM Core v0.1-draft, the mandatory-to-implement (MTI) signature algorithm is **Ed25519** (EdDSA over Curve25519) [RFC-8032], identified by the `"ed25519:"` prefix.

Every conformant verifier MUST support Ed25519 verification. Every conformant producer MUST be capable of signing with Ed25519. Producers MAY additionally sign with any other registered algorithm; verifiers MAY additionally verify any other registered algorithm. A producer that signs only with a non-MTI algorithm risks producing events that conformant verifiers cannot verify.

Ed25519 is the MTI for v0.1-draft because of its deterministic signatures, constant-time reference implementations, absence of per-signature parameter choices, standardization (NIST FIPS 186-5, 2023), and broad library support across all major implementation languages. The MTI choice is expected to evolve: GAAIM targets environments with modern cryptographic infrastructure (TLS 1.2+, hardware-accelerated signature verification, post-quantum library availability), and the MTI designation will be revisited as the algorithm registry grows.

### 5.2.3 Algorithm Prefix and Identification

The `signature` attribute is structured as `<algorithm-prefix>:<base64url-encoded-bytes>`. The prefix identifies which registered algorithm to invoke for verification. Verifiers MUST:

- Reject the event if the prefix is absent or malformed.
- Reject the event if the prefix is not present in the registered algorithm list the verifier supports.
- Reject the event if the algorithm prefix does not match the key's declared `algorithm` field in the Key Registry (prevents prefix-swap downgrade attacks).

### 5.2.4 Post-Quantum Readiness

Ed25519, like all pre-quantum elliptic-curve signature schemes, is vulnerable to sufficiently powerful quantum computers executing Shor's algorithm. NIST has standardized post-quantum signature algorithms — **ML-DSA** (FIPS 204), **SLH-DSA** (FIPS 205), and **FN-DSA** (FIPS 206, 2024) — which are expected to enter the GAAIM algorithm registry in a future MINOR version once ecosystem maturity and cross-implementation interop support it.

Implementations SHOULD design their signing and verification paths to support algorithm negotiation from the outset, reducing migration cost when post-quantum entries are added to the registry. Producers operating under long-horizon integrity requirements (multi-decade audit obligations, regulatory archives, tax-credit substantiation with extended examination windows) SHOULD plan key-rotation cadence (§5.4) to accommodate an eventual algorithm migration, and SHOULD consider **hybrid signing** — concurrent classical and post-quantum signatures on the same event — once the registry admits it.

GAAIM v0.1-draft does not specify a migration timeline for post-quantum algorithms. The timing will be driven by ecosystem readiness and library availability across the reference-implementation languages, and by whichever body is responsible for the registry at the time of migration.

### 5.2.5 Migration Planning for Long-Horizon Integrity

Ed25519, the MTI algorithm for v0.1-draft, is expected to remain cryptographically adequate through the ecosystem's current planning horizon. Producers operating under long-horizon integrity requirements — multi-decade audit obligations, regulatory archives, tax-credit substantiation with extended examination windows, healthcare records, legal records requiring 10+ year retention — SHOULD plan for an algorithm transition before any post-quantum adversary becomes practical.

**Reserved prefixes.** The following algorithm prefixes are RESERVED in v0.1-draft for future registration. Producers MUST NOT emit signatures under these prefixes until the corresponding algorithm registry entries are ratified:

- `ml-dsa:` — FIPS 204 (ML-DSA / Dilithium)
- `slh-dsa:` — FIPS 205 (SLH-DSA / SPHINCS+)
- `fn-dsa:` — FIPS 206 (FN-DSA / Falcon)
- `hybrid:` — concurrent classical + post-quantum signatures on a single event

Verifiers MUST reject events bearing reserved-but-unregistered prefixes with error code `algorithm-reserved`.

**Migration plan declaration.** L1+ producers with declared retention obligations exceeding five years MUST populate the `migrationPlan` field in their Key Registry record (§5.3.2.2). The field is a structured object declaring intended algorithm succession:

```json
{
  "migrationPlan": {
    "retentionYears": 10,
    "plannedSuccessorAlgorithm": "hybrid",
    "plannedTransitionBy": "2029-12-31T00:00:00Z",
    "contingency": "rotate-on-registry-entry"
  }
}
```

Field semantics:

- `retentionYears` (integer, ≥ 1) — the producer's declared retention horizon for events signed under this key.
- `plannedSuccessorAlgorithm` (string) — the algorithm prefix the producer intends to migrate to. MAY be a currently-reserved prefix.
- `plannedTransitionBy` (RFC 3339 datetime) — the date by which the producer commits to have rotated to the successor algorithm.
- `contingency` (string enum) — one of `rotate-on-registry-entry` (rotate as soon as the successor algorithm enters the registry), `rotate-on-schedule` (rotate on the declared date regardless), `evaluate-at-transition` (producer will decide at the transition date).

Producers with retention obligations of five years or less MAY omit `migrationPlan` or emit `null`.

**Hybrid algorithm guidance.** A hybrid Ed25519 + ML-DSA entry is an anticipated future addition to the algorithm registry, expected once ML-DSA libraries are stable and audited across the reference-implementation languages (TypeScript, C#, Python, Go, Rust). "Library availability" means a stable, audited, open-source implementation with documented APIs under an OSI-approved license. Timing depends on ecosystem maturity and on decisions made by whichever body is responsible for the registry at that time.

## 5.3 The Signing Protocol

### 5.3.1 Key Generation and Registration

Before emitting signed events, a producer MUST:

1. **Generate an Ed25519 keypair.** The private key is generated and stored exclusively on the producer. It MUST NOT leave the producer. Recommended storage: OS keychain (for client-side tools), HSM or cloud KMS (for server-side producers), per-process memory (for ephemeral workloads with short-lived keys).

2. **Register the public key** with a GAAIM Key Registry (§5.3.2). Registration includes:
   - Key identifier (the `signaturekey` value that will appear in events)
   - Public key bytes (raw 32-byte Ed25519 public key, encoded per §5.3.3)
   - Producer identity (the URI that will appear in the event `source`)
   - Validity window (`valid_from`, `valid_until`)
   - Key metadata (producer type, rotation policy, contact)

3. **Receive confirmation** that the key is registered and retrievable.

Private keys MUST NOT be registered, transmitted, or logged at any point.

### 5.3.2 Key Registry

A **GAAIM Key Registry** is an HTTP-accessible service that serves registered public keys. This specification defines a minimal normative HTTP API that every conformant registry MUST expose. Implementations MAY additionally expose alternative resolution mechanisms (DID documents, Sigstore transparency log, LDAP) but MUST support the baseline HTTP API for cross-implementation portability.

#### 5.3.2.1 URI-Qualified Key Identifiers

The `signaturekey` attribute on an event (§3.3) MUST be an HTTPS URI that resolves, via the HTTP API defined in §5.3.2.2, to the public-key record used to verify the event's signature. Example:

```
https://keys.acme-engineering.example/v1/keys/code-adapter-instance-7-2026q2
```

The URI binds the event to a specific registry instance. A verifier presented with a `signaturekey` that resolves to an unexpected registry origin MUST either reject the event with error code `registry-mismatch` or consult a sink-configured allowlist of acceptable registry origins (§10.4.1).

Opaque identifiers (non-URI strings) are DEPRECATED and MUST NOT be emitted by L1 or higher producers.

#### 5.3.2.2 HTTP API Surface (Normative)

Every conformant registry MUST expose the following endpoints under a versioned path prefix (`/v1` in v0.1-draft). Requests and responses are `application/json`.

**GET /v1/keys/{keyId}**

Resolves a key identifier to its registered record. The `{keyId}` path segment is the trailing segment of the URI-qualified `signaturekey` after the registry's base path.

Response 200 body (normative schema):

```json
{
  "keyId": "code-adapter-instance-7-2026q2",
  "keyUri": "https://keys.acme-engineering.example/v1/keys/code-adapter-instance-7-2026q2",
  "algorithm": "ed25519",
  "publicKey": "spki-base64:MCowBQYDK2VwAyEA...",
  "publicKeyRaw": "raw-base64url:...",
  "producerUri": "ide-plugin://example.org/code-adapter/instance-7",
  "producerType": "IDEPlugin",
  "validFrom": "2026-04-01T00:00:00Z",
  "validUntil": "2026-07-01T00:00:00Z",
  "revokedAt": null,
  "revocationReason": null,
  "tenantId": "acme-engineering",
  "principalForms": {
    "Human": "full",
    "AIModel": "full",
    "Tool": "full",
    "Service": "full"
  },
  "migrationPlan": null
}
```

Required fields: `keyId`, `keyUri`, `algorithm`, `publicKey`, `producerUri`, `validFrom`, `validUntil`. Optional fields: `publicKeyRaw`, `producerType`, `revokedAt`, `revocationReason`, `tenantId`, `principalForms`, `migrationPlan`.

The `keyUri` in the response MUST equal the request URI. A mismatch MUST be treated by the verifier as a resolution failure and the event MUST be rejected with error code `registry-mismatch`.

**Response codes:**

- `200 OK` — key found, record returned
- `404 Not Found` — key identifier not registered; verifier MUST reject the event with error code `unknown-key`
- `410 Gone` — key was registered and is permanently withdrawn; verifier MUST reject with error code `key-withdrawn`
- `5xx` — verifier MAY retry with exponential backoff; SHOULD NOT accept the event until resolution succeeds

**GET /.well-known/gaaim-registry**

Returns a discovery document describing the registry:

```json
{
  "registryVersion": "v1",
  "baseUrl": "https://keys.acme-engineering.example/v1",
  "supportedAlgorithms": ["ed25519"],
  "specVersion": "0.1-draft",
  "operator": {
    "name": "ACME Engineering",
    "contactUri": "mailto:security@acme-engineering.example"
  }
}
```

A verifier encountering a `signaturekey` URI whose origin it has not previously seen MAY fetch `/.well-known/gaaim-registry` on that origin to validate it is a conformant registry before resolving keys. This endpoint MUST be reachable without authentication.

#### 5.3.2.3 Authentication and Rate Limiting

Registry read endpoints MUST be publicly accessible without authentication for cross-organization verification. Registries MAY rate-limit by IP or by `Origin` header. Registries MUST NOT require authentication for verification flows; a registry that gates reads behind authentication is not conformant.

Registry **write** endpoints (key registration, rotation, revocation) are out of scope for this specification and MAY use whatever authentication the operator prefers.

#### 5.3.2.4 Caching

Registry responses SHOULD include standard HTTP cache headers. Verifiers MAY cache key records up to the record's `validUntil` timestamp but MUST re-fetch on any `revokedAt` check. A cached record whose `validUntil` has passed MUST NOT be used for verification; verifiers MUST re-fetch to confirm the record has not been renewed.

#### 5.3.2.5 Federation

A registry MAY delegate resolution of a `keyId` to a downstream registry by responding with HTTP `307 Temporary Redirect` to another `/v1/keys/{keyId}` URL. Verifiers MUST follow up to three redirects and MUST fail resolution with error code `redirect-loop` if redirection exceeds that depth.

### 5.3.3 Public Key Encoding

Public keys in the registry MUST be emitted using prefix-encoded strings. The format is `<encoding>:<encoded-bytes>`, mirroring the signature format of §5.2.3.

**Registered encodings (v0.1-draft):**

| Prefix          | Meaning                                                      | Support  |
| --------------- | ------------------------------------------------------------ | -------- |
| `spki-base64`   | SubjectPublicKeyInfo DER bytes [RFC-5280], base64-encoded (standard base64, not base64url) | REQUIRED |
| `raw-base64url` | Algorithm-specific raw public key bytes (32 bytes for Ed25519), base64url-encoded | OPTIONAL |

Verifiers MUST support `spki-base64`. Verifiers MAY support `raw-base64url`. A registry MAY include both forms in a key record (`publicKey` uses `spki-base64:`, `publicKeyRaw` uses `raw-base64url:`); verifiers SHOULD prefer `publicKey` and fall back to `publicKeyRaw` only if unable to parse SPKI.

New encoding prefixes MAY be added through the publisher's revision process (or through the process of a future stewardship body). Unknown prefixes MUST cause the verifier to reject the key record with error code `unknown-key-encoding`.

### 5.3.4 Per-Event Signing Procedure

For each event, the producer:

1. Constructs the event JSON with all REQUIRED attributes populated except `signature` and `signaturekey`.
2. Applies canonical serialization per §4.2 to produce the canonical byte sequence.
3. Computes `sig = Ed25519-Sign(privateKey, canonicalBytes)` yielding 64 bytes.
4. Encodes the signature as `"ed25519:" + base64url(sig)` and places it in the event's `signature` attribute.
5. Places the registered key identifier in the event's `signaturekey` attribute.
6. Transmits the event via its chosen transport.

### 5.3.5 Per-Event Verification Procedure

On receipt, a verifier:

1. Extracts `signature` and `signaturekey` from the event.
2. Verifies the `signature` prefix is `"ed25519:"`; rejects other prefixes (until later versions register them).
3. Resolves `signaturekey` to a public key via the Key Registry.
4. Verifies the key was valid (between `validFrom` and `validUntil`, not revoked) at the event's `time`.
5. Removes `signature` and `signaturekey` from the event and applies canonical serialization per §4.2.
6. Decodes the base64url signature bytes.
7. Computes `Ed25519-Verify(publicKey, canonicalBytes, sig)`. If false, MUST reject the event as tampered.
8. If valid, MAY proceed with profile-specific validation of the event `data` payload.

## 5.4 Key Rotation

Every registered key MUST declare a `validUntil` timestamp. Keys MUST be rotated at least **annually**; quarterly rotation is RECOMMENDED for production producers.

Rotation procedure:

1. Producer generates a new keypair.
2. Producer registers the new public key with an overlapping validity window (new key's `validFrom` ≤ old key's `validUntil`).
3. Producer begins signing new events with the new key.
4. Old key remains in the registry; historical events continue to verify against it.
5. At the old key's `validUntil`, the key is **archived** — it remains retrievable for verification but cannot be used to sign new events.

Archived keys MUST NOT be deleted. A signature from 2026 MUST remain verifiable in 2046. Implementations MAY move archived keys to cold storage but MUST preserve retrieval.

## 5.5 Key Revocation

A key MAY be revoked before its `validUntil` when compromise is suspected or the producer is decommissioned. Revocation sets `revokedAt` to a timestamp and `revocationReason` to a human-readable string.

Verifiers encountering an event whose `signaturekey` resolves to a revoked key MUST:

- Accept the event as valid **if** the event's `time` is **strictly before** `revokedAt`.
- Reject the event as invalid **if** the event's `time` is at or after `revokedAt`.

This rule preserves the validity of historical evidence produced before compromise while rejecting events that could have been forged with a leaked key.

## 5.6 Provenance Chains

Individual signatures protect individual events. Related events form **provenance chains** via the `prev` and `causationid` attributes, producing a hash-linked sequence per producer (or per workflow, when workflows are explicitly declared).

### 5.6.1 Chain Construction

Each event MAY carry a `prev` attribute containing the `id` of the immediately preceding event from the same producer (same `source`). When `prev` is present, the event's signed bytes include this reference, cryptographically binding the event to its predecessor.

A **chain hash** is derived for each event as:

```
chainHash(event) = SHA-256( chainHash(prev) || signature(event) )
```

where `chainHash(prev)` is the predecessor's chain hash (or 32 zero bytes if the event has no predecessor). Chain hashes are computed and maintained by sinks; they are not required in the event wire format.

Tampering with any historical event invalidates its signature and propagates to every subsequent chain hash. A verifier walking the chain can localize the earliest point of compromise with single-event precision.

### 5.6.2 Chain Continuity Verification

A verifier traversing a producer's provenance chain MUST treat a missing `prev` as a chain break. Specifically: given events E₁ and E₂ from the same producer with E₁.time < E₂.time, if E₂.prev does not equal E₁.id (or E₂.prev does not chain to E₁ through intermediate events), the verifier MUST flag a **chain gap** between E₁ and E₂.

Chain gaps do not invalidate individual event signatures — each event remains cryptographically verifiable in isolation — but they invalidate claims of *sequence integrity* for audit purposes. Sinks MUST expose chain-gap metadata through their query API (§8.4) so downstream consumers can distinguish complete chains from gapped ones.

The first event emitted under a newly-registered key MUST carry `"prev": null`. The first event after a key rotation MUST carry the `id` of the last event emitted under the prior key as its `prev`, binding the two chains together across the rotation boundary. A sink MAY require producers to emit a `gaaim.core.key.rotated` administrative event (§6.2) at the rotation boundary to make the handoff explicit.

### 5.6.3 Multiple Chains

A single producer MAY maintain multiple concurrent chains (for example, one per session, one per workflow) by partitioning on `correlationid`. The `prev` attribute references the preceding event within the same chain partition. Producers MUST ensure partitioned chains are non-interleaved from the signer's perspective.

### 5.6.4 Cross-Producer Linkage

`causationid` MAY reference an event from a different producer. `causationid` is **not** part of the chain hash — it is a logical pointer used for correlation and displayed in provenance traces, but its predecessor may have been emitted by a different key. Verifiers displaying provenance MUST visually distinguish in-chain references (`prev`) from cross-producer references (`causationid`).

## 5.7 Verification in Practice

A conformant verifier implementation performs, at minimum:

- Per-event signature verification (§5.3.5)
- Key validity and revocation checking (§5.5)
- Chain integrity verification (§5.6) when `prev` is present

Reference verifiers in TypeScript, C#, Python, Go, and Rust are published under Apache 2.0 at `https://github.com/GAAIM-standard/verifier`. A conformance test suite with known-good and known-bad events (tampered payloads, expired keys, broken chains, algorithm mismatches) is published at `https://github.com/GAAIM-standard/conformance-tests`.

## 5.8 Transparency Logs (Deferred)

A transparency-log construction — publishing Merkle roots of hourly event batches to an append-only, publicly auditable log in the manner of Certificate Transparency — is a natural extension of origin signing. GAAIM v0.1-draft does not specify transparency-log publication; it is deferred to a later MINOR version after the base signing protocol achieves adoption. Implementations MAY publish transparency logs on their own terms in the interim.

---

# 6. Core Event Taxonomy

## 6.1 Naming Convention

Core event types follow the pattern:

```
gaaim.core.<entity>.<verb>
```

where `<entity>` is a noun identifying the logical subject of the event and `<verb>` is the past-tense action that occurred. Both segments MUST be lowercase alphanumeric with hyphens permitted inside segments. Additional dotted segments MAY be added for specialization (for example, `gaaim.core.artifact.created.derived`) but SHOULD be avoided in favor of distinguishing event types via the `data` payload.

The same naming rule applies to profile event types, with `core` replaced by the profile slug (§2.3). Profile-defined event types SHOULD follow the Core entity vocabulary where the same logical concept exists, and MUST define distinct entities when the concept is vertical-specific.

## 6.2 Core Event Families

The Core event catalog is organized into seven families. All event types listed here are **normative**: producers that emit events for these logical concepts MUST use the registered type name.

### 6.2.1 Session Lifecycle

A session is a bounded unit of work performed by one or more principals collaborating within a defined time window.

| Event Type                   | Purpose                                                 |
| ---------------------------- | ------------------------------------------------------- |
| `gaaim.core.session.started` | A session begins; declares participants and scope       |
| `gaaim.core.session.ended`   | A session concludes; carries session-level summary data |
| `gaaim.core.session.paused`  | An explicit pause in an ongoing session                 |
| `gaaim.core.session.resumed` | A paused session resumes                                |
| `gaaim.core.session.linked`  | Two sessions are merged into a single logical unit      |

### 6.2.2 Decisions

A decision is a choice among alternatives that affects subsequent work.

| Event Type                     | Purpose                                                      |
| ------------------------------ | ------------------------------------------------------------ |
| `gaaim.core.decision.logged`   | A decision is recorded; carries alternatives, rationale, impact scope |
| `gaaim.core.decision.reviewed` | A logged decision is examined by another principal           |
| `gaaim.core.decision.reversed` | A prior decision is invalidated                              |
| `gaaim.core.decision.ratified` | A decision is formally approved at a governance level        |

### 6.2.3 Artifacts

An artifact is a produced work item: a file, document, data record, diagram, model, or similar tangible output.

| Event Type                      | Purpose                                             |
| ------------------------------- | --------------------------------------------------- |
| `gaaim.core.artifact.created`   | A new artifact is produced                          |
| `gaaim.core.artifact.modified`  | An existing artifact is changed                     |
| `gaaim.core.artifact.reviewed`  | An artifact is examined without modification        |
| `gaaim.core.artifact.published` | An artifact is released beyond its production scope |
| `gaaim.core.artifact.archived`  | An artifact is removed from active use              |

### 6.2.4 Messages and Exchanges

A message is a communication event between principals — human-to-human, human-to-AI, AI-to-AI, or any combination.

| Event Type                      | Purpose                                            |
| ------------------------------- | -------------------------------------------------- |
| `gaaim.core.message.exchanged`  | A communication event between principals           |
| `gaaim.core.message.escalated`  | A message thread is elevated to additional parties |
| `gaaim.core.prompt.submitted`   | A structured prompt is submitted to an AI model    |
| `gaaim.core.response.generated` | An AI model produces a response                    |

### 6.2.5 Tools and Agents

Tool and agent events record mechanical operations — the invocation and result of automated or agentic actions.

| Event Type                   | Purpose                                         |
| ---------------------------- | ----------------------------------------------- |
| `gaaim.core.tool.invoked`    | A tool or function is called                    |
| `gaaim.core.tool.completed`  | A tool invocation returns successfully          |
| `gaaim.core.tool.failed`     | A tool invocation fails                         |
| `gaaim.core.agent.handoff`   | Work is transferred between agents              |
| `gaaim.core.agent.delegated` | A subtask is assigned from one agent to another |

### 6.2.6 Workflow Correlation

Workflow events group related events into multi-step processes that span sessions, principals, or tools.

| Event Type                      | Purpose                            |
| ------------------------------- | ---------------------------------- |
| `gaaim.core.workflow.started`   | A multi-event workflow begins      |
| `gaaim.core.workflow.step`      | A step within a declared workflow  |
| `gaaim.core.workflow.completed` | A workflow ends successfully       |
| `gaaim.core.workflow.abandoned` | A workflow ends without completion |

### 6.2.7 Administrative and Audit

Administrative events record platform-level actions: tenant management, key lifecycle, policy changes, and audit-log access.

| Event Type                        | Purpose                                  |
| --------------------------------- | ---------------------------------------- |
| `gaaim.core.admin.tenant.created` | A new tenant is provisioned              |
| `gaaim.core.admin.user.granted`   | A principal is granted access            |
| `gaaim.core.admin.user.revoked`   | A principal's access is removed          |
| `gaaim.core.admin.key.registered` | A signing key is added to the registry   |
| `gaaim.core.admin.key.rotated`    | A signing key is rotated                 |
| `gaaim.core.admin.key.revoked`    | A signing key is revoked                 |
| `gaaim.core.admin.policy.changed` | A tenant policy is modified              |
| `gaaim.core.audit.data.accessed`  | A read of audit-sensitive data is logged |
| `gaaim.core.audit.data.exported`  | An export of data is performed           |

## 6.3 Events Exempt from Attribution

Administrative events (§6.2.7) MAY omit the `data.attributions` array when the actor is the hosting platform itself (principal type `System`). All other Core events MUST carry attributions. Profile-defined events MUST carry attributions; profiles MUST NOT define attribution-exempt events.

## 6.4 Event Schema Registry

Every event type has a JSON Schema registered at:

```
https://gaaim.org/schemas/v<gaaimversion>/<profile>/<event-type>.json
```

where `<profile>` is `core` or a profile slug. The registry is:

- **Publicly accessible.** Implementations MUST be able to fetch schemas without authentication.
- **Versioned immutably.** Once published, a schema at a given `<gaaimversion>` MUST NOT be altered. Changes require a new version per §3.7.
- **Mirrorable.** The complete registry MUST be retrievable as a Git repository at `https://github.com/GAAIM-standard/schemas` for offline, air-gapped, or long-term archival use.
- **Validated at ingest.** Conformant sinks (L2+) MUST validate received events against the schema identified by `dataschema` before persisting.

## 6.5 Worked Example: A Session Trace

An illustrative session — an AI-assisted authentication refactor — produces the following ordered event trace:

```
t=14:00:00  gaaim.core.session.started
t=14:00:15  gaaim.core.tool.invoked            (read-file)
t=14:00:15  gaaim.core.tool.completed          (read-file)
t=14:00:45  gaaim.core.prompt.submitted
t=14:00:48  gaaim.core.response.generated
t=14:01:20  gaaim.core.decision.logged
t=14:02:10  gaaim.core.artifact.created        (auth-handler source file)
t=14:03:45  gaaim.core.artifact.modified       (wire-up source file)
t=14:05:00  gaaim.core.tool.invoked            (test-runner)
t=14:05:42  gaaim.core.tool.completed          (test-runner)
t=14:06:00  gaaim.core.decision.reviewed
t=14:10:00  gaaim.core.artifact.modified       (auth-handler source file)
t=14:15:30  gaaim.core.tool.invoked            (commit)
t=14:15:31  gaaim.core.tool.completed          (commit)
t=14:16:00  gaaim.core.artifact.published      (auth-handler in commit)
t=14:45:00  gaaim.core.session.ended
```

Every event is individually signed and carries attribution. The `prev` chain binds consecutive events from the same producer into a verifiable sequence. The `correlationid` (a session identifier) groups all events into a single session. A full signed walkthrough of this trace, including canonical serialization and signature computation, appears in **Appendix A.1**.

---

# 7. Profile Architecture

## 7.1 What a Profile Defines

A **Profile** is a versioned extension to GAAIM Core published under the `gaaim.<slug>.` namespace. A conformant profile document MUST specify:

1. **A namespace slug** (for example, `eng`, `legal`, `health`) that is unique across the profile registry.
2. **An event type catalog** enumerating every `gaaim.<slug>.<entity>.<verb>` event the profile defines, with JSON Schemas for each payload.
3. **Profile-specific attribution roles** that extend (but do not redefine) the Core base-set roles (§3.5.3). Role names MUST NOT collide with Core role names.
4. **An expertise vocabulary slice** registering domain-specific expertise identifiers usable in attribution (for example, `expertise://civil-procedure@frcp-2024` for a Legal Profile).
5. **Conformance test fixtures** — valid and invalid event samples that implementations use to verify profile conformance.

A profile MAY additionally specify:

6. **Profile-specific extension attributes** beyond those defined in §3.3, subject to the CloudEvents extension naming rules.
7. **Profile benchmark metrics** — aggregable measurements derived from profile events, with definitions suitable for cross-organization comparison.
8. **Regulatory annexes** documenting how profile events support specific regulatory frameworks (HIPAA, SOX, EU AI Act, FedRAMP, etc.).

Profiles MUST NOT:

- Modify the Core envelope attributes or their semantics.
- Modify the signing protocol or canonical serialization rules.
- Redefine Core event types under a profile namespace.
- Override the Core attribution base-set.

This constraint is what makes Core events and profile events interoperable at the envelope level (§2.3): a Core-only consumer can always parse, verify, and correlate profile events even without understanding their payloads.

## 7.2 Profile Registry

The canonical profile registry is published at `https://gaaim.org/registry/profiles`. The following table reflects the registry as of v0.1-draft publication.

| Profile Name       | Slug       | Status                                | Current Version |
| ------------------ | ---------- | ------------------------------------- | --------------- |
| Engineering        | `eng`      | Draft (shipping with Core v0.1-draft) | v0.1-draft      |
| Legal              | `legal`    | Proposed                              | —               |
| Healthcare         | `health`   | Proposed                              | —               |
| Financial Services | `fin`      | Proposed                              | —               |
| Consulting         | `consult`  | Proposed                              | —               |
| Research           | `research` | Proposed                              | —               |
| Government         | `gov`      | Proposed                              | —               |
| Creative           | `creative` | Proposed                              | —               |

The registry is the authoritative source; this table is illustrative and will be superseded by the registry contents as profiles progress through the lifecycle.

## 7.3 Profile Lifecycle

This section describes the lifecycle under which profiles are intended to progress if and when a formal stewardship body is established. Under the current published-reference posture (§7.6), profiles have no active ratification process; the lifecycle is documented here so implementers understand the envisioned framework.

Profiles are intended to progress through four stages:

### 7.3.1 Proposed

A proposal specifies:

- The profile's scope and motivating use cases
- A sponsoring organization committing implementation capacity
- An initial subcommittee membership of at least three domain experts
- A target event catalog outline (not yet normative)

Proposals are reviewed for alignment with Core, non-overlap with existing profiles, and sufficiency of committed resources. A profile MAY remain in Proposed status indefinitely; abandoned proposals are closed after 18 months of inactivity.

### 7.3.2 Draft (v0.x)

A profile subcommittee drafts the event catalog, attribution roles, expertise vocabulary, and conformance test fixtures. Drafts are published at `https://gaaim.org/spec/profiles/<slug>/v0.x` for review. External implementers are invited to prototype against the draft and file feedback.

Breaking changes between v0.x releases are permitted. Profiles at this stage SHOULD NOT be used for audit-defensible evidence generation, though implementations MAY emit draft-profile events for development and testing.

### 7.3.3 Candidate (v1.0-rc)

A candidate release is appropriate when:

- At least **two independent implementations** emit conformant events against the profile
- The conformance test suite passes for both implementations
- A backward-compatibility policy is documented
- A public comment period of at least 60 days has completed

Candidate releases MAY receive non-breaking changes based on late feedback. No further breaking changes are permitted.

### 7.3.4 Ratified (v1.0)

On ratification by the stewardship body:

- The profile is promoted in the registry
- The "GAAIM Certified" mark becomes available for the profile
- The profile's version enters semantic versioning discipline (breaking changes require v2.0)

Profiles follow independent semantic versioning. A profile at v2.1 MAY coexist with Core at v1.3; breaking Core changes trigger revalidation of all profiles but do not automatically bump profile versions.

## 7.4 Tenant Custom Extensions

Tenants MAY define private event types under `gaaim.<tenant-slug>.*` namespaces without any stewardship-body involvement. Tenant custom extensions:

- MUST follow Core envelope rules, signing protocol, and attribution model.
- MUST use a tenant-slug that is distinct from any registered profile slug and from the reserved `core` slug.
- MUST NOT claim profile conformance.
- MAY be submitted for promotion to a profile namespace if a stewardship body is operating.

A tenant custom extension is a candidate for **promotion** to a profile namespace when:

- At least three independent organizations have adopted it
- A subcommittee forms around it with domain expertise
- The event catalog, roles, and vocabulary meet profile-quality standards

The envisioned promotion path is documented in **Appendix C**.

## 7.5 Profile Namespace Rules

The `gaaim.` prefix is reserved. The following sub-namespaces have restricted registration authority:

| Namespace                      | Registration Authority                                       |
| ------------------------------ | ------------------------------------------------------------ |
| `gaaim.core.*`                 | Publisher or future stewardship body                         |
| `gaaim.<registered-profile>.*` | The profile's subcommittee, subject to stewardship-body approval |
| `gaaim.<tenant-slug>.*`        | The tenant itself, no stewardship-body involvement required  |

Unregistered namespaces (slugs that are neither `core`, nor a registered profile, nor an established tenant extension) MUST NOT be emitted. Sinks encountering unregistered namespaces SHOULD log the event for review and MAY reject it.

## 7.6 Governance Posture

This specification is published as a reference document. **No formal working group is currently operating**, and Edson Technologies, Inc. is not actively running a standards process around it.

**Current posture:**

- Edson Technologies is the **publisher** of this specification. The company authored the text, publishes it openly under the licenses declared in the front matter, and maintains the canonical repository at `https://github.com/GAAIM-standard/spec`.
- Edson Technologies is the first implementer of GAAIM via [ForgeTrack](https://forgetrack.com). The event model is in production use inside that product.
- Issues and pull requests on GitHub are welcome, but response times are not guaranteed and there is no RFC process in flight.
- There is no ratified profile registry, no algorithm-registry governance process, no ratified conformance certification program, and no active profile subcommittee.

**Path to active stewardship.** If independent implementer interest emerges — multiple parties shipping GAAIM-conformant producers or sinks, interop requests, or demand from regulators and auditors — Edson Technologies intends to contribute the specification, trademarks, and registries to a neutral standards body (Linux Foundation, Joint Development Foundation, OpenSSF, IETF, or equivalent) at that time. The company does not intend to operate a long-term independent standards process.

Until such contribution occurs, references in this specification to a "stewardship body," "Working Group," or "subcommittee" describe roles in a hypothetical future governance structure, not actors currently operating.

### 7.6.1 Contribution Terms (Anticipatory)

If and when governance is contributed to a neutral body, the contribution is anticipated to transfer:

- **Specification text.** Copyright assignment of the GAAIM Core and profile specifications under CC-BY-SA 4.0 (already the publication license).
- **Trademark.** Assignment of the "GAAIM" trademark and wordmark.
- **Registries.** Operational control of the algorithm registry, the profile registry, and the schema registry.
- **Certification program.** If a conformance certification program has been established by the contribution date, the receiving body assumes its operation.
- **Repositories.** Administrative control of `github.com/GAAIM-standard/*` repositories.

These are anticipatory terms, not binding commitments. The specific terms of any future contribution will be negotiated with the receiving body at that time.

---

# 8. Compliance Levels

## 8.1 Overview

A GAAIM implementation declares a **compliance level** describing what it does with events. Levels are cumulative: each level inherits all requirements of the levels below it.

| Level  | Name        | Summary                                                      |
| ------ | ----------- | ------------------------------------------------------------ |
| **L0** | Development | Emits GAAIM-structured events without signatures. Not audit-eligible |
| **L1** | Emitter     | L0 + signs events at origin. Audit-eligible                  |
| **L2** | Sink        | L1 + receives, stores, and serves signed events              |
| **L3** | Processor   | L2 + enriches or correlates events while preserving provenance |

Compliance level is declared independently of profile conformance (§8.5.2). An implementation at L1 + Engineering Profile is a different conformance statement from L1 + Engineering Profile + Legal Profile.

## 8.2 L0 — Development

L0 — Development is the non-audit tier. It exists to support local development, integration testing, CI dry-runs, tooling bring-up, and non-audit analytics pipelines where cryptographic evidence is unnecessary. An L0 implementation:

- MUST produce events conforming to the Core envelope schema (§3).
- MUST populate all REQUIRED context attributes (§3.2) and all REQUIRED extension attributes (§3.3) except `signature` and `signaturekey`.
- MUST emit `"auditeligible": false` on every event.
- MUST validate `data` payloads against the `dataschema` each event references.
- MUST NOT populate `signature` or `signaturekey`.
- MAY omit `prev`, `correlationid`, `causationid`, and other OPTIONAL attributes.

**L0 events MUST NOT be presented as audit evidence.** An L0 implementation MUST declare its level in documentation, MUST NOT claim audit or compliance benefits from L0 emission, and MUST NOT emit events to a production sink that is accepting audit-eligible traffic on the same channel.

**Sink behavior.** L2 and L3 sinks MAY refuse L0 events entirely. Sinks that accept L0 events MUST segregate them from L1+ events (separate storage partition, separate query namespace, or explicit filtering on `auditeligible`) and MUST NOT return L0 events through any query marked as returning audit evidence. Sinks MUST expose an `includeDevelopment` query parameter (default `false`) that callers opt into when they want L0 events returned.

**Transition.** An implementation SHOULD progress from L0 to L1 before the first production emission. L0 is explicitly not a long-term operating tier; producers that remain at L0 are neither audit-eligible nor conformant for production use cases.

## 8.3 L1 — Emitter

**L1 — Emitter** is the baseline for audit-eligible emission. An L1 implementation:

- MUST satisfy all L0 requirements.
- MUST generate and maintain at least one Ed25519 keypair per production `source`.
- MUST register public keys in a GAAIM Key Registry (§5.3) before emitting signed events.
- MUST sign every emitted event per §5.3.4 using the MTI algorithm or another registered algorithm.
- MUST populate `signature` and `signaturekey` on every emitted event.
- MUST emit `"auditeligible": true` on every event.
- MUST populate `prev` on every emitted event per §3.3. The first event from a newly-registered key MUST use `"prev": null`.
- MUST rotate keys at least annually per §5.4.
- MUST support key revocation per §5.5.
- SHOULD populate `correlationid` for multi-event workflows.
- SHOULD populate `causationid` when the causal predecessor is known.

**L1 events are audit-eligible.** Any party in possession of the L1 implementation's public keys can verify that the implementation emitted exactly those events, unaltered, at origin.

## 8.4 L2 — Sink

**L2 — Sink** is the baseline for storage and retrieval. An L2 implementation:

- MUST satisfy all L1 requirements (an L2 implementation is also an emitter — even if only of administrative events).
- MUST accept GAAIM events from external producers via a documented transport.
- MUST verify signatures on receipt per §5.3.5 and MUST reject events with invalid signatures.
- MUST preserve the original signed bytes of each received event for verification by downstream consumers.
- MUST NOT mutate received events; corrections MUST be emitted as new events.
- MUST expose a query API allowing retrieval of stored events by `id`, `tenantid`, `correlationid`, `time` range, `type`, and `source`.
- MUST expose retrieval of the Key Registry entries needed to verify stored events.
- MUST preserve events for at least the retention window declared in the implementation's documentation.

**L2 implementations are trusted custody points.** An L2 implementation does not need to be trusted to preserve *truth* — signatures protect truth — but it is trusted to preserve *availability* and *integrity of storage*.

## 8.5 L3 — Processor

**L3 — Processor** is the tier for systems that derive value from events beyond storage. An L3 implementation:

- MUST satisfy all L2 requirements.
- MAY run enrichment, correlation, aggregation, inference, or projection over stored events.
- MUST preserve the provenance chain: any derived record MUST reference the originating event `id`(s) and MUST NOT obscure or discard source signatures.
- MUST emit derived events (projections, summaries, aggregations) as new signed events under its own key, with `causationid` referencing the source event(s).
- MUST document its derivation logic such that an auditor can trace any derived record back to its source events.

**L3 is the tier at which third-party verification becomes most valuable.** An L3 implementation is making claims about its events and about records derived from them; both claims need to be verifiable.

## 8.6 Conformance Declarations

### 8.6.1 Declaration Format

A conformant implementation declares its GAAIM conformance via a machine-readable manifest published at a stable URL. The manifest MUST include:

```json
{
  "implementation": "Example IDE Adapter",
  "vendor": "Example Corp",
  "version": "2.3.1",
  "gaaim": {
    "coreVersion": "1.0",
    "complianceLevel": "L1",
    "profiles": [
      {"slug": "eng", "version": "1.0", "conformance": "full"}
    ],
    "algorithms": ["ed25519"],
    "conformanceTestResults": "https://example.com/gaaim-tests-2026q2.json",
    "attestationType": "self",
    "attestedAt": "2026-04-01T00:00:00Z"
  }
}
```

The `attestationType` MUST be one of:

- `self` — the implementer ran the conformance test suite and self-attests to the results.
- `third-party` — an accredited testing lab ran the conformance test suite and attested to the results.

### 8.6.2 Profile Conformance

Profile conformance is declared per profile. An implementation MAY declare:

- **`full`** — implements every REQUIRED event type and role defined by the profile at the stated version.
- **`partial`** — implements a subset, with the implemented event types enumerated in an accompanying manifest field.
- **`consumer-only`** — does not emit profile events but correctly parses and validates them (applies to L2+ implementations).

### 8.6.3 Progression Policy

- **v0.x (draft) spec:** self-attestation only. Results published openly on a best-effort basis.
- **v1.0–v1.x:** self-attestation for all levels; third-party attestation OPTIONAL and encouraged.
- **v2.0+:** third-party attestation REQUIRED for L3 conformance claims (and for any voting privileges under a future stewardship body, if one is operating). L0, L1, L2 remain self-attestable.

### 8.6.4 Conformance Test Suite

The conformance test suite for Core and each ratified profile is published under Apache 2.0 at `https://github.com/GAAIM-standard/conformance-tests` and is versioned with the specification it validates. Test fixtures cover:

- Envelope well-formedness
- Required-attribute presence
- Canonical-serialization determinism
- Signature verification (valid, tampered, expired, revoked, wrong-algorithm)
- Chain integrity (valid chains, broken chains, interleaved chains)
- Profile event validation against published schemas
- Attribution-model enforcement (required roles, valid principal types, confidence bounds)

Implementations MUST link their conformance test results from their conformance manifest (§8.6.1).

---

# 9. Engineering Profile v0.1-draft

## 9.1 Scope

The Engineering Profile defines event types, attribution conventions, and expertise vocabulary for **software engineering knowledge work**: code authored, reviewed, tested, built, and deployed by humans working with AI assistance. It is the first profile published against GAAIM Core and is intended as the reference model for subsequent profiles.

This profile does not attempt to exhaustively catalog every event an engineering organization might care about. It specifies a minimal set of canonical events that cover the typical lifecycle of a code change — authorship through deployment — and leaves finer-grained events (linting, formatting, dependency updates, observability-derived events) to future MINOR versions of the profile or to tenant extensions.

The profile slug is `eng`. All event types defined in this section are registered under `gaaim.eng.*`.

## 9.2 Profile Metadata

| Attribute             | Value                                       |
| --------------------- | ------------------------------------------- |
| Profile name          | Engineering                                 |
| Profile slug          | `eng`                                       |
| Specification version | 0.1-draft                                   |
| Status                | Draft (shipping with GAAIM Core v0.1-draft) |
| Maintainer            | Edson Technologies, Inc. (publisher)        |
| Schema namespace      | `https://gaaim.org/schemas/v0.1-draft/eng/` |

## 9.3 Event Catalog

The v0.1-draft Engineering Profile defines **fifteen** event types. All are **normative**: implementations claiming Engineering Profile conformance MUST use these type names for the logical concepts they represent.

**Version-control lifecycle events:**

| Event Type                      | Purpose                                                      |
| ------------------------------- | ------------------------------------------------------------ |
| `gaaim.eng.commit.authored`     | A version-control commit is created                          |
| `gaaim.eng.pr.opened`           | A pull request (or equivalent change request) is opened      |
| `gaaim.eng.pr.reviewed`         | A pull request receives a code review action (approve, request-changes, comment) |
| `gaaim.eng.pr.merged`           | A pull request is merged into its target branch              |
| `gaaim.eng.review.completed`    | A standalone code review (not attached to a PR) is completed |
| `gaaim.eng.test.executed`       | A test run (unit, integration, end-to-end) completes         |
| `gaaim.eng.build.completed`     | A build pipeline completes (success or failure)              |
| `gaaim.eng.deployment.recorded` | A deployment to an environment is recorded                   |

**AI-augmented workflow events:**

| Event Type                         | Purpose                                 |
| ---------------------------------- | --------------------------------------- |
| `gaaim.eng.ai.prompt.submitted`    | AI model invoked with a prompt          |
| `gaaim.eng.ai.response.received`   | AI model returned a response            |
| `gaaim.eng.ai.suggestion.accepted` | Human accepted an AI suggestion         |
| `gaaim.eng.ai.suggestion.rejected` | Human rejected an AI suggestion         |
| `gaaim.eng.codegen.applied`        | AI-generated code applied to repository |

**Supply-chain and security events:**

| Event Type                     | Purpose                                          |
| ------------------------------ | ------------------------------------------------ |
| `gaaim.eng.dependency.updated` | Project dependency version changed               |
| `gaaim.eng.security.scanned`   | Security scan executed against code or artifacts |

## 9.4 Event Payload Shapes

Each event's full payload is defined by its JSON Schema at `https://gaaim.org/schemas/v0.1-draft/eng/<event-type>.json`. The following subsections describe required `data` fields beyond the Core contract (§3.4).

### 9.4.0 Reference Format Normalization

Engineering Profile events reference external objects (repositories, commits, pull requests, build runs, deployment targets, packages, containers) using string identifiers. Two tools observing the same logical object MUST produce identical reference strings. This section defines the normalization rules every conformant producer MUST apply.

**Repository references.** The `repository` field MUST be an RFC 3986 URI with scheme in the set `{https, ssh, git+https, git+ssh, hg+https, svn+https}`. Producers MUST normalize repository URIs before emission:

- Scheme MUST be lowercase.
- Host MUST be lowercase.
- Trailing `.git` suffix MUST be stripped (for `git` and `git+*` schemes).
- Trailing `/` MUST be stripped.
- Default port numbers (`:443` for https, `:22` for ssh) MUST be stripped.
- Userinfo (`user@`) MUST be stripped; authentication belongs in transport headers, not the identifier.
- Path-segment case is preserved.

Examples (normalized): `https://github.com/acme/service-api`, `ssh://git@gitlab.example.com/platform/gateway`, `git+https://dev.azure.com/contoso/Project/_git/repo`.

**Commit hash references.** All commit-hash fields MUST be prefix-encoded:

- `sha1:<40-lowercase-hex>` for Git SHA-1 commits
- `sha256:<64-lowercase-hex>` for Git SHA-256 commits, Mercurial, and other SHA-256-hashing VCSes
- `perforce:<changelist-number>` for Perforce changelists
- `svn:<revision-number>` for Subversion revisions

Hex digits MUST be lowercase. Uppercase hex produces a different canonical form and will fail signature verification.

**Artifact references.** The `artifactRef` field MUST be prefix-encoded:

- `oci:<registry>/<repository>@<digest>` for OCI images (digest form REQUIRED when cryptographic identity matters)
- `oci:<registry>/<repository>:<tag>` for OCI images referenced by tag only
- `npm:<package>@<version>` for npm packages
- `nuget:<package>/<version>` for NuGet packages
- `pypi:<package>==<version>` for PyPI packages
- `maven:<group>:<artifact>:<version>` for Maven artifacts
- `cargo:<crate>@<version>` for Rust crates
- `go:<module>@<version>` for Go modules
- `commit:<vcs-hash-ref>` for direct commit references

**Diff and content hashes.** Hashes over diff content, file content, or prompt/response text MUST be prefix-encoded: `sha256:<64-lowercase-hex>` or `blake3:<64-lowercase-hex>`. SHA-256 is the MTI content-hash algorithm.

**Producer obligation.** Normalization MUST be performed at the producer, not at the sink. Sinks that receive non-normalized references MUST NOT rewrite them (rewriting would invalidate the signature); sinks MUST instead flag the event as a **normalization violation** and expose the flag through their query API. Producers that emit non-normalized references are non-conformant at L1+.

### 9.4.1 `gaaim.eng.commit.authored`

Required `data` fields:

- `commitHash` (string) — content-addressed identifier of the commit, prefix-encoded per §9.4.0.
- `vcsSystem` (string) — the version control system: one of `git`, `mercurial`, `fossil`, `perforce`, `svn`.
- `repository` (string) — repository URI, normalized per §9.4.0.
- `branch` (string) — the branch or reference the commit was authored against.
- `linesAdded` (integer, ≥ 0) — lines added in this commit.
- `linesRemoved` (integer, ≥ 0) — lines removed in this commit.
- `filesChanged` (integer, ≥ 0) — count of files touched.
- `message` (string) — the commit message (truncation permitted; truncated messages MUST be marked with `messageTruncated: true`).

### 9.4.2 `gaaim.eng.pr.opened`

Required `data` fields:

- `pullRequestId` (string) — provider-specific PR identifier.
- `repository` (string) — repository URI, normalized per §9.4.0.
- `sourceBranch` (string).
- `targetBranch` (string).
- `title` (string).
- `commitCount` (integer, ≥ 1).

### 9.4.3 `gaaim.eng.pr.reviewed`

Required `data` fields:

- `pullRequestId` (string).
- `repository` (string) — repository URI, normalized per §9.4.0.
- `reviewAction` (string enum) — one of `approved`, `requested_changes`, `commented`, `dismissed`.
- `commentsCount` (integer, ≥ 0).

The reviewing principal is recorded in `data.attributions` with role `reviewer` (Core base role).

### 9.4.4 `gaaim.eng.pr.merged`

Required `data` fields:

- `pullRequestId` (string).
- `repository` (string) — repository URI, normalized per §9.4.0.
- `mergeCommitHash` (string) — hash of the merge/squashed/rebased commit, prefix-encoded per §9.4.0.
- `mergeStrategy` (string enum) — one of `merge`, `squash`, `rebase`, `fast-forward`.

### 9.4.5 `gaaim.eng.review.completed`

For standalone code reviews not bound to a PR (pre-commit reviews, post-hoc audits, pair reviews).

Required `data` fields:

- `artifactRefs` (array of strings) — references to artifacts reviewed, prefix-encoded per §9.4.0 (e.g., `commit:sha1:...`, `file:path/to/file.ts`).
- `reviewType` (string enum) — one of `pre-commit`, `post-commit`, `pair-review`, `audit`.
- `outcome` (string enum) — one of `accepted`, `rejected`, `conditional`.
- `findingsCount` (integer, ≥ 0).

### 9.4.6 `gaaim.eng.test.executed`

Required `data` fields:

- `testSuite` (string) — identifier of the test suite that ran.
- `testFramework` (string) — the test runner used (e.g., `jest`, `pytest`, `xunit`, `go-test`).
- `totalTests` (integer, ≥ 0).
- `passed` (integer, ≥ 0).
- `failed` (integer, ≥ 0).
- `skipped` (integer, ≥ 0).
- `durationMs` (integer, ≥ 0).

The invariant `passed + failed + skipped ≤ totalTests` MUST hold.

### 9.4.7 `gaaim.eng.build.completed`

Required `data` fields:

- `buildId` (string) — the build pipeline's identifier for this run.
- `buildSystem` (string) — the CI system (e.g., `github-actions`, `gitlab-ci`, `jenkins`, `azure-pipelines`).
- `repository` (string) — repository URI, normalized per §9.4.0.
- `commitHash` (string) — the commit the build ran against, prefix-encoded per §9.4.0.
- `outcome` (string enum) — one of `success`, `failure`, `cancelled`, `timeout`.
- `durationMs` (integer, ≥ 0).

### 9.4.8 `gaaim.eng.deployment.recorded`

Required `data` fields:

- `deploymentId` (string).
- `environment` (string) — e.g., `production`, `staging`, `canary`, `dev`.
- `artifactRef` (string) — identifier of what was deployed, prefix-encoded per §9.4.0 (e.g., `oci:ghcr.io/acme/service@sha256:...`, `commit:sha1:...`).
- `deploymentStrategy` (string enum) — one of `rolling`, `blue-green`, `canary`, `recreate`, `direct`.
- `outcome` (string enum) — one of `success`, `failure`, `rolled-back`, `in-progress`.

### 9.4.9 `gaaim.eng.ai.prompt.submitted`

Captures the submission of a prompt to an AI model. Emitted by the tool or agent that dispatches the prompt (IDE plugin, CLI, orchestrator).

Required `data` fields:

- `promptHash` (string) — hash of the prompt text, prefix-encoded per §9.4.0.
- `modelId` (string) — the AI model invoked, as a `principalId` of type `AIModel` (e.g., `ai:anthropic:claude-opus-4-6`).
- `sessionId` (string) — identifier grouping related prompt/response pairs in a single user-facing session.
- `promptTokens` (integer, ≥ 0) — token count of the prompt as measured by the tool.
- `tokenizer` (string) — the tokenizer used to count tokens (e.g., `cl100k_base`, `o200k_base`, `claude-v3`).

Optional `data` fields: `systemPromptHash`, `contextItemHashes` (array, prefix-encoded), `contextItemCount`, `temperature`, `maxTokens`, `toolsDefined`.

`data.attributions` MUST include (a) the Human principal who initiated the prompt (role `proposer` or `delegator`) and (b) the AIModel principal identified by `modelId` (role `executor`).

### 9.4.10 `gaaim.eng.ai.response.received`

Captures receipt of an AI model's response to a prompt. Chain linkage: `causationid` MUST reference the `id` of the `gaaim.eng.ai.prompt.submitted` event that elicited this response.

Required `data` fields:

- `responseHash` (string) — hash of the response text, prefix-encoded per §9.4.0.
- `modelId` (string) — MUST match the `modelId` of the causing prompt event.
- `sessionId` (string) — MUST match the causing prompt event.
- `responseTokens` (integer, ≥ 0).
- `finishReason` (string enum) — one of `stop`, `length`, `tool_use`, `content_filter`, `error`, `cancelled`.
- `latencyMs` (integer, ≥ 0) — elapsed time from prompt submission to response completion.

Optional `data` fields: `toolCallCount`, `reasoningTokens`, `cachedPromptTokens`, `providerRequestId`.

`data.attributions` MUST include the AIModel principal (role `executor`, confidence `1.0`).

### 9.4.11 `gaaim.eng.ai.suggestion.accepted`

Captures a human's acceptance of an AI-generated suggestion. Chain linkage: `causationid` MUST reference the `id` of the `gaaim.eng.ai.response.received` event containing the suggestion.

Required `data` fields:

- `suggestionType` (string enum) — one of `code`, `completion`, `diff`, `refactor`, `documentation`, `test`, `commit-message`, `review-comment`, `other`.
- `artifactRef` (string) — reference to what was modified, prefix-encoded per §9.4.0 (or using `file:` scheme for local file paths).

Optional `data` fields: `responseHash`, `suggestionSpan` (structured reference to response text), `modifiedAfterAcceptance` (boolean), `modificationRatio` (number 0.0–1.0).

`data.attributions` MUST include the Human approver and the AIModel proposer whose suggestion was accepted.

### 9.4.12 `gaaim.eng.ai.suggestion.rejected`

Captures a human's rejection of an AI-generated suggestion. Rejection events let audit consumers compute AI suggestion acceptance rates and demonstrate meaningful human review. Producers that emit only acceptance events cannot substantiate human-in-the-loop claims. Chain linkage: `causationid` MUST reference the `gaaim.eng.ai.response.received` event.

Required `data` fields:

- `suggestionType` (string enum) — same values as §9.4.11.
- `rejectionReason` (string enum) — one of `incorrect`, `stylistic`, `security-concern`, `out-of-scope`, `unnecessary`, `verbose`, `hallucination`, `other`.

Optional `data` fields: `responseHash`, `rejectionNotes` (truncated if > 1024 bytes with `rejectionNotesTruncated: true`).

`data.attributions` MUST include the Human principal who rejected (role `reviewer`). The AIModel principal MAY be included (role `proposer`).

### 9.4.13 `gaaim.eng.codegen.applied`

Captures the application of AI-generated code to a repository. Distinguishes the decision-to-accept event (§9.4.11) from the actual code change that resulted. Chain linkage: `causationid` SHOULD reference the `gaaim.eng.ai.suggestion.accepted` event. If applied without preceding acceptance (autonomous agent mode), the producer MUST emit `"causationid": null` and SHOULD populate `data.autonomousApplication: true`.

Required `data` fields:

- `diffHash` (string) — hash of the applied diff, prefix-encoded per §9.4.0.
- `repository` (string) — repository URI, normalized per §9.4.0.
- `artifactRefs` (array of strings) — file paths or references touched by the diff.
- `linesAdded` (integer, ≥ 0).
- `linesRemoved` (integer, ≥ 0).
- `filesChanged` (integer, ≥ 0).

Optional `data` fields: `commitHash` (prefix-encoded per §9.4.0), `branch`, `uncommittedAtTime` (boolean), `autonomousApplication` (boolean).

`data.attributions` MUST include (a) the Human executor who applied the diff and (b) the AIModel proposer. In autonomous mode, the Human may be substituted by a Workflow or Service principal.

### 9.4.14 `gaaim.eng.dependency.updated`

Captures a dependency version change in a project's manifest file (package.json, requirements.txt, Cargo.toml, pom.xml, go.mod, etc.).

Required `data` fields:

- `packageManager` (string enum) — one of `npm`, `nuget`, `pypi`, `maven`, `cargo`, `go-mod`, `composer`, `gem`, `pub`, `other`.
- `packageName` (string).
- `oldVersion` (string or `null`) — `null` if this is a new dependency addition.
- `newVersion` (string or `null`) — `null` if this is a dependency removal.
- `updateSource` (string enum) — one of `manual`, `dependabot`, `renovate`, `snyk`, `other-automated`.
- `repository` (string) — repository URI, normalized per §9.4.0.

Optional `data` fields: `versionChangeType` (major/minor/patch/prerelease/pin/unpin/revert), `pullRequestId`, `vulnerabilityFixed` (CVE or GHSA identifier), `changelogUrl`.

For manual updates, `data.attributions` MUST include the Human approver. For automated updates, MUST include the automation as a Service/Tool principal (proposer) plus the Human approver who merged the PR.

### 9.4.15 `gaaim.eng.security.scanned`

Captures the execution of a security scan over source code, dependencies, container images, or running services.

Required `data` fields:

- `scannerName` (string) — e.g., `snyk`, `trivy`, `semgrep`, `gitleaks`, `dependabot`, `codeql`.
- `scannerVersion` (string).
- `scanType` (string enum) — one of `sast`, `dast`, `sca`, `secrets`, `container`, `iac`, `license`, `other`.
- `findingsCritical`, `findingsHigh`, `findingsMedium`, `findingsLow`, `findingsInformational` (integers, ≥ 0).
- `target` (string) — what was scanned: a repository URI, a commit reference, an artifact reference, or a URL for dynamic scans (all normalized per §9.4.0).

Optional `data` fields: `durationMs`, `scanReportUrl`, `baselineCommit` (prefix-encoded per §9.4.0), `newFindings`, `resolvedFindings`.

`data.attributions` MUST include the scanner as a Tool or Service principal (role `executor`, confidence `1.0`). If the scan was human-initiated, the initiating Human principal MAY be included (role `delegator`).

## 9.5 Attribution Roles

The Engineering Profile **uses the Core base role set** (§3.5.3) without extension in v0.1-draft. Mapping of common git-era terminology to Core roles:

| Git/Engineering Term                  | GAAIM Core Role                            |
| ------------------------------------- | ------------------------------------------ |
| Author (of a commit)                  | `proposer`                                 |
| Committer (applied the commit)        | `executor`                                 |
| Code reviewer                         | `reviewer`                                 |
| PR approver / maintainer who merges   | `approver`                                 |
| Automated test runner                 | `executor` (with principal type `Tool`)    |
| CI pipeline                           | `executor` (with principal type `Service`) |
| Prompt author                         | `proposer`                                 |
| AI model producing a suggestion       | `proposer`                                 |
| Human accepting AI suggestion         | `approver`                                 |
| Human rejecting AI suggestion         | `reviewer`                                 |
| Automation bot (Dependabot, Renovate) | `proposer`                                 |
| Security scanner                      | `executor`                                 |

Future minor versions of the Engineering Profile MAY register engineering-specific roles if cross-implementation need emerges.

## 9.6 Expertise Vocabulary

The Engineering Profile registers expertise identifiers under the `expertise://` URI scheme for use in `attributions[].context` and in tenant analytics. Expertise identifiers follow the pattern:

```
expertise://<domain>@<version-or-qualifier>
```

Examples:

| Identifier                    | Meaning                          |
| ----------------------------- | -------------------------------- |
| `expertise://react@18`        | React.js, version 18             |
| `expertise://typescript@5`    | TypeScript, version 5            |
| `expertise://kubernetes@1.28` | Kubernetes, version 1.28         |
| `expertise://postgres@16`     | PostgreSQL, version 16           |
| `expertise://rust@stable`     | Rust, stable toolchain           |
| `expertise://aws-lambda`      | AWS Lambda (unversioned service) |

The v0.1-draft Engineering Profile does not publish a closed enumeration of expertise identifiers. New identifiers MAY be proposed via issues on the specification repository; the registry at `https://gaaim.org/registry/expertise/eng` is the authoritative source.

## 9.7 Engineering Profile Conformance

An implementation declares Engineering Profile conformance per §8.6.2 as one of:

- **Full conformance:** emits (or, for sinks/processors, correctly parses and validates) all eight event types with all required fields populated per §9.4.
- **Partial conformance:** emits a named subset. The manifest MUST enumerate the event types implemented.
- **Consumer-only conformance:** L2+ sinks that correctly parse, validate, and store Engineering events but do not emit them.

Engineering Profile conformance is orthogonal to compliance level. An implementation MAY be L1 + Engineering (full), L2 + Engineering (partial: commits, PRs, deployments only), or L3 + Engineering (full) + Legal (consumer-only).

---

# 10. Security Considerations

## 10.1 Threat Model

GAAIM is designed against the following adversaries:

1. **The honest-but-curious storage operator.** A sink or processor that stores GAAIM events MAY attempt to learn information from them, but MUST NOT be trusted to preserve truth. Origin signing places the trust boundary at the producer, not the store.
2. **The malicious storage operator.** A storage operator that mutates, deletes, inserts, or reorders historical events. Signatures localize tampering to the mutated event; chain hashes (§5.6) propagate the detection forward.
3. **The compromised producer.** An attacker who steals a producer's private key and emits fraudulent events. Key revocation (§5.5) bounds the window of forgery to `[compromise, revokedAt]`.
4. **The registry substituter.** An attacker who replaces a public key in the Key Registry with one under their control, retroactively validating forged events. Mitigation is layered: (a) URI-qualified `signaturekey` binds each event to a specific registry instance (§5.3.2.1); (b) sinks SHOULD maintain a per-sink allowlist of accepted registry origins (§10.4.1) and reject events whose `signaturekey` resolves to an unallowed origin with error code `registry-mismatch`; (c) registries SHOULD publish registry state to a transparency log (§5.8, deferred). Until transparency logs land, sink-side origin allowlisting is the primary mitigation.
5. **The prefix-swap downgrader.** An attacker who changes the algorithm prefix on a `signature` attribute to force a verifier into a weaker algorithm. §5.2.3 requires verifiers to reject prefix/key-algorithm mismatches.
6. **The replay attacker.** An attacker who captures a signed event and re-submits it. Event `id` uniqueness (§3.6) and sink-side duplicate detection defend against replay.

GAAIM does NOT defend against:

- **A compromised key that is not reported.** If a producer's key is stolen and not revoked, the attacker can emit events that verify as legitimate until revocation. Producers SHOULD monitor their own emission patterns and rotate keys aggressively when anomalies are detected.
- **A colluding producer and verifier.** If both ends of the chain are controlled by an adversary, signing provides no protection. GAAIM's value is proportional to the independence of the parties.
- **Attacks on the `data` payload's semantic truth.** A properly-signed event can carry false `data` (e.g., fabricated test counts). GAAIM authenticates origin, not accuracy.

## 10.2 Key Compromise

On suspected key compromise, producers MUST:

1. Immediately register the key as revoked with an accurate `revokedAt` timestamp reflecting the earliest possible compromise time.
2. Provide a `revocationReason` to aid downstream verifiers.
3. Generate and register a replacement keypair.
4. Resume emission under the new key.
5. Notify downstream consumers through an out-of-band channel.

Verifiers MUST check revocation status at verification time and MUST re-verify stored events if revocation data changes. A sink that has already accepted events signed with a subsequently-revoked key MUST NOT silently reclassify them; instead, it MUST surface the revocation status alongside the events when queried.

## 10.3 Personally Identifiable Information

GAAIM events carry personally identifying information (PII) in multiple places:

- `principalId` values often encode email addresses, user IDs, or employee identifiers.
- `tenantid` identifies an organization.
- `data` payloads may contain author names, commit messages, document titles, patient identifiers (in the Healthcare Profile), or other sensitive context.

Implementations MUST:

- Respect tenant isolation: events from one `tenantid` MUST NOT be exposed to principals of another tenant.
- Implement data-subject rights (access, deletion, rectification) as required by applicable jurisdiction (GDPR, CCPA, LGPD, etc.).
- Support the three principal-identifier forms defined in §3.5.6 (full, vendor-only, anonymized). Producers emit the form declared in their Key Registry record; sinks accept all three forms. A data-deletion request MAY be fulfilled by re-emitting new events under the anonymized form going forward — but MUST NOT rewrite historical events (which would invalidate their signatures). Historical-event access controls (access lists, retention-based deletion) are the mechanism for data-subject rights on already-signed events.
- NOT include credentials, API keys, tokens, passwords, or equivalent secrets in `data` payloads.

Profiles MAY define additional PII handling rules appropriate to their vertical (HIPAA PHI minimization for the Healthcare Profile, attorney-client privilege rules for the Legal Profile, etc.).

## 10.4 Tenant Isolation

Multi-tenant sinks and processors MUST enforce cryptographic and logical tenant isolation:

- Key Registry entries MUST be scoped by `tenantId`. A producer registered under tenant A MUST NOT be able to verify events to tenant B.
- Query APIs MUST require tenant scoping on every call.
- Cross-tenant joins, aggregations, or benchmarks MUST be performed against anonymized or differentially-private datasets.

The strongest cross-tenant leak risk is **principal identity correlation**: an `principalId` value referencing an AI model version (for example, `ai:anthropic:claude-opus-4-6`) is safely shared across tenants, but a `principalId` referencing a named human is not. Implementations MUST NOT expose human principal identifiers across tenant boundaries.

### 10.4.1 Registry Origin Allowlisting

Sinks MUST support configuration of an **accepted registry origins allowlist**. The allowlist is a set of HTTPS origins that the sink trusts to serve key records. Events whose `signaturekey` URI origin is not in the allowlist MUST be rejected with error code `registry-origin-not-allowed`.

Sinks MUST expose the allowlist through their conformance manifest (§8.6.1) so downstream consumers understand the sink's trust boundary. The allowlist MAY be empty (in which case the sink accepts any registry origin, which is not recommended for production deployments).

Allowlist updates SHOULD require out-of-band authorization. Adding an origin to the allowlist expands the sink's trust surface and should be treated as a security-relevant configuration change.

## 10.5 Registry Trust

The Key Registry is a trust root. Compromise of a registry can invalidate all events signed under keys the registry serves. Implementations MUST:

- Serve registry entries over TLS 1.2 or higher. Plaintext HTTP is non-conformant.
- Enforce HSTS (HTTP Strict Transport Security) on registry origins.
- Support key-record caching with explicit `validUntil` bounds (§5.3.2.4).

Implementations SHOULD:

- Sign registry-operator metadata (the `.well-known/gaaim-registry` discovery document) with a long-lived offline key and publish the signing certificate through an out-of-band channel.
- Publish registry state to a transparency log (analogous to Certificate Transparency) once that capability lands in GAAIM (§5.8).
- Support offline registry snapshots for air-gapped or long-term archival verification. A snapshot is a signed export of all active key records at a given timestamp, verifiable against the registry operator's long-lived key.
- Rotate the registry operator's long-lived signing key no more frequently than every two years, with a 12-month overlap window where both the old and new keys are honored.

**Sink-side origin allowlisting (§10.4.1)** remains the primary defense against registry substitution until transparency logs are ratified.

## 10.6 Signature Verification Pitfalls

Common implementation errors:

- **Verifying signatures before canonicalizing** — or canonicalizing inconsistently with the producer. Reference canonicalization implementations MUST be used where available.
- **Skipping revocation checks** at ingest. A sink that verifies signatures but ignores revocation effectively trusts compromised keys.
- **Accepting self-signed registry entries** without out-of-band verification of the registry operator's identity.
- **Verifying only once, on ingest.** Long-lived stored events SHOULD be periodically re-verified, especially after key revocations or algorithm-registry updates.

---

# 11. IANA Considerations

## 11.1 Media Type Registration

This specification registers the following media type:

- **Name:** `application/gaaim-event+json`
- **Description:** GAAIM event in structured-mode JSON.
- **Specification:** This document.
- **Encoding:** UTF-8.

Producers MAY also use `application/cloudevents+json` for transport-layer interoperability; the event body is identical.

## 11.2 CloudEvents Extension Attribute Registry Entries

This specification defines the following CloudEvents context extension attributes, which are requested for inclusion in the CloudEvents Extension Attributes Registry:

| Attribute       | Type   | Specification |
| --------------- | ------ | ------------- |
| `gaaimversion`  | String | §3.3          |
| `profile`       | String | §3.3          |
| `tenantid`      | String | §3.3          |
| `traceid`       | String | §3.3          |
| `correlationid` | String | §3.3          |
| `causationid`   | String | §3.3          |
| `prev`          | String | §3.3          |
| `signature`     | String | §3.3, §5      |
| `signaturekey`  | String | §3.3, §5      |

## 11.3 URI Scheme Registration

This specification defines one URI scheme for use within GAAIM payloads:

- **Scheme:** `expertise`
- **Purpose:** Identifies domains of expertise for attribution and analytics.
- **Syntax:** `expertise://<domain>@<qualifier>` where `<domain>` is a lowercase alphanumeric identifier with hyphens and `<qualifier>` is an optional version string or release qualifier.
- **Specification:** §9.6 and profile expertise registries.

No other URI schemes are registered by this specification.

---

# 12. References

## 12.1 Normative References

- **[BCP-14]** Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels," BCP 14, RFC 2119, March 1997, updated by RFC 8174.
- **[RFC-3339]** Klyne, G., Newman, C., "Date and Time on the Internet: Timestamps," RFC 3339, July 2002.
- **[RFC-5280]** Cooper, D., et al., "Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile," RFC 5280, May 2008.
- **[RFC-8032]** Josefsson, S., Liusvaara, I., "Edwards-Curve Digital Signature Algorithm (EdDSA)," RFC 8032, January 2017.
- **[RFC-8785]** Rundgren, A., Jordan, B., Erdtman, S., "JSON Canonicalization Scheme (JCS)," RFC 8785, June 2020.
- **[RFC-CE10]** CloudEvents 1.0, Cloud Native Computing Foundation, https://github.com/cloudevents/spec/blob/v1.0/spec.md
- **[RFC-CE-JSON]** CloudEvents JSON Event Format 1.0, https://github.com/cloudevents/spec/blob/v1.0/json-format.md
- **[ULID-SPEC]** ULID Specification, https://github.com/ulid/spec
- **[FIPS-186-5]** NIST FIPS 186-5, "Digital Signature Standard (DSS)," February 2023.
- **[W3C-TRACE]** W3C Trace Context, https://www.w3.org/TR/trace-context/

## 12.2 Informative References

- **[FIPS-204]** NIST FIPS 204, "Module-Lattice-Based Digital Signature Standard (ML-DSA)," August 2024.
- **[FIPS-205]** NIST FIPS 205, "Stateless Hash-Based Digital Signature Standard (SLH-DSA)," August 2024.
- **[FIPS-206]** NIST FIPS 206, "FN-DSA (draft)," NIST, 2024.
- **[SIGSTORE]** Sigstore Project, https://www.sigstore.dev/
- **[IN-TOTO]** in-toto Framework, https://in-toto.io/
- **[CT]** Laurie, B., "Certificate Transparency," RFC 6962, June 2013.
- **[FHIR]** HL7 FHIR Release 5, https://hl7.org/fhir/
- **[OIDC]** OpenID Connect Core 1.0, https://openid.net/specs/openid-connect-core-1_0.html
- **[SOC2]** AICPA SOC 2 Trust Services Criteria, https://www.aicpa-cima.com/

## 12.3 Related Work and Positioning

GAAIM intersects with several mature provenance and attestation standards. The specification was developed as a complementary layer rather than an extension of any one of them. This section explains the delta.

### 12.3.1 Comparison Matrix

| Dimension                | GAAIM                                                        | in-toto                                   | SLSA                                            | C2PA                                  |
| ------------------------ | ------------------------------------------------------------ | ----------------------------------------- | ----------------------------------------------- | ------------------------------------- |
| **Primary object**       | Event (ledger entry)                                         | Attestation (statement)                   | Build provenance (predicate)                    | Manifest (embedded in asset)          |
| **Scope**                | Knowledge work across verticals                              | Supply-chain steps                        | Software build provenance                       | Media asset provenance                |
| **Transport**            | CloudEvents bindings (HTTP, Kafka, NATS, AMQP)               | In-band with artifacts or sidecar         | SLSA attestation carried alongside build output | Embedded in the asset file (JUMBF)    |
| **Attribution model**    | First-class `attributions` array with roles, principals, confidence | Functionary keys; step authorization      | Builder identity; build steps                   | Assertions with actor types           |
| **AI principal support** | Native (`AIModel` principal type, prompt/response event types) | Generic functionary; no AI-specific roles | None; extend via custom predicates              | CAWG AI/ML assertions (recent)        |
| **Compliance levels**    | L0/L1/L2/L3 tiers for what implementations do with events    | Conformance via attestation types         | SLSA levels 1–4 on build pipeline hardening     | None; capability declared in manifest |
| **Signing**              | Ed25519 MTI, per-event origin signing, chained               | DSSE envelopes, per-step signing          | Sigstore/cosign, per-build-output signing       | COSE, per-manifest signing            |
| **Key discovery**        | Registry (URI-qualified, HTTP API — §5.3.2)                  | Functionary key distribution out-of-band  | Sigstore transparency log                       | X.509 chain or trust lists            |

### 12.3.2 Why Not in-toto?

in-toto's attestation model is the closest analog to GAAIM. in-toto statements are signed claims about artifacts, produced by "functionaries" at defined pipeline steps. The model is powerful and widely deployed for software supply-chain attestation.

GAAIM differs in three fundamental ways:

1. **Event ledger vs. attestation envelope.** in-toto attestations describe artifacts at specific lifecycle points — "this build produced this binary." GAAIM events describe actions that occurred — "this human accepted this AI suggestion on this file at this time." The former is artifact-centric; the latter is action-centric. Audit defense for knowledge work (who did what, when, with what AI assistance) benefits from an action stream, not periodic artifact attestations.
2. **First-class attribution.** in-toto's functionary model authorizes a step but does not express joint authorship ("Alice proposed, Bob approved, AI-Model-X executed"). GAAIM's `attributions` array is exactly this — a structured record of collaborative work with roles and confidences. This is the core requirement for AI-augmented work evidence.
3. **Cross-vertical profile framework.** in-toto is supply-chain specific. GAAIM's Core + Profiles architecture (§2) lets Healthcare, Legal, Financial, and Consulting verticals extend a shared signing and attribution spine without forking the standard.

GAAIM events and in-toto attestations are complementary. A SLSA-3 build pipeline that emits in-toto attestations can also emit GAAIM `gaaim.eng.build.completed` events that reference the in-toto attestation via `causationid` or a custom reference field.

### 12.3.3 Why Not C2PA?

C2PA (Coalition for Content Provenance and Authenticity) solves media-asset provenance: an image, video, or audio file carries a signed manifest embedded in the file itself, tracking edits, authorship, and AI involvement. C2PA 2.3 adds AI/ML guidance and CAWG (Creator Assertions Working Group) identity assertions.

GAAIM is not media-centric. Knowledge work events are ledger entries, not file-embedded manifests. A GAAIM event describes an action (a commit, a PR review, a suggestion acceptance); it does not modify an asset. The C2PA manifest format would be the wrong shape: events occur without producing a new asset, or they mutate multiple assets, or they produce no asset at all.

Where GAAIM and C2PA overlap is narrow: an `gaaim.core.artifact.created` event describing a newly-created image could include a reference to the C2PA manifest embedded in that image's bytes. This is an implementation choice, not a standards-level coupling.

### 12.3.4 Why Not SLSA?

SLSA (Supply-chain Levels for Software Artifacts) defines conformance levels for build pipelines — not a wire format for events or attributions. SLSA levels describe how hardened a build process is, and SLSA provenance is typically expressed as an in-toto attestation.

GAAIM's compliance levels (L0–L3) are similarly named but describe what an implementation does with events (emits, receives, stores, processes) rather than how hardened a build pipeline is. The two taxonomies are complementary: a GAAIM L1 emitter running in a SLSA-3 build environment emits doubly-attested work.

The Engineering Profile's `gaaim.eng.build.completed` event (§9.4.7) is a richer, action-oriented counterpart to SLSA provenance — it carries attribution (who triggered the build), AI involvement (did an AI agent initiate it), and chain linkage (what prompt/suggestion led to this build) that SLSA provenance does not natively express.

### 12.3.5 Summary

GAAIM is a **new category**: an action-ledger standard for AI-augmented knowledge work, with first-class attribution, cross-vertical profile extensibility, and cryptographic audit defense. It reuses CloudEvents for transport, JCS for canonicalization, and Ed25519 for signing — all mature standards. It does not replace in-toto, SLSA, or C2PA. It complements them for the knowledge-work layer of the supply chain.

---

# Appendix A. Worked Examples

## A.1 Signed Core Event

A complete `gaaim.core.artifact.created` event with signature placeholder.

**Producer state:**

- `source`: `ide-plugin://example.org/code-adapter/instance-7`
- Key identifier (URI-qualified, per §5.3.2.1): `https://keys.acme-engineering.example/v1/keys/code-adapter-instance-7-2026q2`
- Previous event `id`: `01HQ5P3KJ6X8W2YQGMZB9N4T7P`

**Event before signing:**

```json
{
  "specversion": "1.0",
  "id": "01HQ5P3KJ6X8W2YQGMZB9N4T7R",
  "source": "ide-plugin://example.org/code-adapter/instance-7",
  "type": "gaaim.core.artifact.created",
  "subject": "file/src/auth/handler.ts",
  "time": "2026-04-04T21:30:15.123Z",
  "datacontenttype": "application/json",
  "dataschema": "https://gaaim.org/schemas/v0.1-draft/core/artifact-created.json",
  "gaaimversion": "1.0",
  "profile": "core",
  "tenantid": "acme-engineering",
  "correlationid": "session_01HQ5P3KJ0000000000000000",
  "prev": "01HQ5P3KJ6X8W2YQGMZB9N4T7P",
  "auditeligible": true,
  "data": {
    "artifactType": "source-file",
    "path": "src/auth/handler.ts",
    "sizeBytes": 2847,
    "attributions": [
      {
        "principalId": "ai:example:model-v1",
        "principalType": "AIModel",
        "role": "proposer",
        "confidence": 0.92
      },
      {
        "principalId": "user:alice@acme.example",
        "principalType": "Human",
        "role": "approver",
        "confidence": 1.0
      }
    ],
    "references": {
      "sessionId": "session_01HQ5P3KJ0000000000000000",
      "workItemId": 1247
    },
    "tags": ["auth", "refactor"]
  }
}
```

Note that this event does **not** include `causationid`, `traceid`, or other OPTIONAL attributes. Per §4.2.1, these attributes MUST NOT appear in the canonical form.

**Canonical form** (keys sorted, whitespace removed, `signature`/`signaturekey` excluded — line-wrapped here for presentation only):

```
{"auditeligible":true,"correlationid":"session_01HQ5P3KJ0000000000000000",
"data":{"artifactType":"source-file","attributions":[{"confidence":0.92,
"principalId":"ai:example:model-v1","principalType":"AIModel","role":"proposer"},
{"confidence":1,"principalId":"user:alice@acme.example","principalType":"Human",
"role":"approver"}],"path":"src/auth/handler.ts","references":{"sessionId":
"session_01HQ5P3KJ0000000000000000","workItemId":1247},"sizeBytes":2847,
"tags":["auth","refactor"]},"datacontenttype":"application/json","dataschema":
"https://gaaim.org/schemas/v0.1-draft/core/artifact-created.json","gaaimversion":
"1.0","id":"01HQ5P3KJ6X8W2YQGMZB9N4T7R","prev":"01HQ5P3KJ6X8W2YQGMZB9N4T7P",
"profile":"core","source":"ide-plugin://example.org/code-adapter/instance-7",
"specversion":"1.0","subject":"file/src/auth/handler.ts","tenantid":
"acme-engineering","time":"2026-04-04T21:30:15.123Z","type":
"gaaim.core.artifact.created"}
```

Observe that `"confidence":1.0` in the source becomes `"confidence":1` in the canonical form, per §4.2.2 (ECMAScript `ToString(Number)` emits whole-valued doubles as integers).

**Canonical byte length:** 924
**SHA-256 of canonical bytes:** `fc187674a8c6915dafe60d37bd84e18d86dc9e6ffd3950fca39c6346a95560df`

Implementations MUST produce these exact values when given the source event above. An implementation whose canonicalizer emits `"confidence":1.0` instead of `"confidence":1`, or injects `"causationid":null` because the attribute is absent from the source, will produce a different SHA-256 and fail conformance.

**After signing,** `signature` and `signaturekey` are added (signature value illustrative, not a real Ed25519 output):

```json
{
  "signature": "ed25519:<64-byte-base64url>",
  "signaturekey": "https://keys.acme-engineering.example/v1/keys/code-adapter-instance-7-2026q2"
}
```

Producers generating this event MUST emit the full JSON object with signature attributes in any field order (verifiers re-canonicalize before verifying). A conformance-test fixture version of this example with a real keypair and verifiable signature is published in the conformance-tests repository.

## A.2 Attribution Patterns

**Pattern 1: Pure human work.**

```json
"attributions": [
  {"principalId": "user:alice@acme.example", "principalType": "Human",
   "role": "proposer", "confidence": 1.0},
  {"principalId": "user:alice@acme.example", "principalType": "Human",
   "role": "executor", "confidence": 1.0}
]
```

**Pattern 2: AI drafts, human approves.**

```json
"attributions": [
  {"principalId": "ai:vendor:model-v2", "principalType": "AIModel",
   "role": "proposer", "confidence": 0.9},
  {"principalId": "user:alice@acme.example", "principalType": "Human",
   "role": "approver", "confidence": 1.0}
]
```

**Pattern 3: AI-to-AI handoff.**

```json
"attributions": [
  {"principalId": "ai:vendor:planner-v1", "principalType": "AIModel",
   "role": "proposer", "confidence": 0.85},
  {"principalId": "ai:vendor:executor-v1", "principalType": "AIModel",
   "role": "executor", "confidence": 0.95},
  {"principalId": "user:alice@acme.example", "principalType": "Human",
   "role": "verifier", "confidence": 1.0}
]
```

**Pattern 4: Tool execution with human initiation.**

```json
"attributions": [
  {"principalId": "user:alice@acme.example", "principalType": "Human",
   "role": "delegator", "confidence": 1.0},
  {"principalId": "tool:jest:29.7.0", "principalType": "Tool",
   "role": "executor", "confidence": 1.0}
]
```

## A.3 Verification Walkthrough

A conforming verifier, on receipt of the event in A.1, performs:

1. Parse the event as structured-mode JSON.
2. Extract `signature` (`"ed25519:<...>"`) and `signaturekey` (`"https://keys.acme-engineering.example/v1/keys/code-adapter-instance-7-2026q2"`).
3. Resolve `signaturekey` via the Key Registry, retrieving the public key and its `validFrom`, `validUntil`, `revokedAt`, `algorithm`.
4. Confirm `algorithm` is `Ed25519` and the signature prefix `"ed25519:"` matches.
5. Confirm `time` (`2026-04-04T21:30:15.123Z`) is within `[validFrom, validUntil)` and before `revokedAt` (if set).
6. Remove `signature` and `signaturekey` from the event object.
7. Apply JCS canonicalization per §4.2.
8. Decode the signature bytes from base64url.
9. Compute `Ed25519-Verify(publicKey, canonicalBytes, signatureBytes)`.
10. If true, accept; if false, reject with error code `signature-invalid`.

---

# Appendix B. JSON Schemas (Normative)

Normative JSON Schemas for every Core event type and every Engineering Profile event type are published in the GAAIM schemas repository:

- **Repository:** `https://github.com/GAAIM-standard/schemas`
- **Registry URL:** `https://gaaim.org/schemas/v0.1-draft/`
- **License:** Apache 2.0
- **Format:** JSON Schema draft 2020-12

The Core envelope schema (`envelope.json`) and each event-type schema are stable at the `v0.1-draft` path. On promotion to `v1.0`, schemas are republished at a new immutable path.

Implementations MUST fetch schemas from the canonical URLs or from a git clone of the schemas repository. Implementations MUST NOT modify schemas locally and present themselves as conformant.

---

# Appendix C. Profile Author Guide

This appendix describes the process under which new profiles would be proposed and authored **if and when a formal stewardship body is operating**. Under the current published-reference posture (§7.6), no active proposal process is in flight. The guidance is preserved here as reference material for the anticipated framework.

## C.1 When to Propose a New Profile

A new profile is appropriate when:

- A knowledge-work vertical has distinct event types, attribution roles, or expertise vocabulary that Core does not cover.
- At least one sponsoring organization commits to implementing against the profile.
- Domain experts are available to form a subcommittee (minimum three) under whatever governance structure is operating.
- Cross-organization interoperability will result — if events would only ever be consumed within one organization, a tenant extension (§7.4) is the right tool.

## C.2 Proposal Template

A profile proposal includes:

1. **Name, slug, one-sentence description.**
2. **Scope statement** — what the profile covers and does not cover.
3. **Motivating use cases** — at least three concrete scenarios.
4. **Event catalog outline** — proposed event types with brief descriptions. Need not be schema-complete at proposal stage.
5. **Attribution roles** — any vertical-specific roles beyond Core base.
6. **Regulatory context** — relevant standards, laws, or frameworks the profile aligns with.
7. **Sponsoring organization and subcommittee nominees.**
8. **Commitment to maintain** — the sponsor commits to maintaining the profile through at least v1.0.

Proposals would be filed at `https://github.com/GAAIM-standard/profiles` once a stewardship body establishes that repository and accepts submissions.

## C.3 Promoting Tenant Extensions

A tenant extension is a candidate for promotion to a profile namespace when:

- At least three independent organizations have adopted it.
- A subcommittee forms with sufficient domain expertise.
- The event catalog, roles, and vocabulary meet profile-quality standards.
- A stewardship body is operating and approves the promotion.

On approval, the tenant-prefixed event types are renamed to the profile namespace, tenants migrate to the new event type identifiers, and the old tenant-prefixed types are deprecated with a 12-month sunset window.

---

# Appendix D. Glossary

**Attribution.** A structured claim about which principal contributed to an event, their role, and confidence.

**Chain hash.** A sink-maintained hash linking each event to its predecessor via `SHA-256(predecessor.chainHash || event.signature)`.

**Compliance Level.** One of L0 (Compatible), L1 (Emitter), L2 (Sink), L3 (Processor); describes what an implementation does with events.

**Conformance.** The combined declaration of compliance level and profile support.

**Core.** The vertical-neutral foundation of GAAIM: envelope, attribution model, signing protocol, compliance levels, schema registry.

**Emitter.** An implementation at compliance level L1 or higher; signs and emits events.

**Event.** A single signed, attributed record of a knowledge-work action, conforming to the Core envelope.

**Event type.** A reverse-DNS identifier (e.g., `gaaim.core.artifact.created`) that determines payload schema.

**GAAIM.** Generally Accepted AI Metrics. Pronounced "game." The long-form semantic gloss is "Generally Accepted AI-augmented work Metrics."

**Key Registry.** A retrievable, versioned record of registered public keys and their metadata.

**MTI (Mandatory-to-Implement).** The algorithm every conformant implementation MUST support; Ed25519 for v0.1-draft.

**Principal.** An actor — human, AI model, tool, service, workflow, system, or external party — that participates in producing an event.

**Profile.** A versioned extension to Core defining vertical-specific event types, roles, and vocabulary.

**Provenance chain.** A hash-linked sequence of events from the same producer.

**Sink.** An implementation at compliance level L2 or higher; receives, stores, and serves events.

**Source.** The URI identifying the producer of an event.

**Publisher.** The organization that authored and published this specification; Edson Technologies, Inc. for v0.1-draft. Under the current posture (§7.6), the publisher is not actively operating a standards-body process. Governance terminology referring to a "stewardship body" or "Working Group" describes roles in a hypothetical future structure.

**Stewardship body.** A neutral standards body (such as Linux Foundation, Joint Development Foundation, OpenSSF, or IETF) that would assume governance of GAAIM if the specification is contributed to such a body in the future. No stewardship body is currently operating.

**Working Group.** In anticipated future governance, the body that would own Core, the profile registry, and the algorithm registry. Not currently constituted.

---

# Appendix E. Change Log

## v0.1-draft (April 5, 2026 — revised same day)

### Initial publication

- GAAIM Core v0.1-draft specified.
- GAAIM Engineering Profile v0.1-draft specified.
- Four compliance levels defined.
- Algorithm registry established with Ed25519 as MTI.
- Specification text published under CC-BY-SA 4.0; schemas and reference implementations under Apache 2.0.

### Same-day revisions (13 normative fixes)

**Critical interop fixes:**

- **Fix 1** — §4.2 canonical JSON rules corrected: absent attributes MUST NOT be synthesized as null (§4.2.1). Number-serialization note added (§4.2.2). Normative test vectors introduced (§4.2.3, Appendix A.3).
- **Fix 2+3** — §5.3.2 Key Registry API made normative: HTTPS `GET /v1/keys/{keyId}` + `/.well-known/gaaim-registry` discovery endpoint. §5.3.3 switched to prefix-encoded public keys (`spki-base64:`, `raw-base64url:`). `signaturekey` is now a URI-qualified HTTPS identifier binding events to specific registry instances (§5.3.2.1).
- **Fix 4** — `prev` promoted to REQUIRED at L1+. New `auditeligible` REQUIRED attribute. New §5.6.2 Chain Continuity Verification with chain-gap semantics.
- **Fix 5** — New §9.4.0 Reference Format Normalization. All Engineering Profile events updated to use prefix-encoded commit hashes, normalized repository URIs, and prefix-encoded artifact references.

**Security & conformance:**

- **Fix 6** — New §5.2.5 Migration Planning for Long-Horizon Integrity. Reserved prefixes for `ml-dsa:`, `slh-dsa:`, `fn-dsa:`, `hybrid:`. `migrationPlan` field in Key Registry records.
- **Fix 7** — §10.1 threat 4 updated with layered mitigation. New §10.4.1 Registry Origin Allowlisting (primary defense against registry substitution). §10.5 hardened with MUST requirements.
- **Fix 8** — L0 renamed from "Compatible" to "Development" (non-audit tier with segregation requirements).

**Coverage & positioning:**

- **Fix 9** — Engineering Profile catalog expanded from 8 to 15 events. Added AI workflow events (`ai.prompt.submitted`, `ai.response.received`, `ai.suggestion.accepted`, `ai.suggestion.rejected`, `codegen.applied`) and supply-chain events (`dependency.updated`, `security.scanned`).
- **Fix 10** — §3.6 event identifiers accept both ULID and UUIDv7 [RFC-9562].
- **Fix 11** — New §12.3 Related Work and Positioning with comparison matrix vs in-toto, SLSA, C2PA.
- **Fix 12** — §7.6 governance handoff accelerated: 12 months after v1.0 (was "by year 3"). Added §7.6.1 handoff terms and §7.6.2 timeline.
- **Fix 13** — New §3.5.6 Principal Identifier Forms: three forms (full, vendor-only, anonymized) declared per producer in Key Registry record.

---

**End of Specification — v0.1-draft (revised)**
