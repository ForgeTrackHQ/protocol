# Contributing to GAAIM

Thank you for your interest in contributing. GAAIM is published as a reference specification — technical feedback, implementation reports, typo fixes, and clarifications are all welcome.

This document describes how to contribute under the current published-reference posture. For the governance posture itself, see [GOVERNANCE.md](./GOVERNANCE.md) and [§7.6 of the specification](./gaaim-core-and-engineering-profile-v0_1.md#76-governance-posture).

---

## Expectations

Before contributing, understand the current posture:

- **Response times are best-effort.** No formal working group is operating, so there is no SLA on issues, pull requests, or discussions. Editorial PRs typically get addressed within a few weeks; normative proposals may take longer or may not be actioned at all.
- **Editorial contributions are most likely to be merged.** Typo fixes, clarifications, broken-link repairs, formatting improvements, and non-normative examples are welcome and generally get merged quickly.
- **Normative changes are evaluated against the current specification direction.** They may not be merged under the current posture, because opening formal revisions would require opening an active RFC process that Edson Technologies is not currently operating.
- **The v0.1-draft specification is stable as published.** It will not be revised except to correct errors; substantive changes would be made in a future v0.x or v1.0 revision.

If you want to propose a substantive change that would be merged rapidly, the honest answer is: **fork the specification and ship an implementation of your fork**. Real implementations are the strongest signal that a revision direction is right, and the specification's CC-BY-SA 4.0 license exists precisely to support this.

---

## Ways to Contribute

Four ways to contribute, ordered from lowest to highest ceremony:

### 1. Open a Discussion

Use [GitHub Discussions](https://github.com/GAAIM-standard/spec/discussions) for:

- Questions about the specification's intent or how to interpret specific sections
- Open-ended feedback that isn't a concrete change proposal
- Implementation reports — share what you're building, what worked, what didn't
- Ideas you want to workshop before filing an issue or PR

Discussions don't require a concrete proposal. This is the right starting point if you're not sure what category your contribution falls into.

### 2. File an Issue

Use [GitHub Issues](https://github.com/GAAIM-standard/spec/issues) for:

- Typos, broken links, or formatting problems
- Contradictions between sections
- Ambiguities in normative language (MUST/SHOULD/MAY)
- Incorrect examples
- Security concerns (but read [SECURITY.md](./SECURITY.md) first — some issues should not be filed publicly)

Use a clear title describing the problem. Quote the affected text or link to specific line numbers where possible.

### 3. Open a Pull Request

For editorial changes that can be merged without discussion:

- Typo fixes, grammar corrections, formatting improvements
- Broken-link repairs
- Clarifications that do not change normative meaning
- Non-normative examples

Process:

1. **Fork** the repository and create a branch named `fix/<short-description>` or `docs/<short-description>`
2. **Make your changes.** Keep commits focused; one logical change per commit
3. **Open a pull request** against `main` with a clear title and description
4. **Respond to review comments.** A maintainer will typically respond when they can; there is no guaranteed response time

Editorial PRs that turn out to propose substantive changes will be converted to discussions or left open pending a decision — don't take this personally, it just means the change is bigger than the editorial lane can accommodate.

### 4. Ship an Implementation

The most valuable contribution is a real implementation. If you ship a GAAIM-conformant producer, sink, or processor:

- Open a Discussion titled `[implementation] <your project>` with a brief description and a link
- Maintaining a list of known implementations in the README is planned

You do not need permission from anyone to implement GAAIM. Read the spec, emit conformant events, verify them, and ship.

---

## Specification Text Style

If you're submitting a PR that modifies specification text, these conventions apply:

### Normative language

The specification uses the normative keywords defined in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174):

- **MUST** / **MUST NOT** / **REQUIRED** / **SHALL** / **SHALL NOT** — absolute requirement or prohibition
- **SHOULD** / **SHOULD NOT** / **RECOMMENDED** — strong suggestion with documented exception paths
- **MAY** / **OPTIONAL** — truly optional, implementer's choice

Use these words in uppercase only when they are being used normatively. Lowercase "should" and "may" in ordinary prose do not carry normative weight.

Every MUST and SHOULD in the specification should be testable. If a reader cannot write a conformance check that verifies the statement, the statement is probably not normative and should be rephrased.

### Voice and tense

- Present tense for specification statements ("The producer signs the event" — not "will sign")
- Active voice where possible ("The verifier validates the signature" — not "The signature is validated")
- Define terms on first use, or link to the Glossary (Appendix D)
- Avoid "simply," "just," "obviously," "clearly" — they're never true for the reader who's stuck

### Examples

Examples are non-normative unless explicitly marked as normative. Prefer realistic examples over toy examples. Event payload examples should be valid JSON that a conformance checker could parse.

### Cross-references

Link to specific sections by their section number, not by page number:

```markdown
See [§5.3 Key Registry](./gaaim-core-and-engineering-profile-v0_1.md#53-key-registry)
```

---

## Commit Conventions

Use clear, imperative commit messages:

```
docs: fix typo in §5.3.2 key registry API
spec: clarify canonical serialization rule for absent attributes
fix: correct Ed25519 signature length in §5.2.2
```

Focused commits are easier to review. One logical change per commit.

---

## Code of Conduct

All participants in this project — Discussions, Issues, PRs, and any other project space — are expected to follow the [Code of Conduct](./CODE_OF_CONDUCT.md).

---

## Getting Help

- **Open-ended questions:** [GitHub Discussions](https://github.com/GAAIM-standard/spec/discussions)
- **Concrete problems:** [GitHub Issues](https://github.com/GAAIM-standard/spec/issues)
- **Private questions:** `spec@gaaim.org`
- **Security issues:** see [SECURITY.md](./SECURITY.md) — do not open public Issues for security vulnerabilities

Thank you for contributing to GAAIM.
