# Security Policy

## Supported versions

| Version | Supported |
|---------|-----------|
| v0.1.x  | Yes       |
| < 0.1   | No        |

## Reporting a vulnerability

If you believe you have found a security issue in the SemanticMap spec
itself, in the example documents, in CI tooling, or in any release artifact:

- **Do not** open a public GitHub issue.
- Email `security@unboxapi.pro` with a clear description and any
  proof-of-concept. v0.1.0 accepts plaintext email; a PGP release key
  (`docs/pgp.asc`) ships in v0.1.1 once Founder + CTO have minted and
  cross-signed it. Until then, please mark sensitive details as such in the
  subject line so triage can route appropriately.
- We will acknowledge within **2 business days** and aim to provide an
  assessment within **5 business days**.
- Coordinated disclosure window: **90 days** from acknowledgment, extended by
  mutual agreement if a fix requires more time.

CTO is the first responder. CEO is informed of any High/Critical report
within 24 hours of triage.

## Threat model summary

The full threat-model memo lives on the source issue and covers:

- Schema-injection by malicious SemanticMap producers (T1).
- Parser abuse against consumer YAML/JSON parsers (T2).
- Supply-chain integrity of this repository's published artifacts (T3).
- Accidental information disclosure (T4).
- Abuse of the open repository (T5).

A SemanticMap document is **data**, not code. Consumers must treat untrusted
SemanticMap documents the same way they treat any other untrusted input.

## Hardening guarantees on this repository

- Branch protection on `main`: required PR review, required CI status checks,
  no direct pushes, no force-push, linear history.
- Required signed commits (Sigstore `gitsign` or GPG). Release tags signed.
- Sigstore artifact attestation on every release.
- CycloneDX SBOM published as a release asset.
- CODEOWNERS requires CTO review on every PR.
- Dependabot, GitHub secret scanning, and GitHub Advanced Security code
  scanning enabled.
- `gitleaks` runs on every PR and on the full commit range at release.
