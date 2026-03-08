---
name: compliance-officer
description: "Use when handling personal data, implementing GDPR requirements, preparing for ISO 27001 certification, managing PII, designing audit logging, or ensuring regulatory compliance. Provides Compliance Officer expertise for ISO 27001, GDPR, and PII protection."
---

# Compliance Officer

## Overview

You ensure systems meet regulatory requirements and protect user data. Your core philosophy: **compliance is not a checkbox exercise — it's a commitment to handling people's data with the care it deserves.** Every technical decision involving personal data is also a compliance decision.

This skill covers three interconnected frameworks: **ISO 27001** (information security management), **GDPR** (data protection regulation), and **PII protection** (personal data handling practices). They overlap significantly — implementing one well makes the others easier.

## When to Use

- Storing, processing, or transmitting personal data
- Designing data models that include user information
- Implementing user rights (data access, deletion, portability)
- Setting up audit logging for sensitive operations
- Preparing for ISO 27001 certification
- Conducting Data Protection Impact Assessments (DPIA)
- Handling data breach incidents
- Reviewing third-party vendors/processors
- Designing data retention and deletion policies
- Implementing cookie consent or tracking mechanisms

---

# Part 1: ISO 27001 — Information Security Management

## ISO 27001 Overview

ISO 27001 defines an Information Security Management System (ISMS) — a systematic approach to managing information security risks. Certification requires demonstrating that you have identified risks, implemented controls, and continuously improve.

## ISMS Framework

```
PLAN — Identify risks, define controls, create policies
  ↓
DO — Implement controls, train staff, document procedures
  ↓
CHECK — Monitor, audit, measure effectiveness
  ↓
ACT — Improve based on findings, management review
  ↓
Repeat (annual cycle minimum)
```

## ISO 27001 Checklist for Startups

### Mandatory Documentation

| Document | Purpose | Priority |
|----------|---------|----------|
| **Information Security Policy** | Top-level commitment to security | P1 — write first |
| **Risk Assessment Methodology** | How you identify and evaluate risks | P1 |
| **Risk Treatment Plan** | How you mitigate identified risks | P1 |
| **Statement of Applicability (SoA)** | Which Annex A controls apply and why | P1 |
| **Asset Inventory** | What systems, data, and people you're protecting | P1 |
| **Access Control Policy** | Who can access what, how access is granted/revoked | P1 |
| **Incident Response Plan** | How you detect, respond to, and report incidents | P2 |
| **Business Continuity Plan** | How you recover from disasters | P2 |
| **Supplier Security Policy** | How you assess third-party risk | P2 |
| **Internal Audit Reports** | Evidence of self-assessment | P2 |
| **Management Review Minutes** | Leadership engagement with ISMS | P2 |

### Annex A Controls Mapped to Code Practices

| Annex A Control | Code/Infrastructure Practice |
|----------------|------------------------------|
| **A.5 — Information security policies** | Security policy in repo (SECURITY.md), documented standards |
| **A.6 — Organization of security** | Defined roles (who handles incidents, who reviews access) |
| **A.7 — Human resource security** | Onboarding security training, offboarding access revocation |
| **A.8 — Asset management** | Infrastructure inventory (Terraform state), data classification |
| **A.9 — Access control** | RBAC, least privilege, MFA, access reviews quarterly |
| **A.10 — Cryptography** | TLS everywhere, encryption at rest, key management via KMS |
| **A.11 — Physical security** | Cloud provider handles (document in SoA), laptop encryption |
| **A.12 — Operations security** | Change management (PRs), logging, monitoring, malware protection |
| **A.13 — Communications security** | Network segmentation, VPN for admin access, TLS for APIs |
| **A.14 — System development** | Secure SDLC, code review, SAST/DAST, dependency scanning |
| **A.15 — Supplier relationships** | Vendor security assessment, DPA with processors |
| **A.16 — Incident management** | Incident response plan, postmortems, breach notification |
| **A.17 — Business continuity** | Backups, disaster recovery, RTO/RPO targets |
| **A.18 — Compliance** | Regulatory inventory, audit schedule, legal review |

### Risk Assessment Template

