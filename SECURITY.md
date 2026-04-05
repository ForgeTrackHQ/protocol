<!--
  File: SECURITY.md
  Version: 0.1.001 | Path: ForgeTrackHQ/protocol/SECURITY.md
-->

# Security Policy

## Scope

This security policy covers:

- **The ForgeTrack Protocol specification itself** — flaws in the signing scheme, attribution model, chain-of-custody primitives, auth plane, or any other normative security property defined by the specification
- **Reference implementations** maintained under the [ForgeTrackHQ](https://github.com/ForgeTrackHQ) organization — including SDKs, adapters, and example code
- **Schema vulnerabilities** — ambiguities or definitions in the event schemas that could enable forgery, replay, injection, or impersonation

This policy does **not** cover:

- Third-party implementations of the protocol (report those to their maintainers)
- Commercial products or services that implement the protocol, including any built by the Steward (report those to the respective vendor through their own security disclosure process)
- Vulnerabilities in dependencies (please report those upstream; let us know if an upstream vulnerability affects our specification or reference code)

## Reporting a Vulnerability

**Do not open public GitHub issues for security vulnerabilities.**

Report vulnerabilities privately via one of these channels:

### Preferred: GitHub Security Advisory

Use GitHub's private vulnerability reporting feature:

1. Navigate to the [Security tab](https://github.com/ForgeTrackHQ/protocol/security) of this repository
2. Click "Report a vulnerability"
3. Fill out the form with as much detail as you can provide

This creates a private advisory visible only to maintainers, which we use to coordinate disclosure.

### Alternative: Email

Email **security@forgetrack.org** with:

- A description of the vulnerability
- The specification chapter/section or code file affected
- Steps to reproduce (if applicable)
- Potential impact
- Your preferred credit attribution (or request for anonymity)

If you need to send sensitive details, you may encrypt your email using our PGP key (to be published in v0.2). Until the PGP key is published, please send an initial unencrypted email and we will coordinate a secure channel.

## What to Expect

When you submit a report, you can expect:

| Stage | Timeline |
|---|---|
| **Initial acknowledgement** | Within 3 business days |
| **Preliminary assessment** | Within 7 business days — we confirm whether we consider it a vulnerability, and at what severity |
| **Fix development** | Depends on severity and complexity; we will communicate estimates |
| **Coordinated disclosure** | Typically 30–90 days from initial report, negotiated with reporter |
| **Public advisory** | Published after fix is available; reporter is credited unless they request otherwise |

We will keep you informed throughout the process. If you don't hear from us within the acknowledgement window, please follow up — occasionally emails go missing.

## Severity Classification

We use the following internal severity scale:

| Severity | Description | Examples |
|---|---|---|
| **Critical** | Breaks a core security guarantee of the protocol | Signature forgery, attribution spoofing, cross-tenant event access |
| **High** | Enables significant attack with realistic prerequisites | Replay attacks, chain verification bypass, auth scope escalation |
| **Medium** | Enables attack requiring specific conditions | Information disclosure via side channel, enumeration vulnerability |
| **Low** | Best-practice violation with limited real impact | Suboptimal default configuration, missing defense-in-depth |
| **Informational** | Not a vulnerability, but worth documenting | Clarification needed, potential future concern |

## Specification Vulnerabilities vs. Implementation Vulnerabilities

Because ForgeTrack is both a specification and a set of reference implementations, vulnerability reports fall into two categories:

### Specification vulnerabilities

A flaw in the specification itself — meaning a conforming implementation would still be vulnerable. These are the most serious type of report we can receive, because they require coordinated updates across all implementations.

When we confirm a specification vulnerability:

1. We assign a severity and create a private advisory
2. We draft a corrective specification change (typically via an emergency RFC with shortened review period)
3. We notify all known Reference Implementers privately and coordinate a joint disclosure timeline
4. We publish the advisory, specification update, and migration guidance simultaneously
5. We issue a CVE where applicable

### Reference implementation vulnerabilities

A flaw in our own SDK, adapter, or example code that is not required by the specification. These follow standard software vulnerability disclosure:

1. We assign a severity and create a private advisory
2. We develop and test a patch
3. We release a patched version
4. We publish the advisory with CVE

## Safe Harbor

We commit to not pursuing legal action against security researchers who:

- Make a good-faith effort to comply with this policy
- Avoid privacy violations, destruction of data, and interruption of services
- Do not exploit a vulnerability beyond what is necessary to confirm its existence
- Report the vulnerability promptly after discovery
- Do not disclose the vulnerability publicly before coordinated disclosure

This safe harbor applies to research conducted against the specification and against reference implementations maintained in the ForgeTrackHQ organization. It does not extend to testing against commercial products or services that implement the protocol — those are covered by each vendor's own security policy.

## Recognition

Security researchers who report confirmed vulnerabilities will be:

- Credited in the public advisory (unless they request anonymity)
- Listed in a `SECURITY-CREDITS.md` file (to be created after the first acknowledged report)
- Acknowledged in release notes for any specification or implementation fix

We do not currently operate a bug bounty program. If we establish one in the future, we will update this policy.

## Questions

For non-security questions about the specification's security properties, open a [Discussion](https://github.com/ForgeTrackHQ/protocol/discussions) labeled `security`.

For questions about this policy itself, email **security@forgetrack.org**.

---

*This security policy is at v0.1-draft alongside the specification. We expect to refine it based on real-world disclosure experience.*
