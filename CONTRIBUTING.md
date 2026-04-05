<!--
  File: CONTRIBUTING.md
  Version: 0.1.003 | Path: ForgeTrackHQ/protocol/CONTRIBUTING.md
-->

# Contributing to the ForgeTrack Protocol

Thank you for your interest in contributing. This specification is published as a Request for Comment, and feedback of all kinds is actively wanted — from typo corrections to substantive objections to entire alternative designs.

This document describes how to contribute. For who makes final decisions and how that authority is structured, see [GOVERNANCE.md](./GOVERNANCE.md).

---

## 1. Ways to Contribute

There are five ways to contribute, ordered from lowest to highest ceremony:

1. **Open a GitHub Discussion** for open-ended feedback, questions, or ideas you want to workshop before formalizing. Discussions don't require a concrete proposal.
2. **File a GitHub Issue** for concrete, reproducible problems: typos, broken links, contradictions between chapters, ambiguities in normative language, incorrect examples.
3. **Open a Pull Request** for editorial changes that any Maintainer can merge without process — typo fixes, grammar, formatting, broken-link repairs, clarifications that don't change normative meaning.
4. **Open an RFC** for any change that adds, removes, or modifies the specification's substance. This includes new fields, new adapters, new appendix content, changed defaults, and any breaking change. The RFC process is described in detail in Section 4.
5. **Publish an implementation** of the protocol. Reference Implementers have standing in the governance process (see GOVERNANCE.md §3.3). Declaring your implementation is a pull request against `IMPLEMENTERS.md` (that file will be created with the first declaration).

If you're not sure which path fits your contribution, open a Discussion first. It's better to talk through scope for five minutes than to write a full RFC that turns out to be solving the wrong problem.

---

## 2. Before You Start

A few ground rules that save time for everyone:

**Read the relevant chapter first.** The specification is long. If you're proposing a change to authentication, skim Part IV. If you're proposing a new adapter, skim Part VI. Many proposals turn out to already be addressed somewhere; many objections turn out to be answered in a chapter the objector hadn't yet read.

**Search existing Discussions, Issues, and RFCs.** Duplicate proposals waste reviewer time. If your idea has been raised before, add to the existing thread rather than starting a new one.

**Prefer small scope.** A proposal that changes one field is far more likely to be merged than a proposal that restructures Part V. If your idea requires large changes, consider whether it can be decomposed into independent smaller proposals.

**Expect to be asked "why."** Every normative statement in the specification has a cost — it's another thing implementers must get right, another conformance check, another thing that could be wrong. New normative statements need justification proportional to the burden they impose.

---

## 3. The Pull Request Process (Editorial Changes)

For changes that any Maintainer can merge without an RFC — typo fixes, grammar, formatting, broken-link repair, non-normative clarifications — the process is:

1. **Fork** the repository and create a branch named `fix/<short-description>` or `docs/<short-description>`.
2. **Make your changes.** Keep commits focused; one logical change per commit.
3. **Sign off your commits** with `git commit -s` (see Section 6 on DCO).
4. **Open a pull request** against `main` with a clear title and description. Reference any related Issues.
5. **Respond to review comments.** A Maintainer will typically respond within seven days. If no response after seven days, tag `@ForgeTrackHQ/maintainers` on the PR.
6. **Merge.** Once approved, a Maintainer will squash-merge the PR.

Editorial PRs that turn out to propose substantive changes will be converted to RFCs by a Maintainer — don't take this personally, it just means the change needs a wider review window.

---

## 4. The RFC Process

The RFC process is how substantive changes to the specification get made. It is modeled on the Rust RFC process and the Python PEP process, with modifications appropriate to a v0.1-draft specification.

### 4.1 When to open an RFC

Open an RFC for:

- New fields, new event types, new adapter specifications, new appendix content
- Changes to default values, required-vs-optional status, or normative language
- Changes to the authentication plane, signing scheme, or provenance model
- Any change that would require conforming implementations to change their code
- Governance changes (amendments to GOVERNANCE.md or this document)

Don't open an RFC for:

- Typos, grammar, formatting (use a PR)
- Questions about the specification (use a Discussion)
- Bugs in examples that don't affect normative meaning (use an Issue or PR)