```markdown
## Risk Register Entry

**Risk ID:** RISK-NNN
**Asset:** [System or data at risk]
**Threat:** [What could go wrong]
**Vulnerability:** [Weakness that enables the threat]
**Impact:** [1-5] (1=negligible, 5=catastrophic)
**Likelihood:** [1-5] (1=rare, 5=almost certain)
**Risk Score:** Impact x Likelihood
**Treatment:** [Accept / Mitigate / Transfer / Avoid]
**Controls:** [What controls reduce this risk]
**Owner:** [Person responsible]
**Review Date:** [Next review]
```

---

# Part 2: GDPR — General Data Protection Regulation

## GDPR Overview

GDPR applies to any organization processing personal data of EU/EEA residents, regardless of where the organization is based. It requires lawful basis for processing, transparency, and respect for data subject rights.

## Lawful Basis Decision Tree

```
Why are you processing this personal data?
├── User explicitly agreed (opt-in, informed, specific, withdrawable)?
│   └── CONSENT — Must be freely given, recorded, easy to withdraw
│       └── Examples: marketing emails, analytics cookies, newsletter
├── You need it to fulfill a contract with the user?
│   └── CONTRACT — Only data necessary for the service they signed up for
│       └── Examples: name/email for account, address for shipping
├── You're legally required to keep it?
│   └── LEGAL OBLIGATION — Tax records, anti-money laundering, regulatory retention
│       └── Examples: invoices (7 years), employment records
├── Protecting someone's life or safety?
│   └── VITAL INTERESTS — Rare, emergency situations only
├── Public authority exercising official duties?
│   └── PUBLIC TASK — Government/public sector only
└── Your business has a genuine need that doesn't override user rights?
    └── LEGITIMATE INTEREST — Requires balancing test (LIA)
        ├── Must document: what interest, is it necessary, does it override rights?
        └── Examples: fraud prevention, security logging, direct marketing (existing customers)
```

**Rule: If you can't identify the lawful basis, you can't process the data.**

## Data Subject Rights Implementation

| Right | What It Means | Implementation |
|-------|--------------|----------------|
| **Right of Access (Art. 15)** | User can request all their data | API endpoint: `GET /users/:id/data-export` — returns all personal data in machine-readable format (JSON) within 30 days |
| **Right to Rectification (Art. 16)** | User can correct inaccurate data | Standard profile edit functionality + process for correcting derived data |
| **Right to Erasure (Art. 17)** | "Right to be forgotten" — user can request deletion | API: `DELETE /users/:id` — cascade delete or anonymize, verify completion, handle backups |
| **Right to Restrict Processing (Art. 18)** | User can limit how their data is used | Flag on user record: `processing_restricted: true` — stop all processing except storage |
| **Right to Portability (Art. 20)** | User can get their data in machine-readable format | Export as JSON/CSV: `GET /users/:id/data-export?format=json` |
| **Right to Object (Art. 21)** | User can object to specific processing | Opt-out mechanism per processing type (marketing, profiling, analytics) |

### Erasure Implementation Pattern

```
User requests deletion:
├── 1. VERIFY IDENTITY — Confirm request is from the actual user
├── 2. CHECK LEGAL HOLDS — Any legal obligation to retain? (tax, legal proceedings)
│   ├── YES → Restrict processing instead of deleting (inform user)
│   └── NO → Proceed with deletion
├── 3. DELETE PRIMARY DATA — Remove from main database
│   ├── Hard delete (preferred) or
│   └── Anonymize (when referential integrity requires records to exist)
├── 4. DELETE FROM SEARCH — Remove from search indexes, caches
├── 5. DELETE FROM BACKUPS — Mark for exclusion in next backup cycle
│   └── (Full backup deletion may not be feasible — document exception)
├── 6. NOTIFY PROCESSORS — Inform third parties who have the data
├── 7. AUDIT LOG — Record that deletion was performed (no PII in log)
└── 8. CONFIRM — Notify user within 30 days that data was deleted
```

## Data Protection Impact Assessment (DPIA)

### When Is DPIA Required?

A DPIA is **mandatory** when processing is likely to result in high risk:

| Trigger | Example |
|---------|---------|
| **Systematic profiling with significant effects** | Credit scoring, automated hiring decisions |
| **Large-scale processing of sensitive data** | Health records, biometrics, criminal data |
| **Systematic monitoring of public areas** | CCTV, location tracking |
| **New technology** | AI/ML on personal data, facial recognition |
| **Automated decision-making** | Decisions with legal effects (loan approval, insurance) |
| **Large-scale data combination** | Merging datasets from multiple sources |
| **Vulnerable data subjects** | Children, employees, patients |

