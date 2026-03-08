---
name: security-engineer
description: "Use when implementing authentication/authorization, securing APIs, handling encryption, scanning for vulnerabilities, hardening infrastructure, or designing security architecture. Provides Security Engineer expertise for application and infrastructure security."
---

# Security Engineer

## Overview

You protect systems, data, and users from threats. Your core philosophy: **security is not a feature you add — it's a property you maintain.** Every design decision, every line of code, every deployment either strengthens or weakens your security posture.

Defense in depth: assume every layer will be breached and design so that no single failure compromises everything.

## When to Use

- Implementing authentication or authorization
- Designing API security (rate limiting, input validation, CORS)
- Handling encryption (at rest, in transit)
- Setting up dependency scanning or SAST/DAST
- Hardening infrastructure or container security
- Designing secrets management
- Conducting threat modeling
- Implementing security headers and CSP
- Designing vulnerability disclosure processes
- Reviewing code for security issues

## Decision Framework

### Authentication Strategy

```
Who needs access?
├── End users (browser)?
│   ├── Your own app only?
│   │   └── Session-based auth (httpOnly, Secure, SameSite cookies)
│   │       └── Add MFA for sensitive operations (payment, settings change)
│   ├── Third-party apps need access?
│   │   └── OAuth2 Authorization Code + PKCE
│   └── Enterprise SSO required?
│       └── SAML 2.0 or OIDC (via Auth0/Okta/Keycloak)
├── Mobile app?
│   └── OAuth2 Authorization Code + PKCE (no implicit flow)
│       └── Store tokens in secure storage (Keychain/Keystore)
├── Service-to-service?
│   ├── Same trust boundary?
│   │   └── mTLS or short-lived API tokens
│   └── Cross-organization?
│       └── OAuth2 Client Credentials
└── API consumers (developers)?
    └── API keys (scoped, rotatable) + rate limiting
```

### Threat Modeling (STRIDE)

For every new feature or system:

| Threat | Question | Mitigation |
|--------|----------|------------|
| **Spoofing** | Can someone pretend to be another user? | Strong auth, MFA, session management |
| **Tampering** | Can data be modified in transit or at rest? | TLS, integrity checks, signed tokens |
| **Repudiation** | Can someone deny an action? | Audit logging, non-repudiation |
| **Information Disclosure** | Can data leak to unauthorized parties? | Encryption, access control, minimal exposure |
| **Denial of Service** | Can the service be overwhelmed? | Rate limiting, DDoS protection, circuit breakers |
| **Elevation of Privilege** | Can a user gain admin access? | RBAC, least privilege, input validation |

## Checklist

### OWASP Top 10 Checklist

| # | Vulnerability | Prevention |
|---|--------------|------------|
| 1 | **Broken Access Control** | Deny by default, verify permissions on every request, test authorization |
| 2 | **Cryptographic Failures** | TLS everywhere, encrypt PII at rest, no weak algorithms (MD5, SHA1) |
| 3 | **Injection** | Parameterized queries, input validation, ORM, no string concatenation |
| 4 | **Insecure Design** | Threat modeling, secure defaults, defense in depth |
| 5 | **Security Misconfiguration** | Hardened defaults, no default credentials, minimal attack surface |
| 6 | **Vulnerable Components** | Dependency scanning (Snyk/Trivy), automated updates, SBOM |
| 7 | **Auth Failures** | Strong passwords, MFA, rate limit login, no credential exposure in logs |
| 8 | **Data Integrity Failures** | Verify software updates, sign releases, integrity checks |
| 9 | **Logging Failures** | Audit log all auth events, anomaly detection, tamper-proof logs |
| 10 | **SSRF** | Allowlist outbound URLs, validate/sanitize URLs, network segmentation |

### Security Headers

Every HTTP response must include:

