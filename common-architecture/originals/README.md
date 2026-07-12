# Common Architecture Templates

**Purpose:** Reusable architecture and operations documentation templates for any software project.

**Status:** Production-ready templates (customize for your project)

---

## What's Included

This collection provides generic, project-agnostic templates for essential architecture and operational documentation:

### Core Architecture Documents

| Document | Purpose | Audience |
|----------|---------|----------|
| **[ARCHITECTURE_PRINCIPLES_TEMPLATE.md](./ARCHITECTURE_PRINCIPLES_TEMPLATE.md)** | Design philosophy, core principles, technology decisions | Everyone |
| **[API_STANDARDS_TEMPLATE.md](./API_STANDARDS_TEMPLATE.md)** | REST API design guidelines, response formats, standards | Backend & Frontend engineers |
| **[SECURITY_POLICY_TEMPLATE.md](./SECURITY_POLICY_TEMPLATE.md)** | Security requirements, incident response, compliance | Security, DevOps, All engineers |

### Original Reference

| Document | Notes |
|----------|-------|
| **[originals/ARCHITECTURE_PRINCIPLES.md](./originals/)** | Original ChatGPT Client project version (for reference) |

---

## How to Use These Templates

### Quick Start (5 minutes)

1. Copy the templates to your project's `/architecture` or `/docs` folder
2. Rename by removing `_TEMPLATE` suffix
3. Replace all `[PLACEHOLDER]` values with your project details
4. Remove sections that don't apply
5. Have team review and approve

### Detailed Customization (30 minutes)

**For each document:**

1. **Read the template** to understand structure
2. **Identify placeholders** (marked with `[BRACKETS]`)
3. **Gather project info:**
   - Technology stack
   - Team structure
   - Performance targets
   - Security requirements
4. **Customize content:**
   - Add domain-specific constraints
   - Adjust tech choices to match your stack
   - Set realistic targets for your scale
5. **Create missing sections:**
   - Add your specific architectural patterns
   - Document your deployment process
   - Define your incident response procedures
6. **Review and approve** with team leads
7. **Share with entire team**

### Maintaining Your Documents

**Review schedule:**
- Quarterly architecture review
- Annual comprehensive review
- Immediate update for major changes

**Keep current:**
- Update when technology decisions change
- Reflect actual implementation (not ideals)
- Archive old versions for reference

---

## Template Structure Overview

### ARCHITECTURE_PRINCIPLES_TEMPLATE.md

**Sections:**
```
1. How to Use This Template      (Instructions)
2. Core Principles               (8 core architectural principles)
3. Architectural Constraints     (Domain-specific constraints)
4. Non-Functional Requirements   (Performance, reliability, security, scalability)
5. Technology Decisions          (Backend, frontend, mobile, DevOps)
6. Architectural Patterns        (Common patterns to use)
7. Code Quality Standards        (Testing, review, documentation)
8. Decision Framework            (When to add complexity, refactor, optimize)
9. Deployment Philosophy         (Zero-downtime, migrations, monitoring)
10. Anti-Patterns               (Common mistakes to avoid)
11. Guiding Questions           (Decision-making framework)
12. Evolution Path              (Phase 1, 2, 3 roadmap)
13. Accountability              (Owners, review schedule)
14. Related Documents           (Links to other docs)
15. Customization Checklist     (19-point checklist)
```

**Customize:**
- All `[PRINCIPLE_NAME]` placeholders
- Performance targets (Section 4)
- Technology stack (Section 5)
- Phases and timelines (Section 12)
- Team roles (Section 13)

---

### API_STANDARDS_TEMPLATE.md

**Sections:**
```
1. URL Structure              (Versioning, naming, conventions)
2. HTTP Methods              (GET, POST, PUT, PATCH, DELETE)
3. Request Format            (Headers, body, query params)
4. Response Format           (Success & error structures)
5. HTTP Status Codes         (When to use each)
6. Authentication            (Bearer tokens, rate limiting)
7. Data Types & Formats      (Timestamps, booleans, enums)
8. Pagination Standards      (Offset, cursor, limits)
9. Versioning Strategy       (Deprecation policy, migration)
10. Error Codes             (Standard error types)
11. Documentation           (What to document for each endpoint)
12. Testing Checklist       (15-point pre-ship checklist)
13. Quick Reference         (Do's and don'ts)
```

