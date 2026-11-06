# MapleFile — Security Controls & AI Governance Specification

*Canadian AI tax assistant · all provinces and territories · working title*

| | |
|---|---|
| **Document** | Security Controls & AI Governance Specification (2 of 2) |
| **Version** | 0.1 — draft for review |
| **Author** | Oluwatosin (Tosin) Aina |
| **Classification** | Public — published as a portfolio excerpt |
| **Frameworks applied** | NIST AI RMF · ISO/IEC 42001 · OWASP LLM Top 10 · MITRE ATLAS · PIPEDA / Quebec Law 25 / AIDA |

> This is a design specification for an application that has not been built. It is
> published to show how security and AI-governance controls are specified **before**
> implementation, not to describe a running system.

---

## 1. Purpose & Scope

This document specifies the security and AI-governance controls for MapleFile and maps them to recognised frameworks. Controls are designed in, not added later. It is the companion to Document 1 (Architecture & Features).

## 2. Data Classification

Controls are proportionate to data sensitivity. Three tiers:

| Tier | Examples | Baseline controls |
|---|---|---|
| CRITICAL | SIN, bank account numbers, credentials | Redact from AI; field-level encrypt; strict RLS; full audit |
| SENSITIVE | Income, financial statements, addresses, T-slips | Encrypt at rest; RLS; least-privilege access |
| INTERNAL | Tax guides, product content | Standard access control |


## 3. PII & SIN Redaction Pipeline

The highest-value control, and the reason the AI is safe to use on tax data.
- Detect. Detect SIN and PII with pattern rules + named-entity recognition (Presidio / Azure or GCP DLP) at every ingest point and before any LLM call.
- Redact / tokenize. Replace SIN with a reversible token; mask PII not needed for the task. The LLM only ever receives redacted text.
- Validate locally. SIN validation uses the checksum locally — never sent to a model.
- Controlled reveal. Original values are decrypted and shown only to authorized users, under RLS, with access logged.

| PRIVACY   SIN is treated per the Office of the Privacy Commissioner guidance: collect only when legally required, never use it as a general identifier, and protect it as CRITICAL. |
|---|


## 4. Encryption

| Layer | Control |
|---|---|
| In transit | TLS 1.2+ everywhere; HSTS; no mixed content |
| At rest (database) | AES-256 full-database encryption |
| At rest (fields) | Field-level encryption for SIN & financial columns (envelope encryption) |
| Documents | Encrypted object storage; per-tenant separation |
| Key management | Cloud KMS / Key Vault; rotation; separation of duties; keys never in code |
| Backups | Encrypted; residency-bound; tested restore |


## 5. Authentication (Identity)
- Standards. OIDC-based login; short-lived access tokens + refresh tokens in HttpOnly, Secure, SameSite cookies.
- MFA + step-up. MFA required; passkeys/WebAuthn offered; step-up auth for sensitive actions (viewing SIN, filing).
- Sessions. Session management: idle + absolute timeouts; revocation; device/session visibility.

## 6. Authorization
- RBAC. Role-based access (individual, consultant, employer, admin) with least privilege.
- RLS. Row-Level Security in Postgres enforces tenant/user isolation in the database, not the app layer.
- Retrieval-time authZ. For RAG and any retrieval, authorization is applied BEFORE documents enter the context — never left to the model to withhold. Cross-tenant retrieval is a Critical defect.

| TOP RISK   Broken Access Control / IDOR is OWASP #1: every request re-checks that THIS user may touch THIS record. An authenticated user is not an authorized one. |
|---|


## 7. AI-Specific Security (OWASP LLM Top 10)

| Risk (OWASP LLM) | Control in MapleFile |
|---|---|
| LLM01 Prompt injection | Treat all retrieved/tool content as untrusted; instruction-data separation; injection regression tests (PyRIT/Garak/Promptfoo) |
| LLM02/07 Info disclosure | PII redaction; system-prompt hardening; no cross-tenant data; output scanning |
| LLM05 Output handling | Sanitize model output before it hits HTML/SQL/shell/code sinks |
| LLM06 Excessive agency | Tool least privilege; consequential actions need human approval |
| LLM08 (RAG) authZ | Pre-retrieval filtering by identity/tenant; vector-store isolation |
| LLM09 Misinformation | Ground answers in CRA/provincial sources; cite sections; deterministic calculators |
| LLM10 Unbounded consumption | Rate/cost limits; abuse detection |


## 8. AI Governance & Responsible AI
- Deterministic core. Deterministic tax computation; the LLM explains and guides but never computes a filing or states law from memory.
- Human oversight. Human-in-the-loop for filings and other consequential actions.
- Traceability. Every prompt, retrieval, tool call, and output is logged for audit and incident review.
- Honesty. Grounding, citations, and a clear 'research/verify with a professional' disclaimer on advice.
- Provenance. Model, prompt, and knowledge-base versions are tracked (supply-chain provenance, LLM03).

## 9. Privacy by Province

The province rules engine also selects the applicable privacy regime; MapleFile applies the strictest applicable to each user.

| Jurisdiction | Law | Key obligations honoured |
|---|---|---|
| Federal (default) | PIPEDA | Consent, purpose limitation, safeguards, breach reporting |
| Quebec | Law 25 | Enhanced consent, DPIA, data residency, right to portability |
| Alberta / BC | PIPA | Substantially similar; provincial breach rules |
| All | Data residency | Store & process in Canadian regions |


## 10. Control-to-Framework Mapping

| Control | NIST AI RMF | ISO 42001 | Privacy |
|---|---|---|---|
| PII/SIN redaction | Map / Manage | A.7 Data | PIPEDA safeguards; Law 25 |
| Encryption at rest/transit | Manage | A.7 / A.9 | PIPEDA safeguards |
| OIDC + MFA | Manage | Access control | Safeguards |
| RBAC + RLS (authZ) | Manage | Access control | Purpose limitation |
| Grounded, deterministic AI | Measure | A.6 Lifecycle | Accuracy |
| Audit logging | Govern | A.9 Ops | Accountability |
| Human oversight | Govern | A.5 Policies | Automated-decision rights |
| Data residency | Govern | A.7 Data | PIPEDA / Law 25 |


| CANADA AI LAW   Canada's Artificial Intelligence and Data Act (AIDA) and the federal Directive on Automated Decision-Making shape the governance posture: risk assessment, transparency, and human oversight for consequential automated decisions. |
|---|


## 11. Pre-Launch Security Checklist (summary)
- ☐ Threat model documented; trust boundaries marked (incl. AI boundary).
- ☐ SIN/PII redaction verified on every ingest + LLM path (test with synthetic SIN).
- ☐ Field-level encryption + KMS rotation confirmed; no keys in code/repo.
- ☐ OIDC/MFA + step-up on sensitive actions; session timeouts + revocation.
- ☐ RLS proven with cross-tenant tests; IDOR tests pass.
- ☐ Prompt-injection + RAG-isolation red-team (your Playbook 6 & 7) passed.
- ☐ Audit logging on all AI + filing actions; residency confirmed.
- ☐ Disclaimers, consent flows, and breach-response runbook in place.
