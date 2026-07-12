# Common Architecture Templates

**Purpose:** Reusable architecture and operations documentation templates for any software project.

**Status:** Production-ready templates (customize for your project)

---

## Integration with Development Standards

**Important:** These templates work in conjunction with [Common Development Rules](../common-AI-rules/development.rules.md). Review both documents:

- **Development Rules** → Language-specific standards, naming conventions, layering principles
- **Architecture Templates** → System-wide patterns, technology decisions, organizational structure

Key standards referenced:
- Three-tier architecture pattern (Swing/UI → UIControllers → REST API → Services → Repositories → Database)
- Framework-agnostic UIControllers
- Dependency injection over direct instantiation
- Database logic isolation to Repository layer only

---

## What's Included

This collection provides generic, project-agnostic templates for essential architecture and operational documentation:

### Core Architecture Documents

| Document | Purpose | Audience | Size |
|----------|---------|----------|------|
| **[ARCHITECTURE_PRINCIPLES_TEMPLATE.md](./ARCHITECTURE_PRINCIPLES_TEMPLATE.md)** | Design philosophy, core principles, technology decisions, evolution roadmap | Everyone | ~23KB |
| **[API_STANDARDS_TEMPLATE.md](./API_STANDARDS_TEMPLATE.md)** | REST API design, versioning, response formats, error handling | Backend & Frontend engineers | ~13KB |
| **[SECURITY_POLICY_TEMPLATE.md](./SECURITY_POLICY_TEMPLATE.md)** | Security requirements, incident response, compliance, data protection | Security, DevOps, All engineers | ~16KB |

### Platform-Specific Architecture

| Document | Purpose | Audience | Size |
|----------|---------|----------|------|
| **[FRONTEND_ARCHITECTURE_TEMPLATE.md](./FRONTEND_ARCHITECTURE_TEMPLATE.md)** | Frontend patterns, component architecture, state management | Frontend engineers | ~31KB |
| **[MOBILE_ARCHITECTURE_TEMPLATE.md](./MOBILE_ARCHITECTURE_TEMPLATE.md)** | Mobile-first design, offline-first patterns, device constraints | Mobile engineers | ~24KB |
| **[STATE_MANAGEMENT_TEMPLATE.md](./STATE_MANAGEMENT_TEMPLATE.md)** | State patterns (Redux, MobX, service-based, Context API) | Frontend engineers | ~24KB |

### Execution & Operations

| Document | Purpose | Audience | Size |
|----------|---------|----------|------|
| **[SPRINT_STRATEGY_TEMPLATE.md](./SPRINT_STRATEGY_TEMPLATE.md)** | Sprint planning, task workflow, success metrics, risk management | Scrum masters, team leads | ~6KB |

---

## How to Use These Templates

### Quick Start (5 minutes)

1. **Review [Common Development Rules](../common-AI-rules/development.rules.md)** to understand mandatory code patterns
2. Copy the templates to your project's `/architecture` or `/docs` folder
3. Rename by removing `_TEMPLATE` suffix
4. Replace all `[PLACEHOLDER]` values with your project details
5. Remove sections that don't apply
6. Have team review and approve

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

Strategic architecture decisions and design philosophy.

**Sections:**
```
1. How to Use This Template      (Instructions)
2. Core Principles               (8+ architectural principles)
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

**Key Customizations:**
- Core principles for your domain
- Performance targets (latency, throughput)
- Technology stack choices and justification
- Phased evolution roadmap
- Team roles and accountability
- **Connect to:** Development rules for code-level enforcement of principles

---

### API_STANDARDS_TEMPLATE.md

REST API contract between backend and frontend/mobile clients.

**Sections:**
```
1. URL Structure              (Versioning, naming, conventions)
2. HTTP Methods              (GET, POST, PUT, PATCH, DELETE semantics)
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

**Key Customizations:**
- Base URLs (development, staging, production)
- API versioning strategy (URL path, header, or query param)
- Naming conventions for your domain
- Required headers and authentication method
- Error codes specific to your domain
- Rate limiting and quota policies
- **Connect to:** Development rules for REST @RestController layer

---

### SECURITY_POLICY_TEMPLATE.md

Security requirements, threat model, and incident response procedures.

**Sections:**
```
1. Executive Summary           (Security commitments)
2. Security Governance         (Roles, incident response)
3. Authentication & Authorization (Methods, tokens, permissions)
4. Data Protection            (Encryption, classification, retention)
5. Input Validation & Output Encoding (Prevent injection attacks)
6. Common Vulnerabilities     (SQL injection, XSS, CSRF, OWASP Top 10)
7. Secrets Management         (Where, rotation, redaction)
8. Security Testing          (SAST, DAST, penetration testing)
9. Access Control            (Development, production)
10. Third-Party Security     (Vendors, dependencies)
11. Compliance               (Standards, verification)
12. Incident Response Plan   (P1-P4 classification, timeline, post-incident)
13. Security Training        (Requirements, topics)
14. Security Checklist       (15-point pre-deploy checklist)
15. Security Headers         (Required HTTP headers)
16. Monitoring & Alerts      (What to watch, thresholds)
17. Quick Reference          (Do's and don'ts)
```