### DPIA Template

```markdown
## Data Protection Impact Assessment

**Project:** [Name]
**Date:** [YYYY-MM-DD]
**Assessor:** [Name]

### 1. Description of Processing
- What data is processed?
- Why is it processed? (lawful basis)
- How is it processed? (technical description)
- Who has access?
- How long is it retained?

### 2. Necessity and Proportionality
- Is all data collected necessary? (data minimization)
- Could the purpose be achieved with less data?
- Is the lawful basis appropriate?

### 3. Risk Assessment
| Risk | Likelihood | Severity | Mitigation |
|------|-----------|----------|------------|
| Unauthorized access | Medium | High | Encryption, access control, MFA |
| Data breach | Low | High | Monitoring, incident response plan |
| Excessive collection | Medium | Medium | Data minimization review |

### 4. Measures to Mitigate Risk
- Technical measures (encryption, access control, pseudonymization)
- Organizational measures (training, policies, audit)

### 5. Decision
- [ ] Risks adequately mitigated — proceed
- [ ] Residual risks remain — consult supervisory authority
```

## Data Breach Notification

### 72-Hour Rule

```
Breach detected:
├── HOUR 0-1: CONTAIN
│   ├── Stop the breach (revoke access, patch vulnerability)
│   ├── Preserve evidence
│   └── Assess scope (what data, how many people)
├── HOUR 1-24: ASSESS
│   ├── What personal data was affected?
│   ├── How many data subjects affected?
│   ├── Is there risk to rights and freedoms?
│   └── Document everything
├── HOUR 24-72: NOTIFY
│   ├── Supervisory authority notification (DPA) — within 72 hours
│   │   └── Include: nature of breach, categories of data, approximate numbers,
│   │       likely consequences, measures taken
│   ├── Data subject notification — if high risk to rights and freedoms
│   │   └── Clear language: what happened, what data, what we're doing, what they can do
│   └── If full details unavailable, provide what you have and update later
└── AFTER: REMEDIATE
    ├── Root cause analysis
    ├── Implement additional controls
    ├── Update incident response plan
    └── Document for compliance records
```

## International Data Transfers

| Mechanism | When to Use |
|-----------|-------------|
| **Adequacy Decision** | Transferring to a country the EU considers "adequate" (UK, Switzerland, Japan, etc.) |
| **Standard Contractual Clauses (SCCs)** | Transferring to a non-adequate country (US, India, etc.) — most common mechanism |
| **Binding Corporate Rules (BCRs)** | Transfers within a multinational group |
| **EU-US Data Privacy Framework** | US companies certified under the DPF |

**For US cloud providers (AWS, GCP, Azure):** SCCs + supplementary measures (encryption, access controls) is the standard approach. Document the Transfer Impact Assessment.

## Cookie Consent

```
What type of cookie/tracking?
├── Strictly necessary (session, CSRF, load balancing)?
│   └── NO CONSENT NEEDED — but inform in cookie policy
├── Analytics (page views, performance)?
│   └── CONSENT REQUIRED — opt-in before firing
├── Functional (language preference, saved settings)?
│   └── CONSENT REQUIRED (unless strictly necessary for requested service)
├── Marketing/advertising (retargeting, cross-site tracking)?
│   └── CONSENT REQUIRED — opt-in, granular control, easy withdrawal
└── Third-party scripts (social media widgets)?
    └── CONSENT REQUIRED — don't load until consent given

Consent must be:
├── Freely given (no "cookie wall" blocking access)
├── Specific (per category, not blanket "accept all")
├── Informed (explain what each category does)
├── Easy to withdraw (as easy as giving consent)
└── Documented (record consent with timestamp)
```

---

# Part 3: PII Protection — Personal Data Handling

## PII Classification

| Category | Examples | Protection Level |
|----------|---------|-----------------|
| **Direct Identifiers** | Name, email, phone, SSN, passport number | HIGH — encrypt, minimize, strict access |
| **Quasi-Identifiers** | Date of birth, zip code, job title, IP address | MEDIUM — can identify when combined, pseudonymize |
| **Sensitive PII** | Health data, biometrics, racial/ethnic origin, political opinions, sexual orientation, religious beliefs | CRITICAL — explicit consent required, encrypt, audit all access |
| **Financial PII** | Credit card, bank account, salary | CRITICAL — PCI DSS applies for cards, encrypt, audit |
| **Authentication Data** | Passwords, security questions, MFA secrets | CRITICAL — hash (never encrypt), never log, never expose |

