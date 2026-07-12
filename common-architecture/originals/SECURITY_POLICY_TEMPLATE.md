# Security Policy

**Project:** [PROJECT_NAME]  
**Last Updated:** [DATE]  
**Next Review:** [DATE]  
**Owner:** [CISO/Security Team Lead]

---

## Executive Summary

This document defines security requirements and practices for [PROJECT_NAME]. Security is everyone's responsibility.

**Key commitments:**
- No production data in development
- All data encrypted in transit and at rest
- Regular security audits and penetration testing
- Incident response team on-call 24/7
- [Add your org's commitments]

---

## Security Governance

### Roles & Responsibilities

| Role | Responsibility |
|------|---|
| **CISO / Security Lead** | Owns security policy, approves exceptions |
| **Engineering Leads** | Implement security in code/infra |
| **QA Team** | Security testing, vulnerability scanning |
| **DevOps Team** | Infrastructure security, access controls |
| **All Engineers** | Report security issues immediately |

### Incident Response

**If you find a security issue:**

1. **Do NOT commit or push code**
2. **Report immediately to:** [security@example.com]
3. **Do NOT discuss in public channels**
4. **Expect response within 24 hours**

**Severity levels:**

| Level | Example | Response Time |
|-------|---------|---|
| **P1 (Critical)** | Data breach, exposed keys | 1 hour |
| **P2 (High)** | Auth bypass, SQL injection | 4 hours |
| **P3 (Medium)** | XSS vulnerability, weak crypto | 24 hours |
| **P4 (Low)** | Missing security header | 1 week |

---

## Authentication & Authorization

### User Authentication

**Required method:** [JWT / OAuth / SAML / OIDC]

**Implementation:**
- No plaintext passwords stored (use bcrypt/argon2)
- Passwords minimum [X] characters, must include [uppercase, numbers, symbols]
- Password reset requires email verification
- Login attempts limited to [X] per 15 minutes
- Failed logins require CAPTCHA after [X] attempts

**Token management:**
- Access tokens expire after [X] minutes
- Refresh tokens expire after [X] days
- Tokens never logged
- Tokens stored securely (httpOnly cookies preferred over localStorage)

### Multi-Factor Authentication (MFA)

**Required for:**
- Administrative accounts
- [Service accounts]
- [High-privilege operations]

**Supported methods:**
- TOTP (Time-based One-Time Password via Google Authenticator)
- [SMS (if applicable)]
- [Hardware keys (if applicable)]

### Authorization Model

**Approach:** [Role-Based Access Control / Attribute-Based Access Control]

**Permission hierarchy:**
```
Roles:
├── admin      (all permissions)
├── moderator  (content moderation)
├── user       (own data access)
└── guest      (read-only public)

Permissions:
├── users:read
├── users:write
├── users:delete
└── [Custom permissions]
```

**Resource-level controls:**
- Users can only access their own data
- Admins have full access
- Service-to-service auth via API keys
- [Custom rules for your domain]

---

## Multi-Tenant Architecture Enforcement (MANDATORY)

**[PROJECT_NAME] must enforce strict multi-tenant isolation.**

### Tenant Identifier Strategy

**Tenant scoping method:** [ACCOUNT_ID / COMPANY_ID / WORKSPACE_ID]

Every query and write operation MUST be scoped to tenant:

```sql
-- ✅ CORRECT: Always include tenant filter
SELECT * FROM orders WHERE company_id = ? AND order_id = ?

-- ❌ WRONG: Missing tenant filter
SELECT * FROM orders WHERE order_id = ?
```

### Cross-Tenant Denial (MANDATORY)

**System must actively REJECT cross-tenant access before database query:**

1. User from Tenant A requests data
2. System identifies tenant: Tenant A
3. User requests data with ID from Tenant B
4. System REJECTS request (403 Forbidden) BEFORE querying database
5. Never allow queries that could return another tenant's data

**Implementation requirement:**
```java
// In Repository layer
@Override
public Optional<Order> findById(String orderId, String tenantId) {
    // MANDATORY: Verify order belongs to tenant
    Order order = database.getOrder(orderId);
    if (order == null) {
        throw new AccessDeniedException();  // Not found = access denied
    }
    if (!order.getTenantId().equals(tenantId)) {
        throw new AccessDeniedException();  // Wrong tenant = access denied
    }
    return Optional.of(order);
}
```

### Testing Multi-Tenant Security (MANDATORY)

Every data-access test MUST include cross-tenant denial test:

```java
@Test
public void userFromTenantA_cannotAccess_TenantB_data() {
    User userA = createUser(TENANT_A);
    Order orderB = createOrder(TENANT_B);
    
    assertThatThrownBy(() -> service.getOrder(orderB.getId(), userA))
        .isInstanceOf(AccessDeniedException.class);
}
```

---

## Data Protection

### Encryption at Rest

**Database encryption:**
- Algorithm: AES-256
- Key management: [AWS KMS / Vault / HSM]
- Automatic key rotation: [Every X months]

**File storage encryption:**
- All uploaded files encrypted at rest
- Separate encryption keys per file
- Deletion removes encryption keys

**Backup encryption:**
- All backups encrypted with master key
- Keys stored in separate secure location
- Tested restore procedures

### Encryption in Transit

**HTTPS/TLS:**
- TLS 1.2 minimum (TLS 1.3 preferred)
- Strong ciphers only (no RC4, MD5, SHA1)
- Certificate pinning for mobile apps
- HSTS headers enabled

**API communication:**
- All APIs require HTTPS
- All webhooks use HTTPS with certificate validation
- Internal service-to-service: TLS or mTLS

### Data Classification

| Classification | Handling | Encryption | Access |
|---|---|---|---|
| **Public** | Public websites, blogs | Optional | Anyone |
| **Internal** | Non-sensitive business data | In transit | Employees only |
| **Confidential** | PII, financial data | At rest & transit | Specific roles |
| **Secret** | Keys, passwords, tokens | Encrypted + secrets vault | Very limited |

**Examples:**
- Public: Product documentation
- Internal: Meeting notes
- Confidential: User email addresses
- Secret: API keys, database passwords

### Personal Data (GDPR/CCPA)

**User rights:**
- Right to access: Can download all their data
- Right to be forgotten: Can request deletion
- Right to portability: Can export data in standard format
- Right to object: Can opt-out of processing

**Data retention:**
- Keep only as long as necessary
- Default delete: [X years] after last login
- Archive old data separately
- Permanent deletion: [X days] after deletion request

**Data breach notification:**
- Notify affected users within [X days]
- Notify regulators as required
- Provide credit monitoring if applicable

---

## Input Validation & Output Encoding

### Input Validation Rules

**ALL inputs must be validated:**

| Input Type | Validation |
|---|---|
| **Email** | RFC 5322 format, domain exists |
| **URL** | Valid scheme (http/https), no javascript: |
| **Number** | Type check, range check |
| **String** | Length check, character whitelist |
| **File upload** | Type check, size limit, virus scan |
| **JSON** | Schema validation |

**Example (JavaScript):**
```javascript
// ❌ WRONG
if (email) { ... }  // Just check if exists

// ✅ RIGHT
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
if (!emailRegex.test(email)) {
  throw new Error("Invalid email format");
}
```

### Output Encoding

**Context-specific encoding:**

| Context | Encoding |
|---|---|
| HTML | HTML entity encoding |
| JavaScript | JavaScript string encoding |
| URL | URL encoding |
| SQL | Prepared statements (never string concat) |
| CSV | Quote and escape fields |

**Example (HTML):**
```javascript
// ❌ WRONG
html = `<p>${userInput}</p>`  // XSS vulnerability!

// ✅ RIGHT
html = `<p>${escapeHtml(userInput)}</p>`
```

---

## Common Vulnerabilities Prevention

### SQL Injection

**Prevention:**
- Use parameterized queries/prepared statements
- Never concatenate SQL strings

```java
// ❌ WRONG
String query = "SELECT * FROM users WHERE id = " + userId;

// ✅ RIGHT
String query = "SELECT * FROM users WHERE id = ?";
stmt.setInt(1, userId);
```

### Cross-Site Scripting (XSS)

**Prevention:**
- Encode output for HTML context
- Use Content Security Policy headers
- Avoid `innerHTML`, use `textContent`

```javascript
// ❌ WRONG
element.innerHTML = userInput;

// ✅ RIGHT
element.textContent = userInput;
```

### Cross-Site Request Forgery (CSRF)

**Prevention:**
- CSRF tokens on all state-changing requests
- SameSite cookies
- Verify Origin/Referer headers

```html
<form method="POST">
  <input type="hidden" name="_csrf" value="token123">
  <button type="submit">Delete Account</button>
</form>
```

### Insecure Deserialization

**Prevention:**
- Never deserialize untrusted data
- Use JSON instead of pickle/Java serialization
- Validate type before deserialization

```python
# ❌ WRONG
obj = pickle.loads(user_data)

# ✅ RIGHT
data = json.loads(user_data)
assert isinstance(data, dict)
```

### Weak Cryptography

**Prevention:**
- Use approved algorithms (AES-256, SHA-256)
- Use random IVs per encryption
- Never roll your own crypto

**Approved:**
- ✅ AES-256-GCM (authenticated encryption)
- ✅ SHA-256, SHA-512 (hashing)
- ✅ PBKDF2 (password derivation)

**Not Approved:**
- ❌ DES, RC4 (weak)
- ❌ MD5, SHA-1 (broken)
- ❌ ECB mode (patterns visible)

---

## Secrets Management

### Where Secrets Go

**NEVER in:**
- ❌ Source code
- ❌ Comments
- ❌ Configuration files in repo
- ❌ Docker images
- ❌ Logs
- ❌ Error messages

**ALWAYS in:**
- ✅ Secrets vault (HashiCorp Vault, AWS Secrets Manager)
- ✅ Environment variables (in deployment only)
- ✅ Encrypted configuration service

### Secret Rotation

**Rotation schedule:**
- API keys: Every 90 days
- Passwords: Every 90 days
- Certificates: Before expiration (30 days)
- Database passwords: Every 6 months

**Process:**
1. Generate new secret
2. Update in vault
3. Services pick up new secret
4. Wait X days
5. Revoke old secret

### Secrets in Logs

**Sanitize sensitive data:**

```python
# ❌ WRONG - Token visible in logs
logger.info(f"Authorization header: {auth_header}")

# ✅ RIGHT - Token redacted
logger.info(f"Authorization header: Bearer [REDACTED]")
```

**Automated redaction:**
- Mask passwords, tokens, keys
- Mask PII (emails, SSNs, credit cards)
- Mask any field matching pattern `.*password.*`

---

## Security Testing

### Code Security Scanning

**Static Analysis (SAST):**
- Run on every commit: [SonarQube, Checkmarx, Fortify]
- Scan for: SQL injection, XSS, weak crypto, etc.
- Fail build if high-severity issues found

**Dependency Scanning:**
- Check dependencies for known vulnerabilities
- Tools: [npm audit, Snyk, WhiteSource, OWASP Dependency Check]
- Update or mitigate within [X] days of disclosure

### Dynamic Analysis (DAST)

**Penetration testing:**
- Annual external penetration test
- Quarterly internal security assessments
- Bug bounty program: [Yes/No]

**Vulnerability scanning:**
- Regular automated scanning of production
- Manual code review for critical paths
- Threat modeling for new features

### Test Coverage Requirements

**Security tests:**
- Authentication tests (valid/invalid tokens)
- Authorization tests (users can't access others' data)
- Input validation tests (malicious inputs rejected)
- Output encoding tests (XSS prevented)
- Encryption tests (data encrypted properly)

**Example:**
```javascript
describe('Authentication', () => {
  test('invalid token is rejected', () => {
    // Test that invalid JWT is rejected
  });

  test('expired token is rejected', () => {
    // Test that expired token is rejected
  });

  test('tampered token is rejected', () => {
    // Test that modified token is rejected
  });
});
```

---

## Access Control

### Development Access

**Repository:**
- All code changes require pull request
- Minimum 2 reviews before merge
- [X] automated checks must pass
- No direct pushes to main branch

**Secrets:**
- Limited to engineers with need-to-know
- All access logged
- Automatic revocation on role change

**Servers:**
- SSH keys for all access (no passwords)
- Key rotation every 90 days
- Separate keys per environment
- Bastion host for production access

### Production Access

**Principle of least privilege:**
- Minimal required permissions only
- Read-only access by default
- Elevated access approved and logged
- [X] person rule for sensitive operations

**Audit logging:**
- All production access logged
- Logs retained for [X] years
- Regular review of access logs
- Alerts on unusual access patterns

---

## Third-Party & Supply Chain Security

### Vendor Risk Management

**When using third-party services:**
- ✅ Data Protection Agreement (DPA) required
- ✅ SOC 2 certification reviewed
- ✅ Annual security assessment
- ✅ Incident notification requirements

**No cloud services allowed without:**
- Encryption in transit
- Encryption at rest
- Regular backups
- Vendor breach notification within [X] days

### Dependency Management

**Rules:**
- Use pinned versions (not `*` or `^`)
- Lock files committed to repo
- Regular dependency updates ([X] times/year)
- Security patches within [X] days of release
- Avoid beta/alpha versions in production

---

## Compliance

### Standards Compliance

**Your project must comply with:**
- [ ] OWASP Top 10
- [ ] [GDPR (if EU users)]
- [ ] [CCPA (if California users)]
- [ ] [HIPAA (if health data)]
- [ ] [PCI-DSS (if payment processing)]
- [ ] [SOC 2]
- [ ] [ISO 27001]

### Compliance Verification

- Annual security audit
- Penetration testing results reviewed
- Compliance checklist verified
- Gaps documented and remediated
- Report to board/executives

---

## Security Incident Response Plan

### Incident Classification

| Type | Example | Response |
|------|---------|----------|
| **Data Breach** | Unauthorized access to customer data | Notify users, regulators within [X] days |
| **System Outage** | Service unavailable | Status page update, user notification |
| **Malware** | Virus/ransomware detected | Isolate, scan, restore from backup |
| **DDoS Attack** | Volumetric attack | Activate DDoS mitigation |
| **Supply Chain** | Vendor breach | Assess impact, revoke credentials |

### Response Timeline

```
T+0:     Incident reported to security team
T+15min: Initial triage and severity assessment
T+30min: Executive notification if P1/P2
T+1hr:   Containment measures implemented
T+2hr:   Investigation begins
T+24hr:  Internal incident report
T+[X]days: External notification if required
T+7days:  Root cause analysis complete
T+30days: Remediation plan implemented
```

### Post-Incident Process

1. Investigate root cause thoroughly
2. Fix the underlying issue (not just symptom)
3. Implement detection for recurrence
4. Update security training if applicable
5. Document lessons learned
6. Share non-sensitive findings with team

---

## Security Training

**All engineers must:**
- ✅ Complete security training within [X] days of hire
- ✅ Annual security awareness training
- ✅ Secure coding best practices
- ✅ Incident response procedures
- ✅ Password/secret management

**Topic coverage:**
- OWASP Top 10 vulnerabilities
- Secure coding patterns
- [Your domain-specific threats]
- Incident response procedures
- Social engineering awareness

---

## Security Checklist

**Before deploying any code:**

- [ ] Input validation on all fields
- [ ] Output encoding in templates
- [ ] No secrets in code/logs
- [ ] Authentication required
- [ ] Authorization checked
- [ ] HTTPS enforced
- [ ] Security headers set
- [ ] Dependencies scanned for vulnerabilities
- [ ] No SQL injection possible (prepared statements)
- [ ] No XSS possible (encoded output)
- [ ] Rate limiting configured
- [ ] Security tests pass
- [ ] No hardcoded credentials
- [ ] Sensitive data not logged
- [ ] Code review completed

---

## Security Headers

**Required HTTP response headers:**

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=()
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'
```

---

## Monitoring & Alerts

**Security events to monitor:**
- Failed login attempts (threshold: [X] in 5 min)
- Multiple users from same IP (potential credential stuffing)
- Access from unusual location
- Large data exports
- Privilege escalation attempts
- Changes to security settings
- Certificate expiration (alert [X] days before)

**Alerting:**
- Critical: Page on-call within 5 minutes
- High: Team notification within 30 minutes
- Medium: Email notification within 24 hours

---

## Quick Reference

**Do's:**
- ✅ Report security issues immediately
- ✅ Validate all user input
- ✅ Encode all output
- ✅ Use HTTPS everywhere
- ✅ Keep dependencies updated
- ✅ Use secrets vault for credentials
- ✅ Log security events
- ✅ Test security regularly

**Don'ts:**
- ❌ Commit secrets to repo
- ❌ Trust client-side validation
- ❌ Use weak crypto algorithms
- ❌ Skip security reviews
- ❌ Log sensitive data
- ❌ Hardcode credentials
- ❌ Use HTTP in production
- ❌ Ignore security warnings