**Key Customizations:**
- Incident response contact info and escalation
- Encryption standards (algorithms, key rotation)
- Compliance requirements (GDPR, HIPAA, SOC2, etc.)
- Authentication method (JWT, OAuth, SAML, OIDC)
- Access control model (RBAC, ABAC)
- Alert thresholds and escalation procedures
- **Connect to:** Development rules for Repository layer isolation and input validation

---

## Integration with Development Rules

The [Common Development Rules](../common-AI-rules/development.rules.md) define **mandatory code-level patterns** that enforce the architecture decisions documented in these templates:

### How They Work Together

**Architecture Templates** (Strategy)
↓
**Development Rules** (Enforcement)
↓
**Code Implementation**

### Key Alignment Points

| Architecture Decision | Development Rule | Purpose |
|---|---|---|
| Three-tier separation | Tier definitions & isolation rules | Enforces layer separation in code |
| Framework-agnostic UI | No AppView in Swing classes | Prevents UI coupling to infrastructure |
| Business logic in backend | Database logic isolation to Repository | Prevents mixing concerns across layers |
| Dependency injection | DI over direct instantiation | Ensures testability and modularity |
| REST API layer | @RestController, HTTP semantics | Contracts API layer behavior |
| No direct database access | Repositories only in data layer | Enforces data access isolation |
| Configuration classes | Infrastructure bootstrap only | Prevents framework concerns in UI |

### Review Both Documents

1. **Start with Architecture Templates** to understand *why* a decision was made
2. **Review Development Rules** to see *how* to enforce it in code
3. **Implement using Code patterns** from the rules

Example: Three-Tier Architecture
- **Template says:** Separate UI, UIControllers, REST API, Services, Repositories
- **Rule says:** UIControllers must not import Swing, Services must use repositories, no database in UI layer
- **Code implements:** Specific class structure, imports, and method signatures

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
| 1.1 | 2026-07-12 | Added integration with Development Rules; clarified platform-specific templates; enhanced section descriptions |
| 1.0 | 2026-07-12 | Initial templates created (ARCHITECTURE_PRINCIPLES, API_STANDARDS, SECURITY_POLICY, FRONTEND_ARCHITECTURE, MOBILE_ARCHITECTURE, STATE_MANAGEMENT, SPRINT_STRATEGY) |

---

## Contributing

These templates are living documents. If you improve them:

1. Update your copy
2. Share improvements with other teams
3. Update this README with lessons learned
4. Create issues for gaps or confusing sections

---

## Quick Links

**Companion Documents:**
- [**Common Development Rules**](../common-AI-rules/development.rules.md) — Code-level standards and patterns (Java, architecture, naming, security)

**Inside this directory:**
- [`ARCHITECTURE_PRINCIPLES_TEMPLATE.md`](./ARCHITECTURE_PRINCIPLES_TEMPLATE.md) — System design & philosophy (~23KB)
- [`API_STANDARDS_TEMPLATE.md`](./API_STANDARDS_TEMPLATE.md) — REST API guidelines (~13KB)
- [`SECURITY_POLICY_TEMPLATE.md`](./SECURITY_POLICY_TEMPLATE.md) — Security & compliance (~16KB)
- [`FRONTEND_ARCHITECTURE_TEMPLATE.md`](./FRONTEND_ARCHITECTURE_TEMPLATE.md) — Frontend patterns (~31KB)
- [`MOBILE_ARCHITECTURE_TEMPLATE.md`](./MOBILE_ARCHITECTURE_TEMPLATE.md) — Mobile-first design (~24KB)
- [`STATE_MANAGEMENT_TEMPLATE.md`](./STATE_MANAGEMENT_TEMPLATE.md) — State patterns (~24KB)
- [`SPRINT_STRATEGY_TEMPLATE.md`](./SPRINT_STRATEGY_TEMPLATE.md) — Sprint execution (~6KB)

**External Resources:**
- [OWASP Top 10](https://owasp.org/Top10/) — Common security vulnerabilities
- [REST API Best Practices](https://restfulapi.net/) — RESTful design principles
- [The Twelve Factor App](https://12factor.net/) — Cloud-native architecture
- [Google Cloud Best Practices](https://cloud.google.com/docs) — Infrastructure patterns
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/) — System design principles

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
