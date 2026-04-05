# Security Policy

## Scope

This security policy covers:

- **The GAAIM specification itself** — flaws in the signing scheme, attribution model, chain-of-custody primitives, or any other normative security property defined by the specification
- **Reference implementations** maintained under the [GAAIM-standard](https://github.com/GAAIM-standard) organization — including any future SDKs, verifiers, canonicalization libraries, and example code
- **Schema vulnerabilities** — ambiguities or definitions in the event schemas that could enable forgery, replay, injection, or impersonation

This policy does **not** cover:

- Third-party implementations of the specification (report those to their maintainers)
- The ForgeTrack SaaS product (see the product's separate disclosure policy at [forgetrack.com/security](https://forgetrack.com/security))
- Vulnerabilities in dependencies (please report those upstream; let us know if an upstream vulnerability affects the specification or reference code)

---

## Reporting a Vulnerability

**Do not open public GitHub issues for security vulnerabilities.**

Report vulnerabilities privately via one of these channels:

### Preferred: GitHub Security Advisory

Use GitHub's private vulnerability reporting feature:

1. Navigate to the [Security tab](https://github.com/GAAIM-standard/spec/security) of this repository
2. Click "Report a vulnerability"
3. Fill out the form with as much detail as you can provide

This creates a private advisory visible only to maintainers, which we use to coordinate disclosure.

### Alternative: Email

Email **security@gaaim.org** with:

- A description of the vulnerability
- The specification section or code file affected
- Steps to reproduce (if applicable)
- Potential impact
- Your preferred credit attribution (or request for anonymity)

If you need to send sensitive details, please send an initial unencrypted email and we will coordinate a secure channel.

---

## What to Expect

Given the published-reference posture of GAAIM (no formal working group currently operating), response times are best-effort rather than contractual. When you submit a report, you can expect:

| Stage                       | Target                                                       |
| --------------------------- | ------------------------------------------------------------ |
| **Initial acknowledgement** | Within 5 business days                                       |
| **Preliminary assessment**  | Within 14 business days — we confirm whether we consider it a vulnerability, and at what severity |
| **Fix development**         | Depends on severity and complexity; we will communicate estimates |
| **Coordinated disclosure**  | Typically 30–90 days from initial report, negotiated with reporter |
| **Public advisory**         | Published after fix is available; reporter is credited unless they request otherwise |

We will keep you informed throughout the process. If you don't hear from us within the acknowledgement window, please follow up — occasionally emails go missing.

---

## Severity Classification

We use the following internal severity scale:

| Severity          | Description                                             | Examples                                                     |
| ----------------- | ------------------------------------------------------- | ------------------------------------------------------------ |
| **Critical**      | Breaks a core security guarantee of the specification   | Signature forgery, attribution spoofing, cross-tenant event access |
| **High**          | Enables significant attack with realistic prerequisites | Replay attacks, chain verification bypass, registry substitution |
| **Medium**        | Enables attack requiring specific conditions            | Information disclosure via side channel, enumeration vulnerability |
| **Low**           | Best-practice violation with limited real impact        | Suboptimal default configuration, missing defense-in-depth   |
| **Informational** | Not a vulnerability, but worth documenting              | Clarification needed, potential future concern               |

---

## Specification Vulnerabilities vs. Implementation Vulnerabilities

Because GAAIM is both a specification and (planned) reference implementations, vulnerability reports fall into two categories:

### Specification vulnerabilities

A flaw in the specification itself — meaning a conforming implementation would still be vulnerable. These are the most serious type of report we can receive, because they require coordinated updates across all implementations.

When we confirm a specification vulnerability:

1. We assign a severity and create a private advisory
2. We draft a corrective specification change
3. We notify known implementers privately and coordinate a joint disclosure timeline
4. We publish the advisory, specification update, and migration guidance simultaneously
5. We issue a CVE where applicable

Note: under the current published-reference posture, "known implementers" may be a short list. If you are shipping a GAAIM-conformant implementation, opening a Discussion to register as a known implementer is the best way to ensure you receive security notifications.

### Reference implementation vulnerabilities

A flaw in a reference SDK, verifier, or example code that is not required by the specification. These follow standard software vulnerability disclosure:

1. We assign a severity and create a private advisory
2. We develop and test a patch
3. We release a patched version
4. We publish the advisory with CVE

---

## Safe Harbor

We commit to not pursuing legal action against security researchers who:

- Make a good-faith effort to comply with this policy
- Avoid privacy violations, destruction of data, and interruption of services
- Do not exploit a vulnerability beyond what is necessary to confirm its existence
- Report the vulnerability promptly after discovery
- Do not disclose the vulnerability publicly before coordinated disclosure

This safe harbor applies to research conducted against the specification and against reference implementations maintained in the GAAIM-standard organization. It does not extend to testing against the ForgeTrack SaaS product — see that product's separate security policy.

---

## Recognition

Security researchers who report confirmed vulnerabilities will be:

- Credited in the public advisory (unless they request anonymity)
- Acknowledged in release notes for any specification or implementation fix

We do not currently operate a bug bounty program.

---

## Questions

For non-security questions about the specification's security properties, open a [Discussion](https://github.com/GAAIM-standard/spec/discussions) labeled `security`.

For questions about this policy itself, email **security@gaaim.org**.

---

*This security policy is at v0.1-draft alongside the specification. It will be refined based on real-world disclosure experience.*