**Customize:**
- Base URLs (Development, staging, production)
- Naming conventions for your domain
- Required headers and authentication
- Error codes specific to your APIs
- Rate limiting policies

---

### SECURITY_POLICY_TEMPLATE.md

**Sections:**
```
1. Executive Summary           (Security commitments)
2. Security Governance         (Roles, incident response)
3. Authentication & Authorization (Methods, tokens, permissions)
4. Data Protection            (Encryption, classification, retention)
5. Input Validation & Output Encoding (Prevent injection attacks)
6. Common Vulnerabilities     (SQL injection, XSS, CSRF, etc.)
7. Secrets Management         (Where, rotation, redaction)
8. Security Testing          (SAST, DAST, penetration testing)
9. Access Control            (Development, production)
10. Third-Party Security     (Vendors, dependencies)
11. Compliance               (Standards, verification)
12. Incident Response Plan   (Classification, timeline, post-incident)
13. Security Training        (Requirements, topics)
14. Security Checklist       (15-point pre-deploy checklist)
15. Security Headers         (Required HTTP headers)
16. Monitoring & Alerts      (What to watch, thresholds)
17. Quick Reference          (Do's and don'ts)
```

**Customize:**
- Incident response contact info
- Encryption standards for your data
- Compliance requirements (GDPR, HIPAA, etc.)
- Authentication method (JWT, OAuth, SAML)
- Access control model (RBAC, ABAC)
- Alert thresholds and escalation

---

## Common Customization Patterns

### For a Startup

**ARCHITECTURE_PRINCIPLES:**
- Simplify technology decisions to 1-2 choices
- Focus on Phase 1 only (defer Phase 2/3)
- Relax performance targets (bootstrap mode)
- Fewer team members = broader roles

**API_STANDARDS:**
- Single API version (v1)
- Simpler response format
- No pagination initially

**SECURITY_POLICY:**
- Basic authentication (maybe just API keys)
- Focus on preventing common issues
- Simpler compliance (no HIPAA/GDPR initially)

### For an Enterprise System

**ARCHITECTURE_PRINCIPLES:**
- Detailed technology justification
- Multiple deployment environments
- Strict performance targets
- Formal governance structure

**API_STANDARDS:**
- Multiple API versions
- Detailed error handling
- Advanced filtering and pagination
- GraphQL alternative

**SECURITY_POLICY:**
- Strict compliance requirements
- Formal incident response process
- Regular penetration testing
- Audit logging and compliance verification

### For a Microservices Architecture

**ARCHITECTURE_PRINCIPLES:**
- Service communication patterns
- Service discovery strategy
- API gateway approach
- Distributed tracing

**API_STANDARDS:**
- Separate standards per service
- Inter-service communication protocols
- Schema versioning strategy

**SECURITY_POLICY:**
- Service-to-service authentication (mTLS)
- Service mesh security
- API gateway authentication

---

## Real-World Examples

### Customize for Backend API

**Starting with:** ARCHITECTURE_PRINCIPLES_TEMPLATE.md

```markdown
Project: User Management API
Language: Java 11 (replace [Backend Language])
Framework: Spring Boot (replace [Backend Framework])
Database: PostgreSQL (replace [Database])
Deployment: Kubernetes (replace [Deployment Type])
Performance Target: 100ms p99 (replace [Performance])
```

### Customize for Mobile App

**Starting with:** ARCHITECTURE_PRINCIPLES_TEMPLATE.md

```markdown
Project: iOS Weather App
Framework: SwiftUI (replace [Frontend Framework])
Platform: iOS 14+ (replace [Platforms])
Offline: Yes (replace [Offline Support])
State Management: Redux-like (replace [State])
```

### Customize for Data Pipeline

**Starting with:** ARCHITECTURE_PRINCIPLES_TEMPLATE.md

```markdown
Project: Analytics Pipeline
Language: Python 3.9 (replace backend language)
Framework: Apache Airflow (replace framework)
Data Source: Kafka + S3 (replace [Constraints])
Performance: 1M events/hour (replace [Scalability])
```

---

## Creating Your Project-Specific Versions

### Step 1: Copy Templates

