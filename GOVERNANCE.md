# Governance

This document describes the current governance posture of GAAIM. For the full normative text, see [§7.6 of the specification](./gaaim-core-and-engineering-profile-v0_1.md#76-governance-posture).

---

## Current Posture: Published Reference

GAAIM is published as a **reference specification**. **No formal working group is currently operating**, and Edson Technologies, Inc. is not actively running a standards process around it.

**Edson Technologies is the publisher** of this specification. The company authored the text, publishes it openly under the licenses declared in the front matter, and maintains the canonical repository at `https://github.com/GAAIM-standard/spec`.

**ForgeTrack is the first implementation.** The event model is in production use inside [ForgeTrack](https://forgetrack.io), a product that tracks AI-amplified engineering work for R&D credit substantiation, audit trails, and contribution reporting.

**What this means practically:**

- Issues and pull requests on GitHub are welcome, but **response times are not guaranteed**.
- There is no RFC process in flight.
- There is no ratified profile registry, no algorithm-registry governance process, no ratified conformance certification program, and no active profile subcommittee.
- Editorial PRs (typos, clarifications, broken links) are most likely to be merged quickly. Normative changes will be evaluated against the current specification direction but may not be merged under the current posture.

---

## Licensing Is Irrevocable

The specification text is licensed under [CC-BY-SA 4.0](./LICENSE-SPEC). Code, schemas, and example content are licensed under [Apache 2.0](./LICENSE-CODE). Both licenses are permanent and irrevocable.

**Anyone may fork the specification at any time, for any reason, without notice.** The publisher's authority exists only insofar as the community continues to recognize this repository as the canonical repository. If the publisher ever acts against the specification's interest, the specification can be forked and continued independently.

---

## Path to Active Stewardship

If independent implementer interest emerges — multiple parties shipping GAAIM-conformant producers or sinks, interop requests, or demand from regulators and auditors — Edson Technologies intends to contribute the specification, trademarks, and registries to a neutral standards body (such as the Linux Foundation, Joint Development Foundation, OpenSSF, IETF, or equivalent) at that time.

Edson Technologies does not intend to operate a long-term independent standards process. Contribution to a neutral body is the intended long-term governance outcome, not continued single-organization stewardship.

Until such contribution occurs, references in the specification to a "stewardship body," "Working Group," or "subcommittee" describe roles in a hypothetical future governance structure, not actors currently operating.

---

## Anticipatory Contribution Terms

If and when governance is contributed to a neutral body, the contribution is anticipated to transfer:

- **Specification text.** Copyright assignment under CC-BY-SA 4.0 (already the publication license).
- **Trademark.** Assignment of the "GAAIM" trademark and wordmark.
- **Registries.** Operational control of the algorithm registry, the profile registry, and the schema registry.
- **Certification program.** If a conformance certification program has been established by the contribution date, the receiving body assumes its operation.
- **Repositories.** Administrative control of `github.com/GAAIM-standard/*` repositories.

These are anticipatory terms, not binding commitments. The specific terms of any future contribution will be negotiated with the receiving body at that time.

---

## Fork Policy

Edson Technologies commits to:

- Not pursuing trademark, copyright, or other legal action against good-faith forks that use the specification content under its license terms
- Not claiming the GAAIM name for forked specifications, and asking fork maintainers to choose a distinct name for their forked specification to avoid confusion (a naming request, not a legal demand — the licenses do not restrict naming)

If Edson Technologies is acquired, becomes insolvent, or otherwise ceases to be a credible publisher, any party using the specification may continue to do so under the existing licenses, and may declare a successor canonical repository by public statement.

---

## Contact

- **Public:** [GitHub Discussions](https://github.com/GAAIM-standard/spec/discussions)
- **Private:** `spec@gaaim.org`
- **Security:** see [SECURITY.md](./SECURITY.md)
- **Code of conduct:** see [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)