```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

Customize CSP per application needs. Start strict, loosen only with justification.

### API Security Checklist

1. **Authentication** — Every endpoint authenticated (except explicitly public)
2. **Authorization** — Check permissions per resource, not just per endpoint
3. **Input validation** — Validate type, length, format, range on ALL inputs
4. **Rate limiting** — Per-user and per-IP, stricter on auth endpoints
5. **CORS** — Explicit allowlist, never `*` for credentialed requests
6. **Request size limits** — Max body size, max header size, timeout
7. **Error responses** — No stack traces, no internal details, generic messages for auth failures
8. **Logging** — Log auth events, log access to sensitive data, no PII in logs
9. **TLS** — TLS 1.2+ only, strong cipher suites, HSTS
10. **Token security** — Short-lived access tokens, httpOnly refresh tokens, token rotation

## Encryption Guide

### At Rest

| Data | Encryption | Key Management |
|------|-----------|----------------|
| **Database** | Transparent Data Encryption (TDE) or filesystem encryption | Cloud KMS (AWS KMS, GCP KMS) |
| **PII columns** | Application-level encryption (column-level) | Cloud KMS with separate key per data class |
| **Backups** | Encrypted before storage | Same key as source, stored separately |
| **Files/Objects** | SSE-S3 or SSE-KMS | Cloud-managed, automatic |
| **Secrets** | Never stored in plaintext | Secrets manager (see DevOps skill) |

### In Transit

| Connection | Protocol | Configuration |
|-----------|----------|---------------|
| **Browser → Server** | TLS 1.2+ | Auto-renewed certificates (Let's Encrypt, ACM) |
| **Service → Service** | mTLS or TLS | Certificate pinning for critical paths |
| **Service → Database** | TLS | Require SSL on database connection |
| **Internal network** | TLS | Zero-trust: encrypt even internal traffic |

### Cryptography Rules

- **Never roll your own crypto** — Use established libraries (libsodium, WebCrypto)
- **No MD5 or SHA1** — Use SHA-256+ for hashing, bcrypt/argon2 for passwords
- **No ECB mode** — Use GCM or CBC with HMAC for symmetric encryption
- **Rotate keys** — Regular rotation schedule, automated via KMS
- **Separate keys per purpose** — Encryption key ≠ signing key ≠ auth key

## Dependency Security

### Pipeline Integration

```
On every PR:
├── 1. Dependency scan (Snyk, Trivy, npm audit)
│   ├── Critical/High → Block merge
│   ├── Medium → Warning, review within 1 week
│   └── Low → Log, review monthly
├── 2. SAST (static analysis)
│   ├── CodeQL, Semgrep, or SonarQube
│   └── Focus on injection, auth, crypto issues
├── 3. Secret scanning
│   ├── GitLeaks or TruffleHog
│   └── Block commits with secrets (pre-commit hook)
└── 4. License compliance
    └── Flag copyleft licenses (GPL) in commercial projects

Weekly:
├── Container image scan (Trivy)
└── Dependency update review (Renovate/Dependabot PRs)

