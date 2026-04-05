<!--
  File: GOVERNANCE.md
  Version: 0.1.003 | Path: ForgeTrackHQ/protocol/GOVERNANCE.md
-->

# Governance

This document describes how decisions are made about the ForgeTrack Protocol specification, who makes them, and how that authority is expected to evolve over time.

The governance model is deliberately minimal at v0.1-draft. A specification with no implementers needs a single clear steward, a published RFC process, and a credible commitment to broaden governance as adoption warrants. It does not need a foundation, a technical steering committee, or an elected board — those structures become meaningful only once there are enough independent stakeholders for representation to matter.

---

## 1. Stewardship Model

The ForgeTrack Protocol is currently in **Phase 1: Single-Organization Stewardship**.

**The current Steward is Edson Technologies Inc.** ("the Steward"). The Steward is responsible for:

- Accepting, reviewing, and merging contributions to the specification
- Running the RFC process and deciding on RFC outcomes
- Cutting releases and maintaining the `main` branch
- Representing the specification to external parties (auditors, standards bodies, press)
- Ensuring the specification repository remains available, licensed as documented, and free of vendor lock-in

The Steward does **not** own the specification. The specification text is licensed under CC-BY-SA 4.0 and code is licensed under Apache 2.0, both of which are irrevocable. Anyone may fork the specification at any time. The Steward's authority exists only insofar as the community continues to recognize the canonical repository as the canonical repository.

### 1.1 Why a single-organization steward at v0.1

Distributed governance adds value once there are multiple independent parties with a stake in the outcome. At v0.1-draft, the ForgeTrack Protocol has one implementer (a commercial product built by the Steward) and is soliciting its first external review. Imposing a multi-stakeholder governance structure before stakeholders exist would be ceremonial rather than functional.

Single-organization stewardship carries known risks — vendor capture, specification drift toward one implementer's needs, loss of credibility with enterprise adopters. These risks are addressed through:

- **Public specification text** under an irrevocable open license, so the specification cannot be clawed back
- **Public RFC process** with documented comment periods, so specification changes are visible and contestable
- **Published phase transition criteria** (Section 4), so the path out of single-stewardship is committed to in writing
- **Conflict-of-interest disclosures** (Section 5), so Steward actions that benefit the Steward's commercial product are visible
- **A documented fork policy** (Section 6), so the community retains credible exit if the Steward acts against the specification's interests

---

## 2. Decision-Making Process

Decisions about the specification fall into four categories, each with a different process.

### 2.1 Editorial changes

Typo fixes, grammar corrections, broken-link repairs, clarifications that do not change normative meaning, and formatting improvements.

- **Process:** Pull request. Any contributor may open one.
- **Approval:** Any Maintainer may merge.
- **Comment period:** None required.

### 2.2 Substantive non-breaking changes

New optional fields, new adapter specifications, new appendix content, new examples, additional non-normative guidance, clarifications that resolve ambiguity without changing behavior.

- **Process:** RFC with `substantive` label (see CONTRIBUTING.md).
- **Approval:** Steward decision after minimum 14-day public comment period.
- **Required review:** At least one Maintainer other than the RFC author must approve.

### 2.3 Breaking changes

Any change that would cause a conforming implementation of version N to become non-conforming under version N+1. Includes: removing fields, renaming fields, changing field semantics, changing signing algorithms, changing required-vs-optional status of existing fields, changing wire format.

- **Process:** RFC with `breaking-change` label.
- **Approval:** Steward decision after minimum 30-day public comment period.
- **Required review:** At least two Maintainers other than the RFC author must approve, and at least one Reference Implementer (once any exist) must have commented.
- **Documentation:** Migration guidance MUST be included in the RFC and in the CHANGELOG entry.

During the `-draft` phase (v0.x), breaking changes are expected and do not require Major version bumps. After v1.0, breaking changes require a Major version bump per semantic versioning.

### 2.4 Governance changes

Amendments to this document, changes to the RFC process in CONTRIBUTING.md, changes to the license structure, changes to phase transition criteria, Steward succession.

- **Process:** RFC with `governance` label.
- **Approval:** Steward decision after minimum 30-day public comment period. Governance RFCs cannot be approved during the same comment window they were opened in — a minimum of 7 days must elapse between the close of comments and the merge decision, to allow for last-minute objections.
- **Required review:** At least two Maintainers must approve. Objections from any Reference Implementer must be addressed in writing before merge.

---

## 3. Roles

The specification recognizes four roles during Phase 1. Additional roles are expected in later phases (see Section 4).

### 3.1 Steward

The organization responsible for maintaining the canonical repository. There is exactly one Steward at any time. Steward succession requires a Governance RFC (Section 2.4).

During Phase 1, the Steward is Edson Technologies Inc.

### 3.2 Maintainers

Individuals granted commit access to the canonical repository. Maintainers review pull requests, shepherd RFCs, and have authority to merge editorial changes without RFC.

Maintainers are appointed by the Steward. The initial Maintainer set is published in MAINTAINERS.md (added when the second Maintainer is appointed). Maintainers may resign at any time by opening a pull request removing themselves.

Maintainers are expected to disclose any commercial affiliation that could create a conflict of interest under Section 5.

### 3.3 Reference Implementers