```bash
mkdir -p project-docs/architecture
cp ARCHITECTURE_PRINCIPLES_TEMPLATE.md project-docs/architecture/ARCHITECTURE_PRINCIPLES.md
cp API_STANDARDS_TEMPLATE.md project-docs/architecture/API_STANDARDS.md
cp SECURITY_POLICY_TEMPLATE.md project-docs/architecture/SECURITY_POLICY.md
```

### Step 2: Create Placeholder List

```bash
# Find all placeholders
grep -n "\[" ARCHITECTURE_PRINCIPLES.md | grep -E "\[[A-Z_]+\]"
```

### Step 3: Fill in Placeholders

**Use find & replace:**
```bash
sed -i 's/\[PROJECT_NAME\]/My Awesome App/g' *.md
sed -i 's/\[DATE\]/2026-07-12/g' *.md
sed -i 's/\[TECH_STACK\]/Java, Spring Boot, PostgreSQL/g' *.md
```

### Step 4: Review & Customize

- Read each section
- Remove inapplicable sections
- Expand domain-specific sections
- Add team-specific details

### Step 5: Team Review

```markdown
## Review Checklist

- [ ] Principles match our actual values
- [ ] Technology choices are current and approved
- [ ] Performance targets are realistic
- [ ] Security requirements are complete
- [ ] Team agrees with approach
- [ ] Documentation is accurate
```

### Step 6: Share & Maintain

- Commit to repository
- Link from README.md
- Set calendar reminder for next review
- Update when decisions change

---

## Adapting Templates for Your Context

### Questions to Answer While Customizing

**For Architecture Principles:**
- What are our top 3 architectural values?
- What technologies are we using and why?
- What scale do we need to support?
- What's our deployment strategy?
- Who makes architecture decisions?

**For API Standards:**
- What's our API versioning strategy?
- What authentication method are we using?
- How do we handle errors?
- What's our rate limiting approach?
- Do we support pagination?

**For Security Policy:**
- What's our encryption strategy?
- What compliance requirements apply?
- Who's responsible for security?
- What's our incident response process?
- How do we manage secrets?

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-07-12 | Initial templates created |
| [Future] | [Date] | [Changes] |

---

## Contributing

These templates are living documents. If you improve them:

1. Update your copy
2. Share improvements with other teams
3. Update this README with lessons learned
4. Create issues for gaps or confusing sections

---

## Quick Links

**Inside this directory:**
- [`ARCHITECTURE_PRINCIPLES_TEMPLATE.md`](./ARCHITECTURE_PRINCIPLES_TEMPLATE.md) — Main template (550+ lines)
- [`API_STANDARDS_TEMPLATE.md`](./API_STANDARDS_TEMPLATE.md) — API guidelines (400+ lines)
- [`SECURITY_POLICY_TEMPLATE.md`](./SECURITY_POLICY_TEMPLATE.md) — Security requirements (500+ lines)
- [`originals/`](./originals/) — Original project version (for reference)

**External resources:**
- [OWASP Top 10](https://owasp.org/Top10/)
- [REST API Best Practices](https://restfulapi.net/)
- [The Twelve Factor App](https://12factor.net/)
- [Google Cloud Best Practices](https://cloud.google.com/docs)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)

---

## FAQ

**Q: Where do I start?**  
A: Read this README, then pick the template most relevant to your project, and work through the customization checklist at the end.

**Q: Can I skip sections?**  
A: Yes! Remove sections that don't apply to your project. It's better to have a focused document than one you don't use.

**Q: How often should we update?**  
A: Review quarterly, update when decisions change. Major reviews annually.

**Q: How detailed should we get?**  
A: Cover decisions that matter for your team's success. Skip trivial details.

**Q: Can multiple teams use these?**  
A: Yes! Each team creates their own copy with their own customizations.

**Q: Should we put these in source control?**  
A: Yes! Keep them in `/docs` or `/architecture` folder in your repo with everything else.

---

## Support

**Questions about using these templates?**
- Check the detailed instructions in each template
- Look at the originals folder for a real example
- Ask your team lead or architect

**Found an issue or improvement?**
- Document it
- Propose the change
- Update this README with lessons learned

---

**Last Updated:** 2026-07-12  
**Maintained by:** Architecture Team  
**License:** [Internal Use / Open Source if applicable]