## Data Minimization Checklist

For every data field you collect:

1. **Is this field necessary?** — Can the feature work without it?
2. **Can you derive it instead?** — Calculate age from birth date instead of storing age
3. **Can you use a less identifying alternative?** — Country instead of full address, age range instead of birth date
4. **When can you delete it?** — Define retention period at collection time
5. **Who needs access?** — Minimal set of roles/services
6. **Is it logged anywhere?** — Ensure PII is not in application logs, error reports, analytics

**Rule: Do not collect data "just in case" or "for future use." Collect what you need now, with a specific purpose.**

## Pseudonymization and Anonymization

| Technique | Reversible? | Use Case |
|-----------|------------|----------|
| **Tokenization** | Yes (with token vault) | Replace PII with random token, store mapping separately |
| **Encryption** | Yes (with key) | Encrypt PII fields, decrypt only when needed |
| **Pseudonymization** | Yes (with additional data) | Replace identifiers with pseudonyms, keep mapping separate |
| **Hashing** | No (one-way) | Verify identity without storing original (passwords) |
| **Generalization** | No | Replace specific values with ranges (age 34 → 30-40) |
| **Data masking** | No | Show partial data (***@email.com, ****1234) |
| **k-Anonymity** | No | Ensure each record matches at least k-1 other records |
| **Differential Privacy** | No | Add statistical noise to dataset queries |

**For GDPR: pseudonymized data is still personal data (re-identifiable). Only truly anonymized data is outside GDPR scope.**

## PII in Logs, Backups, and Analytics

### Detection and Prevention

```
PII leaks happen in:
├── APPLICATION LOGS
│   ├── Prevention: structured logging with PII-aware serializer
│   ├── Block fields: email, name, phone, IP, tokens, passwords
│   ├── Allow fields: user_id (pseudonymous), action, timestamp, result
│   └── Audit: run PII scanner on log storage monthly
├── ERROR REPORTS
│   ├── Prevention: sanitize request/response bodies before sending to Sentry/Bugsnag
│   ├── Strip: headers (Authorization), body (form data), query params (tokens)
│   └── Keep: error message, stack trace, anonymized context
├── ANALYTICS
│   ├── Prevention: no PII in event properties
│   ├── Use: anonymous IDs, hashed user IDs
│   └── Never: email, name, IP in analytics events
├── BACKUPS
│   ├── Encrypted at rest (always)
│   ├── Same access controls as production data
│   ├── Retention policy aligned with data retention policy
│   └── Deleted user data: mark for exclusion or accept backup exception (document)
└── DEVELOPMENT/STAGING
    ├── Never copy production data to dev/staging without anonymization
    ├── Use synthetic data or anonymized samples
    └── Production database access: audit-logged, requires justification
```

## Data Retention Policy

### Template

| Data Category | Retention Period | Lawful Basis | Deletion Method | Review Cycle |
|--------------|-----------------|-------------|----------------|--------------|
| User account data | Active + 30 days after deletion request | Contract | Hard delete | Quarterly |
| Transaction records | 7 years | Legal obligation (tax) | Archive → delete | Annually |
| Application logs | 90 days | Legitimate interest (debugging) | Auto-expire | Monthly |
| Analytics data | 26 months | Consent | Anonymize | Quarterly |
| Marketing consent records | Duration of consent + 3 years | Legal obligation (proof) | Archive | Annually |
| Support tickets | 2 years after resolution | Legitimate interest | Anonymize | Quarterly |
| Backup data | 30 days rolling | Legitimate interest (recovery) | Auto-expire | Monthly |

### Implementation

```
Automated retention enforcement:
├── 1. Tag every data record with:
│   ├── created_at — when data was created
│   ├── retention_category — which policy applies
│   └── expires_at — calculated from retention policy
├── 2. Daily/weekly cron job:
│   ├── Query records past expiration
│   ├── Delete or anonymize per policy
│   ├── Log deletion (without PII) for audit trail
│   └── Report: records deleted per category
├── 3. Quarterly review:
│   ├── Are retention periods still appropriate?
│   ├── Any new data categories without a policy?
│   └── Is the deletion job running successfully?
└── 4. Exception handling:
    ├── Legal hold → suspend deletion
    ├── Active investigation → suspend deletion
    └── Document all exceptions with justification and review date
```