If unsure, open a Discussion first.

### 4.2 RFC template

RFCs are opened as pull requests that add a file to the `rfcs/` directory (the directory will be created with the first RFC). The file is named `NNNN-short-title.md` where `NNNN` is the next available four-digit RFC number.

Every RFC must contain these sections:

```markdown
# RFC NNNN: <Title>

- **Author(s):** <name and affiliation>
- **Status:** Draft
- **Opened:** YYYY-MM-DD
- **Target version:** v0.x or v1.y

## Summary

One paragraph. What does this RFC propose, in plain language?

## Motivation

Why is this change needed? What problem does it solve? What's the cost
of not making this change?

## Detailed Design

The full proposal. What exactly changes in the specification? Include
the proposed text in as close to normative form as possible. Reference
the specific chapters and sections being modified.

## Alternatives Considered

What other designs were evaluated? Why were they rejected? "None" is
rarely an acceptable answer.

## Drawbacks

What's the cost of adopting this? What becomes harder? What might break?
Who has to change their code?

## Commercial Impact

(Required only if the RFC author is affiliated with the Steward and the
change materially affects the Steward's commercial product. See
GOVERNANCE.md §5.1.)

## Unresolved Questions

What aspects of this RFC are still open? What would you want feedback
on during the comment period?

## Migration Guidance

(Required for `breaking-change` RFCs.) How should existing implementations
adapt? What's the migration path? Is there a transition period?
```

### 4.3 RFC labels

Every RFC receives at least one of these labels when it's opened:

| Label | Meaning | Comment period |
|---|---|---|
| `substantive` | Non-breaking change that adds or modifies specification content | 14 days minimum |
| `breaking-change` | Change that would cause a conforming v0.N implementation to become non-conforming under v0.N+1 | 30 days minimum |
| `governance` | Amendment to GOVERNANCE.md, CONTRIBUTING.md, or licensing | 30 days minimum |

Additional non-exclusive labels:

- `adapter` — affects Part VI (adapters)
- `auth` — affects Part IV (authentication plane)
- `data-model` — affects Part V (data model)
- `compliance` — affects Part X (compliance & reporting)
- `editorial` — applied by Maintainers when an RFC turns out not to be substantive after all

### 4.4 RFC lifecycle

An RFC moves through these states:

1. **Draft** — RFC is open for comment. The comment clock starts on the day the `substantive`, `breaking-change`, or `governance` label is applied, not the day the PR was opened.
2. **Final Comment Period (FCP)** — a Maintainer has announced an intent to merge, reject, or postpone. FCP lasts 7 days. New substantive comments during FCP can reset the clock.
3. **Accepted** — RFC is merged to `main`. The specification change is now part of the repository, targeted at the declared version.
4. **Rejected** — RFC is closed without merge. A Maintainer must leave a comment explaining the rejection rationale. Rejected RFCs may be reopened later if circumstances change, but should be reopened as a new RFC that addresses the original rejection reasons.
5. **Postponed** — RFC is closed with the intent to revisit later. Typically used when an RFC is premature (e.g., depends on work that hasn't happened yet).
6. **Withdrawn** — RFC author closes the RFC.

### 4.5 Who approves an RFC

Per GOVERNANCE.md §2, approval requirements are:

- **`substantive` RFCs:** At least one Maintainer other than the author must approve. Steward decision after 14-day minimum comment period.
- **`breaking-change` RFCs:** At least two Maintainers other than the author must approve, and at least one Reference Implementer (once any exist) must have commented. Steward decision after 30-day minimum comment period.
- **`governance` RFCs:** At least two Maintainers must approve. Objections from any Reference Implementer must be addressed in writing. Steward decision after 30-day minimum comment period, with a 7-day cooling-off period between comment close and merge decision.

### 4.6 Champions

An RFC author is implicitly the RFC's champion — the person responsible for responding to comments, revising the proposal, and shepherding it through the lifecycle. If the author becomes unavailable, the RFC may go stale. Any Contributor may volunteer to take over as champion by commenting on the RFC.

RFCs with no champion for 30 days will be moved to Postponed by a Maintainer.

---

## 5. Specification Text Style

The specification is read by auditors, compliance officers, security architects, and implementers. Clarity and consistency matter more than elegance. These conventions are enforced on all RFCs.

### 5.1 Normative language

The specification uses the normative keywords defined in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174):