Organizations or individuals who maintain a publicly available, actively developed implementation of the protocol. Reference Implementer status is self-declared via pull request to the `IMPLEMENTERS.md` file (added when the first external implementation is declared), and confirmed by the Steward based on the implementation being real, public, and substantively conforming.

Reference Implementers have standing to:

- Be consulted on any `breaking-change` RFC (Section 2.3)
- Object to `governance` RFCs (Section 2.4)
- Request phase transition review (Section 4)

### 3.4 Contributors

Anyone who has had a pull request or RFC merged. Contributors are recognized in release notes and in `CONTRIBUTORS.md` (added with the first external contribution). No approval is required to become a Contributor — submitting a merged change is sufficient.

---

## 4. Phase Transition Criteria

The specification's governance model is expected to evolve through three phases. The criteria for each transition are documented below and are themselves subject only to Governance RFC (Section 2.4).

### Phase 1 → Phase 2: Community Governance

Phase 2 introduces a Technical Steering Committee (TSC) with representation from multiple implementing organizations. The Steward retains final authority on governance changes and license matters but delegates technical decisions to the TSC.

**Transition requires all of:**

- At least three independent Reference Implementers, where "independent" means not controlled by or financially dependent on the Steward
- At least one production deployment of a Reference Implementation outside the Steward's own products
- A published conformance test suite
- Specification has reached v1.0 (stable)
- Steward has published a TSC charter proposal as a Governance RFC

**Target timeframe:** 12–24 months from v0.1-draft, contingent on adoption.

### Phase 2 → Phase 3: Foundation Governance

Phase 3 transfers stewardship of the specification to a neutral foundation (e.g., the Linux Foundation, CNCF, Apache Software Foundation, or a purpose-formed foundation). The foundation owns the canonical repository, the trademark (if any), and the decision-making process. The original Steward becomes one participant among many.

**Transition requires all of:**

- Phase 2 TSC has been operational for at least 12 months
- At least two Major version releases have been cut under TSC governance
- A suitable foundation has been identified and has accepted a transfer proposal
- Trademark assignment (if applicable) has been legally structured
- A Governance RFC authorizing the transfer has passed with no sustained objections from Reference Implementers

**Target timeframe:** conditional on Phase 2 maturation; no fixed date.

### Blocking criteria

A phase transition MUST NOT occur if:

- The Steward or TSC has an outstanding unresolved conflict-of-interest disclosure (Section 5)
- A `breaking-change` RFC is mid-comment-period
- The specification is in a state where conformance is ambiguous (e.g., contradictory normative statements)

---

## 5. Conflict of Interest

The Steward operates a commercial product built on this specification. This creates an inherent conflict of interest: specification changes can benefit or harm the Steward's commercial interests. This section documents how that conflict is managed during Phase 1.

### 5.1 Disclosure requirement

Any RFC authored or championed by a Maintainer affiliated with the Steward that would materially affect the Steward's commercial product MUST include a "Commercial Impact" section disclosing:

- How the change affects the Steward's product
- Whether the change was prompted by a commercial customer request
- Whether alternative designs were considered and why they were rejected

### 5.2 Recusal

Maintainers MAY recuse themselves from reviewing RFCs where they have a commercial interest. Recusal is encouraged but not required during Phase 1, since all current Maintainers are expected to be affiliated with the Steward.

### 5.3 Objections on conflict grounds

Any Reference Implementer or Contributor may file a conflict-of-interest objection to an RFC. The objection MUST be addressed in writing by the Steward before the RFC is merged. "Addressed" means substantively responded to, not necessarily resolved in the objector's favor.

### 5.4 External audit

Beginning in Phase 2, the TSC SHOULD commission an annual governance audit by a party not affiliated with the Steward. The audit reviews whether the RFC process was followed, whether conflict disclosures were made, and whether objections were addressed. Audit findings are published.

---

## 6. Fork Policy

The specification text (CC-BY-SA 4.0) and code (Apache 2.0) are permanently, irrevocably licensed. Anyone may fork the specification at any time, for any reason, without notice.

The Steward commits to:

- Not pursuing trademark, copyright, or other legal action against good-faith forks that use the specification content under its license terms
- Linking to notable forks in `FORKS.md` (added if and when a notable fork exists) so users can find them
- Not claiming the name "ForgeTrack Protocol" for forked specifications, and asking fork maintainers to choose a distinct name for their forked specification to avoid confusion (this is a naming request, not a legal demand — the licenses do not restrict naming)

If the Steward is acquired, becomes insolvent, or otherwise ceases to be a credible steward, Reference Implementers and Maintainers are expected to coordinate on a successor steward via a Governance RFC. If no RFC can be merged because the Steward is unresponsive, Reference Implementers MAY declare a fork the new canonical repository by joint statement, and this document recognizes such a declaration as valid.

---

## 7. Amendments

This document may be amended only through a Governance RFC (Section 2.4). Amendments become effective when merged to `main` and take effect for all subsequent RFCs; they do not retroactively change the process for RFCs already in-flight.

The version of this document in effect for any given RFC is the version that was on `main` at the time the RFC was opened.

---

## 8. Contact

Questions about governance:

- **Public:** [GitHub Discussions](https://github.com/ForgeTrackHQ/protocol/discussions) in the "Governance" category
- **Private:** `protocol@forgetrack.org`

Security issues: see [SECURITY.md](./SECURITY.md).

Code of conduct violations: see [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md).