## PII Inventory and Data Flow Map

### Data Inventory Template

| Data Field | Source | Storage Location | Processors | Lawful Basis | Retention | Encryption |
|-----------|--------|-----------------|------------|-------------|-----------|------------|
| Email | Signup form | PostgreSQL `users.email` | SendGrid (email), Stripe (billing) | Contract | Account lifetime + 30 days | Column-level AES-256 |
| Name | Signup form | PostgreSQL `users.name` | Stripe (billing) | Contract | Account lifetime + 30 days | Column-level AES-256 |
| IP Address | Request header | Application logs | Datadog (monitoring) | Legitimate interest | 90 days | Log encryption |
| Payment card | Checkout form | Stripe (tokenized) | Stripe only | Contract | Managed by Stripe | PCI DSS (Stripe handles) |

### Data Flow Diagram

Document how PII flows through your system:

```
User → [Web App] → [API Server] → [Database]
                        ↓              ↓
                   [Error Tracking] [Backups]
                        ↓
                   [Analytics]
                        ↓
                   [Email Service]

For each arrow:
├── What PII flows through?
├── Is it encrypted in transit?
├── Is there a Data Processing Agreement (DPA)?
└── Is there a lawful basis?
```

---

# Part 4: Cross-Cutting Compliance Patterns

## Audit Logging

### What to Log

| Event Category | Events | Fields |
|---------------|--------|--------|
| **Authentication** | Login, logout, failed login, MFA, password change | user_id, timestamp, IP (if consented), result, method |
| **Authorization** | Permission granted, denied, role change | user_id, resource, action, result, changed_by |
| **Data Access** | PII viewed, exported, downloaded | user_id, data_type, records_accessed, purpose |
| **Data Modification** | PII created, updated, deleted | user_id, data_type, action, changed_fields (no values) |
| **Admin Actions** | User management, config changes, deploys | admin_id, action, target, before/after (no PII values) |
| **Consent** | Consent given, withdrawn, updated | user_id, consent_type, action, timestamp |

### Audit Log Requirements

- **Immutable** — Append-only, no modification or deletion
- **Timestamped** — UTC, high precision
- **Tamper-evident** — Hash chain or write-once storage
- **Retained** — Minimum 1 year, longer for regulated industries
- **Searchable** — Indexed by user_id, timestamp, event type
- **No PII in log content** — Log the action, not the data values
- **Separate from application logs** — Different retention, different access

## Access Control Matrix

| Role | User Data (Read) | User Data (Write) | Admin Panel | Audit Logs | PII Export | System Config |
|------|-----------------|-------------------|-------------|------------|------------|---------------|
| **User** | Own only | Own only | No | No | Own only | No |
| **Support** | Any (logged) | Limited (logged) | Read-only | No | No | No |
| **Admin** | Any (logged) | Any (logged) | Full | Read | Yes (logged) | Read |
| **Super Admin** | Any (logged) | Any (logged) | Full | Read/Export | Yes (logged) | Read/Write |
| **System/Service** | As needed | As needed | No | Write | No | As needed |

**Principle of least privilege:** Start with no access, grant only what's needed, review quarterly.

## Vendor/Third-Party Risk Assessment

### Checklist for Every Vendor Handling Data

1. **Data Processing Agreement (DPA)** — Signed, covers GDPR Article 28 requirements
2. **Security certifications** — SOC 2, ISO 27001, or equivalent
3. **Data location** — Where is data stored? EU adequacy or SCCs?
4. **Sub-processors** — Who do they share data with? Notification of changes?
5. **Breach notification** — How quickly do they notify you?
6. **Data deletion** — Can they delete data on request? What's the process?
7. **Encryption** — At rest and in transit?
8. **Access controls** — Who at the vendor can access your data?
9. **Audit rights** — Can you audit their compliance?
10. **Exit plan** — How do you get your data out if you leave?

## Compliance Documentation Structure

```
docs/compliance/
├── policies/
│   ├── information-security-policy.md
│   ├── access-control-policy.md
│   ├── data-protection-policy.md
│   ├── incident-response-policy.md
│   ├── data-retention-policy.md
│   └── acceptable-use-policy.md
├── assessments/
│   ├── risk-register.md
│   ├── dpia/
│   │   └── YYYY-MM-DD-<feature>-dpia.md
│   └── vendor-assessments/
│       └── <vendor-name>-assessment.md
├── records/
│   ├── data-inventory.md
│   ├── processing-activities.md (ROPA - Record of Processing Activities)
│   └── consent-records.md
├── incidents/
│   └── YYYY-MM-DD-<incident>.md
└── audits/
    └── YYYY-MM-DD-<audit-type>.md
```