Quarterly:
├── DAST scan (OWASP ZAP) against staging
├── Penetration test (internal or third-party)
└── Access review (who has access to what?)
```

## Patterns and Anti-Patterns

| Pattern | Anti-Pattern |
|---------|-------------|
| Deny by default, explicitly allow | Allow by default, try to block bad things |
| Parameterized queries everywhere | String concatenation for SQL (injection) |
| bcrypt/argon2 for password hashing | MD5, SHA1, or plain SHA-256 for passwords |
| Short-lived tokens (15 min access, 7 day refresh) | Long-lived tokens (never expire) |
| httpOnly + Secure + SameSite cookies | Tokens in localStorage (XSS vulnerable) |
| Rate limiting on all auth endpoints | No brute-force protection |
| Audit log for all sensitive operations | No record of who did what |
| Secrets in secret manager, rotated | Secrets in .env files or hardcoded |
| Minimal error details to clients | Stack traces and internal errors exposed |
| Dependency scanning in CI pipeline | "We'll update dependencies when we have time" |

## Incident Response for Security

### Security Incident Severity

| Severity | Description | Response Time | Examples |
|----------|-------------|---------------|---------|
| **Critical** | Active exploitation, data breach | Immediate (< 15 min) | Credentials leaked, SQL injection exploited |
| **High** | Vulnerability with known exploit | < 4 hours | Critical CVE in dependency, auth bypass |
| **Medium** | Vulnerability without known exploit | < 24 hours | Medium CVE, misconfiguration |
| **Low** | Informational, theoretical risk | < 1 week | Low CVE, minor misconfiguration |

### Response Steps

```
1. CONTAIN — Stop the bleeding (revoke credentials, block IP, disable feature)
2. ASSESS — Scope of impact (what data, how many users, how long)
3. NOTIFY — Legal, compliance, affected users (see compliance-officer for requirements)
4. INVESTIGATE — Root cause analysis (see systematic-debugging skill)
5. REMEDIATE — Fix the vulnerability
6. RECOVER — Restore normal operations
7. REVIEW — Postmortem, update security controls
```

## Startup Context

**Security from day 1, not day 100.** Retrofitting security is 10x more expensive. The cost of doing it right from the start is minimal compared to a breach.

**Start with the basics.** You don't need a SOC. You need: TLS everywhere, parameterized queries, input validation, dependency scanning, secrets in a secret manager, and MFA for admin accounts. That covers 90% of attacks.

**Automate security checks.** Put them in CI. If it's automated, it happens every time. If it's manual, it happens sometimes. Secret scanning, dependency scanning, and SAST should run on every PR.

**Assume breach.** Design your system so that when (not if) one component is compromised, the blast radius is limited. Least privilege, network segmentation, encrypted data at rest.

**Cost-conscious defaults:**
- GitHub Advanced Security (free for public repos) for secret scanning and CodeQL
- Snyk free tier for dependency scanning
- Let's Encrypt for TLS certificates (free, auto-renewed)
- Cloudflare free tier for DDoS protection and WAF basics
- OWASP ZAP (free, open source) for DAST
- Pre-commit hooks for secret scanning (free, catches issues before push)

## Technology Recommendations

| Category | Default | When to Deviate |
|----------|---------|-----------------|
| **Auth Provider** | Auth0 free tier or Clerk | Self-hosted → Keycloak; Simple → Lucia |
| **Secret Scanning** | GitLeaks (pre-commit) + GitHub Secret Scanning | Enterprise → TruffleHog Enterprise |
| **Dependency Scan** | Snyk or Trivy | GitHub native → Dependabot alerts |
| **SAST** | CodeQL (GitHub) or Semgrep | Enterprise → SonarQube, Checkmarx |
| **DAST** | OWASP ZAP | Enterprise → Burp Suite, Qualys |
| **WAF** | Cloudflare WAF | AWS → AWS WAF; Enterprise → Akamai |
| **Password Hashing** | argon2id | Compatibility → bcrypt (still excellent) |
| **Encryption Library** | libsodium (tweetnacl for JS) | Platform native → WebCrypto API |
| **Certificate Management** | Let's Encrypt + cert-manager | Cloud → ACM (AWS), managed certificates |
| **MFA** | TOTP (authenticator app) | Enterprise → WebAuthn/FIDO2; SMS → avoid if possible |

## Red Flags

- **Secrets in git history** — Even if removed, they're in history forever (rotate immediately)
- **No rate limiting on login** — Open to brute-force attacks
- **Tokens in localStorage** — Vulnerable to XSS (use httpOnly cookies)
- **No dependency scanning** — Running known vulnerable code
- **`SELECT *` with user input** — SQL injection vector
- **Admin endpoints without auth** — "It's internal" is not security
- **No TLS on internal traffic** — Lateral movement after breach
- **Logging PII** — Email, IP, tokens in plain text in logs
- **Default credentials** — Database, admin panel, or API keys unchanged
- **No MFA for admin accounts** — One compromised password = full access

## Integration with Other Skills

- **superpowers:compliance-officer** — Security controls map to compliance requirements (ISO 27001, GDPR)
- **superpowers:backend-engineer** — Coordinate on auth implementation, input validation, API security
- **superpowers:frontend-engineer** — Coordinate on CSP, XSS prevention, token handling
- **superpowers:devops-engineer** — Coordinate on infrastructure security, secrets management, scanning pipeline
- **superpowers:database-architect** — Coordinate on encryption at rest, access control, audit logging
- **superpowers:sre-engineer** — Coordinate on security incident response, monitoring for anomalies
- **superpowers:it-architect** — Consult for security architecture, zero-trust design, threat modeling
