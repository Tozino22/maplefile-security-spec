# MapleFile — Security Controls & AI Governance Specification

A worked example of **specifying security and AI-governance controls before a line of
code is written** — for a Canadian AI tax assistant handling SIN, banking details and
financial statements.

Built by **Tosin Aina** — CISSP, CISA, CRISC, ISO/IEC 27001 Lead Auditor, ISACA AAISM.

---

## Why this repository exists

Most security documentation is written after the system is built, to satisfy an auditor.
This is the opposite: a control specification produced at design time, where the controls
still shape the architecture.

**The application has not been built.** This repository is the specification, published
as a worked example of the method.

## What it covers

| Section | Content |
|---|---|
| Data classification | Three tiers — CRITICAL (SIN, bank accounts, credentials), SENSITIVE, INTERNAL — with proportionate baseline controls |
| **PII & SIN redaction pipeline** | The central control: detect → redact/tokenize → validate locally → controlled reveal. SIN checksum validation happens locally and is never sent to a model |
| Encryption | At rest, in transit, and field-level for CRITICAL data |
| Authentication | OIDC, short-lived tokens, HttpOnly/Secure/SameSite cookies, mandatory MFA, passkeys/WebAuthn, step-up auth |
| Authorization | RBAC, Postgres row-level security enforced at the database rather than in application code, and **retrieval-time authorization applied before documents enter the context window** |
| AI-specific security | Mapped to the OWASP Top 10 for LLM Applications |
| AI governance | Deterministic tax computation with the LLM confined to explanation; human-in-the-loop for consequential actions; full prompt/retrieval/tool-call traceability; model and knowledge-base version provenance |
| Privacy by province | PIPEDA, Quebec Law 25 and provincial regimes — the strictest applicable rule is applied |
| Control-to-framework mapping | Every control traced to NIST AI RMF, ISO/IEC 42001, OWASP LLM Top 10, MITRE ATLAS and the applicable privacy law |
| Pre-launch checklist | Go/no-go security gate |

## The control worth reading

**Section 3 — the redaction pipeline.** It is the reason an LLM can be used on tax data
at all: sensitive identifiers are tokenized before any model call, the SIN checksum is
validated locally and never transmitted, and original values are revealed only to
authorized users under row-level security with a full audit trail.

## Read it

[`SECURITY_CONTROLS_AND_AI_GOVERNANCE.md`](SECURITY_CONTROLS_AND_AI_GOVERNANCE.md)

## Licence

MIT.