- **MUST** / **MUST NOT** / **REQUIRED** / **SHALL** / **SHALL NOT** — absolute requirement or prohibition
- **SHOULD** / **SHOULD NOT** / **RECOMMENDED** — strong suggestion with documented exception paths
- **MAY** / **OPTIONAL** — truly optional, implementer's choice

Use these words in uppercase only when they are being used normatively. Lowercase "should" and "may" in ordinary prose do not carry normative weight.

Every MUST and SHOULD in the specification should be testable. If a reader cannot write a conformance check that verifies the statement, the statement is probably not normative and should be rephrased.

### 5.2 Voice and tense

- Present tense for specification statements ("The adapter sends an event" — not "will send" or "sends" / "sent").
- Active voice where possible ("The server validates the signature" — not "The signature is validated").
- Define terms on first use, or link to the Glossary (Appendix H).
- Don't use "simply," "just," "obviously," "clearly" — they're never true for the reader who's stuck.

### 5.3 Examples

Examples are non-normative unless explicitly marked as normative. Prefer realistic examples over toy examples. Use consistent identifiers across examples in the same chapter (e.g., always use `tenant_42` for the example tenant).

Event payload examples should be valid JSON that a conformance checker could parse. Truncated examples must say they are truncated.

### 5.4 Cross-references

Link to specific chapters and sections, not to page numbers. Use the form:

```markdown
See [Chapter 12: Canonical Event Store](./spec/forgetrack-protocol-v0.1-draft.md#chapter-12).
```

---

## 6. Commit Conventions and DCO

### 6.1 Commit message format

Use [Conventional Commits](https://www.conventionalcommits.org/) format:

```
<type>(<scope>): <short summary>

<optional body>

<optional footer>

Signed-off-by: Your Name <your.email@example.com>
```

Types: `fix`, `docs`, `feat`, `refactor`, `style`, `test`, `chore`.
Scope examples: `spec`, `part-iv`, `rfc-0003`, `governance`, `ci`.

Examples:

```
docs(part-iv): fix typo in §17.3 managed identity flow

fix(spec): correct Ed25519 signature length in §9.2

feat(rfcs): add RFC 0003 for multi-algorithm signing
```

### 6.2 Developer Certificate of Origin (DCO)

All contributions must be made under the [Developer Certificate of Origin 1.1](https://developercertificate.org/). This is a lightweight alternative to a Contributor License Agreement (CLA) that does not require a separate signing process.

Signing off a commit certifies that you wrote the contribution, or have the right to submit it, and that you agree to its being distributed under the repository's license (CC-BY-SA 4.0 for specification text, Apache 2.0 for code).

To sign off a commit:

```bash
git commit -s -m "docs(part-v): clarify attribution confidence semantics"
```

The `-s` flag adds the `Signed-off-by` trailer automatically. You can configure git to sign off automatically:

```bash
git config --global format.signOff true
```

Pull requests with commits missing sign-off will fail DCO check and cannot be merged until signed-off commits are pushed.

### 6.3 Corporate contributions

If you are contributing on behalf of an employer, ensure you have authorization to do so. Your sign-off represents that you have this authorization. Some organizations require a corporate CLA for all open-source contributions — check your employer's policy before signing off.

---

## 7. Code of Conduct

All participants in this project — Discussions, Issues, PRs, RFCs, and any other project space — are expected to follow the [Code of Conduct](./CODE_OF_CONDUCT.md). Violations can be reported per the process in that document.

---

## 8. Getting Help

- **Open-ended questions:** [GitHub Discussions](https://github.com/ForgeTrackHQ/protocol/discussions)
- **Concrete problems:** [GitHub Issues](https://github.com/ForgeTrackHQ/protocol/issues)
- **Private questions:** `protocol@forgetrack.org`
- **Security issues:** see [SECURITY.md](./SECURITY.md) — do not open public Issues for security vulnerabilities

Thank you for contributing to the ForgeTrack Protocol.