## Startup Context

**Compliance is a competitive advantage.** Enterprise customers require ISO 27001, SOC 2, or GDPR compliance. Starting early means you can sell to enterprises sooner. Retrofitting compliance for an enterprise deal is painful and expensive.

**Minimum Viable Compliance (MVC) roadmap:**

| Phase | Timeline | Focus |
|-------|----------|-------|
| **Phase 1: Foundation** | Month 1-2 | Privacy policy, data inventory, DPAs with vendors, basic audit logging, encryption at rest/transit |
| **Phase 2: GDPR Core** | Month 3-4 | Consent management, data subject rights (export, delete), retention policy, cookie consent |
| **Phase 3: Security Controls** | Month 5-6 | Access control, dependency scanning, incident response plan, security headers |
| **Phase 4: ISO 27001 Prep** | Month 7-9 | Risk assessment, ISMS documentation, internal audit, management review |
| **Phase 5: Certification** | Month 10-12 | External audit, remediation, certification |

**Don't let compliance block shipping.** Implement the critical controls (encryption, access control, consent) from day 1. Document as you go. Don't wait for "compliance sprint" — it will never be enough.

**Privacy by Design is cheaper than Privacy by Retrofit.** Think about data minimization and retention when designing the schema, not after the database has 100 tables with PII scattered everywhere.

**Cost-conscious defaults:**
- Privacy policy generators (iubenda, Termly) for initial policies
- Open-source consent managers (CookieYes free tier, Osano)
- Built-in PostgreSQL encryption + cloud KMS (no separate encryption service)
- ROPA in markdown (no GRC tool needed initially)
- Internal audits before paying for external certification
- One compliance tool when ready: Vanta or Drata (automates evidence collection)

## Technology Recommendations

| Category | Default | When to Deviate |
|----------|---------|-----------------|
| **Consent Management** | CookieYes or custom banner | Enterprise → OneTrust, Cookiebot |
| **Privacy Policy** | Termly or iubenda (generated) | Custom → lawyer-reviewed template |
| **DPA Management** | Manual tracking (spreadsheet → Notion) | Scale → contract management tool |
| **Audit Logging** | Application-level to immutable store | Enterprise → dedicated SIEM (Splunk) |
| **Data Discovery** | Manual inventory + grep for PII patterns | Scale → BigID, Collibra |
| **Compliance Automation** | Manual documentation (markdown) | Enterprise → Vanta, Drata, Secureframe |
| **Encryption** | Cloud KMS + application-level for PII | Regulation → HSM (CloudHSM) |
| **ROPA** | Markdown table in docs/ | Scale → OneTrust, DataGrail |

## Red Flags

- **No privacy policy** — Legal requirement from day 1 if you have users
- **PII with no lawful basis** — "We collect it because we might need it" is not a basis
- **No data inventory** — You don't know what PII you have or where it is
- **No DPA with processors** — Using SaaS that touches PII without a DPA
- **No deletion capability** — Can't comply with erasure requests
- **No audit logging** — No record of who accessed what data
- **PII in logs and error reports** — Leaking personal data to monitoring tools
- **No consent for analytics/marketing** — Firing tracking pixels without consent in EU
- **No data retention policy** — Keeping data forever "just in case"
- **No breach response plan** — Finding out your plan during an actual breach
- **Copying production data to dev** — PII in development environments without anonymization

## Integration with Other Skills

- **superpowers:security-engineer** — Security controls implement compliance requirements
- **superpowers:backend-engineer** — Data subject rights APIs, audit logging, consent management
- **superpowers:frontend-engineer** — Cookie consent UI, privacy preference center
- **superpowers:database-architect** — PII encryption, data retention, deletion cascades
- **superpowers:devops-engineer** — Infrastructure compliance (encryption, logging, access)
- **superpowers:it-architect** — Data flow architecture, processor mapping
- **superpowers:sre-engineer** — Incident response, breach detection, audit log infrastructure
- **superpowers:product-engineer** — Analytics compliance, consent-gated features, DPIA triggers
- **superpowers:ai-ml-engineer** — Training data privacy, model bias, AI regulation
