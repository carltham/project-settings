# Architecture Principles

**Project:** [PROJECT_NAME]  
**Version:** 1.0  
**Last Updated:** [DATE]  
**Next Review:** [DATE + 3 MONTHS]

---

## How to Use This Template

**Instructions for project teams:**

1. Replace all `[PLACEHOLDER]` values with your project-specific information
2. Remove sections that don't apply to your architecture
3. Modify examples to match your tech stack
4. Adjust performance targets for your domain
5. Add domain-specific constraints below each principle
6. Customize the evolution path to match your sprint schedule

**Key sections to customize:**
- Architectural Constraints (Lines 108-151)
- Non-Functional Requirements (Lines 154-178)
- Technology Decisions (Lines 182-224)
- Phase evolution path (Lines 350-368)

---

## Core Principles

### 1. [PRINCIPLE_1_NAME]

**[One-sentence principle statement]**

**Why:** [Business and technical reasons]

**Key characteristics:**
- ✅ [What should happen]
- ✅ [What should happen]
- ❌ [What should NOT happen]

**How to apply:**
- [Concrete guidance 1]
- [Concrete guidance 2]
- [Concrete guidance 3]

**Exceptions:**
- [Document any exceptions or edge cases]

---

### 2. [PRINCIPLE_2_NAME]

**[One-sentence principle statement]**

**Why:** [Business and technical reasons]

**Implementation:**
- [Pattern 1]
- [Pattern 2]
- [Pattern 3]

**Trade-offs:**
- [Pro] vs [Con]
- [Pro] vs [Con]

---

### 3. Security First

Security is not an afterthought; it's baked into the architecture.

**Core components for [PROJECT_TYPE] systems:**
- Authentication & authorization
- Data encryption (at-rest and in-transit)
- Input validation at boundaries
- Rate limiting to prevent abuse
- Audit logging for compliance
- [Domain-specific requirement]

**Security layers:**

| Layer | Responsibility | Technology |
|-------|-----------------|-----------|
| Application | Business logic security | [Your tech] |
| API | Authentication/authorization | [Your tech] |
| Network | Transport security | TLS 1.2+ |
| Data | Encryption & integrity | [Your crypto] |
| Infrastructure | Access controls | [Your infra] |

---

### 4. Scalability Through Separation

Each architectural tier scales independently based on its bottlenecks.

**Scaling strategy by tier:**

| Tier | Scaling Method | Bottleneck |
|------|---|---|
| [Frontend/UI] | [CDN/Caching] | [Bandwidth/Browser] |
| [API/Backend] | [Horizontal] | [CPU/Memory] |
| [Database] | [Replication/Sharding] | [Disk I/O] |
| [Cache] | [Distributed] | [Memory] |

**Decoupling approach:**
- [Communication pattern 1] (e.g., REST, gRPC, events)
- [Communication pattern 2]
- [Load balancing strategy]

---

### 5. Observability Built-In

All systems are observable from day one. You cannot improve what you cannot measure.

**Observable components:**
- ✅ Structured logging (machine-readable format)
- ✅ Distributed tracing for request flows
- ✅ Metrics collection (throughput, latency, errors)
- ✅ Health checks on critical services
- ✅ Alerting on SLA breaches
- ✅ User-facing status indicators

**Implementation requirements:**
- Logs include correlation IDs for request tracking
- Metrics tagged by service, version, and environment
- Traces show end-to-end request path
- Health checks fail fast (< 5 second timeout)
- Alerts have runbooks attached

---

### 6. Fail Fast, Recover Gracefully

Systems should fail predictably and recover automatically without manual intervention.

**Resilience patterns:**
- Circuit breaker for external dependencies
- Retry logic with exponential backoff
- Graceful degradation (fallback modes)
- Health-based automatic restarts
- Dead letter queues for failed messages
- Bulkheads to isolate failures

**Failure scenarios to handle:**
- [External service down → use cache/fallback]
- [Database unavailable → circuit breaker]
- [Rate limited → queue and retry]
- [Timeout → return partial data]
- [Invalid input → reject gracefully]

---

### 7. Data Consistency Over Speed

Correctness and consistency are prioritized over premature optimization.

**Data philosophy:**
- Database is source of truth
- Critical operations use ACID transactions
- Eventual consistency only where business allows
- Important changes logged to audit trail
- Data validation at multiple layers

**When eventual consistency is acceptable:**
- [Example: Cache invalidation]
- [Example: Reporting data]

**When strong consistency is required:**
- [Example: Financial transactions]
- [Example: Authorization decisions]

---

### 8. Simplicity and Clarity

Prefer proven simple solutions over clever custom implementations.

**Guidelines:**
- ❌ Avoid premature optimization (measure first)
- ✅ Use standard patterns familiar to team
- ✅ Well-documented design decisions (see ADRs)
- ✅ Clear naming (avoid abbreviations)
- ✅ Prefer battle-tested libraries over custom code
- ✅ Explicit code over implicit magic

**Decision heuristic:**
```
Simple solution understood by team    → CHOOSE
Clever solution only one engineer understands → AVOID

Well-tested library with 10k stars    → CHOOSE
Custom implementation for same problem       → AVOID
```

---

## Mandatory Layering Architecture

**[PROJECT_NAME] uses strict six-tier separation of concerns.**

### Six-Tier Model

```
Tier 1 (UI):         Swing/AWT components (JFrame, JPanel, JDialog)
Tier 2 (UI Logic):   UIControllers (HttpClient delegation, state management)
Tier 3 (API):        REST @RestController (HTTP endpoints, validation)
Tier 4 (Business):   @Service classes (business logic, rules)
Tier 5 (Data):       @Repository (persistence, queries)
Tier 6 (Database):   [DATABASE_TYPE: MySQL, PostgreSQL, etc.]
```

### Why This Structure?

- **Tier 1 (UI):** Handles rendering and user events only. No business logic.
- **Tier 2 (UIControllers):** Framework-agnostic state management. Testable without UI.
- **Tier 3 (API):** HTTP contract layer. Defines endpoints and validation.
- **Tier 4 (Business):** All business logic and rules centralized. Reusable.
- **Tier 5 (Data):** ONLY place for database operations. Enforces data isolation.
- **Tier 6 (Database):** Raw data storage.

### Enforcement Rules (MANDATORY)

Each tier handles ONE responsibility. No tier crosses into another's domain.

**Tier 1 (UI) Requirements:**
- ✅ Layout, rendering, event handling ONLY
- ❌ NO business logic, NO database access, NO service instantiation
- ❌ NEVER import `gs.Application.AppView` (breaks testability)
- Delegates all logic to Tier 2 (UIControllers)

**Tier 2 (UIControllers) Requirements:**
- ✅ Framework-agnostic (ZERO Swing/AWT imports, ZERO database imports)
- ✅ Communicate with Tier 3 via HttpClient for REST calls
- ✅ Manage state and UI listeners
- ❌ NO database code, NO service instantiation
- **Benefit:** Testable without UI framework or database

**Tier 3 (API) Requirements:**
- ✅ HTTP endpoints and request/response handling
- ✅ API-level validation
- ✅ Delegates to Tier 4 (Services)
- ❌ NO database access, NO direct repository instantiation

**Tier 4 (Business Services) Requirements:**
- ✅ Business logic, validation, rules
- ✅ Delegates to Tier 5 (Repositories) via dependency injection
- ❌ NO database code, NO direct repository creation
- ✅ Use constructor injection for repository access

**Tier 5 (Repositories) Requirements:**
- ✅ ALL database operations (SQL, RecordSet, JDBC) ONLY here
- ✅ Queries, persistence, data access
- ✅ Tenant scoping enforced in all queries
- ❌ NO business logic, NO service logic

### Dependency Injection (MANDATORY)

- **Always use constructor-based injection**
- Never use `new ServiceClass()`, `new RepositoryImpl()`, or `new DatabaseConnection()`
- Exception: `new DatabaseConnection()` ONLY in @Configuration classes
- Enables testability and modularity

### Violations (Never Do These)

- ❌ Database code in UI layer
- ❌ Service instantiation in Swing code
- ❌ AppView import in Swing classes
- ❌ Direct database access from Service or API layers
- ❌ Spring context access from Swing components
- ❌ PreparedStatement or JDBC in UI code

---

## Audit & Observability Architecture

### Audit Logging (MANDATORY)

Every write operation MUST be audited.

**Record for every write:**
- Actor: Who triggered the change (user ID, service account)
- Action: What operation (create, update, delete)
- Entity: Which object changed (aggregate ID, type)
- Timestamp: When it happened (UTC)
- Correlation ID: Link to originating request
- Tenant: Which tenant owns the data

**Example audit log entry:**
```
timestamp: 2026-07-12T10:30:00Z
actor: user-123
action: ORDER_CREATED
entity: order-456
tenant: tenant-001
correlationId: req-67890
details: { total: 99.99, items: 3 }
```

### Structured Logging Standards

**Log format:** Include when permitted (respect privacy)
- Tenant ID (identify which customer)
- Correlation ID (trace requests)
- Actor ID (who triggered action)
- Event type (what happened)

**MUST NOT expose:**
- Secrets, API keys, tokens
- Personal data (unless required for audit)
- Passwords, credit cards, SSNs

---

## Events Architecture

**[PROJECT_NAME] uses domain events for state change notifications.**

### Event Emission Rules (MANDATORY)

- **Emit ONLY after successful transaction** (database commit succeeded)
- **NEVER emit before commit** (would create orphaned events)
- Events are immutable once emitted

### Event Payload Requirements

**Every event MUST include:**
- `eventId`: Unique identifier (UUID)
- `eventType`: Event name (OrderCreated, UserDeleted)
- `timestamp`: When event occurred (UTC)
- `aggregateId`: What entity changed (ID only)
- `aggregateType`: Entity type
- `tenantId`: Multi-tenant identifier
- `correlationId`: Link to originating request
- `actorId`: Who triggered event
- `version`: Event format version

**NEVER include in events:**
- Secrets, API keys, tokens
- Credit card numbers
- Passwords, PINs
- Personal identification numbers

### Event Consumers

List which services consume which events and why:

| Event | Consumers | Reason |
|-------|-----------|--------|
| [EventName] | [Service1], [Service2] | [Business process] |

---

## Testing Architecture

### Test Determinism (MANDATORY)

Tests MUST produce identical results every run.

**Requirements:**
- Inject `Clock.fixed()` instead of `System.currentTimeMillis()`
- Inject seeded random instead of `new Random()`
- Mock external dependencies: [LIST_EXTERNAL_SERVICES]
- No execution order dependency
- No mutable global state

### Cross-Tenant Isolation Tests (MANDATORY)

Every data-access feature MUST include test verifying cross-tenant access is DENIED.

**Test requirement:**
- User from Tenant A cannot read/write Tenant B data
- System rejects cross-tenant access before database query

### Secrets Protection Tests (MANDATORY)

Assert secrets do NOT appear in: logs, errors, API responses, audit events

---

## Architectural Constraints

**Domain:** [PROJECT_DOMAIN]  
**Users:** [TARGET_USERS]  
**Deployment:** [DEPLOYMENT_TYPE: Cloud/On-prem/Hybrid]

### Constraint 1: [NAME]

**Requirement:** [What must be supported]

**Why:** [Business driver]

**Pattern:** [How to implement]
- [Pattern element 1]
- [Pattern element 2]
- [Pattern element 3]

**Alternatives considered:** [Other options and why rejected]

**Validation:** [How to verify this is working]

---

### Constraint 2: [NAME]

**Requirement:** [What must be supported]

**Implementation approach:**
- [Approach 1 with rationale]
- [Approach 2 with rationale]
- [Chosen approach with reason]

**Success criteria:**
- [ ] [Measurable goal 1]
- [ ] [Measurable goal 2]

---

### Constraint 3: [NAME - DOMAIN SPECIFIC]

[Add 1-2 domain-specific architectural constraints]

---

## Non-Functional Requirements

### Performance

| Metric | Target | Rationale |
|--------|--------|-----------|
| [Operation] response time | [X]ms p99 | [Why this matters] |
| [Operation] throughput | [X] req/sec | [Capacity planning] |
| [Resource] load time | [X] seconds | [User experience] |
| [Query] latency | [X]ms p99 | [SLA requirement] |
| [Bulk operation] duration | [X] minutes | [Batch processing] |

**Performance testing strategy:**
- Load testing at [X]x expected peak
- Latency profiling per endpoint
- Database query optimization
- Cache effectiveness measurement

---

### Reliability & Availability

| Metric | Target | Consequence |
|--------|--------|-------------|
| Uptime | [99.x]% | [What this means in minutes/month] |
| Error rate | [X]% of requests | [User impact] |
| P0 incident response | [X] minutes | [On-call SLA] |
| Mean time to recovery (MTTR) | [X] minutes | [Acceptable downtime] |
| Data loss tolerance | [X] | [Backup retention policy] |

**Reliability measures:**
- Multi-region deployment (if needed)
- Database replication setup
- Backup & restore procedures
- Incident runbooks for common issues

---

### Security

| Control | Standard | Implementation |
|---------|----------|-----------------|
| Encryption in transit | TLS 1.2+ | [Your approach] |
| Encryption at rest | [AES-256] | [Key management] |
| Authentication | [JWT/OAuth/SAML] | [Token lifecycle] |
| Authorization | [RBAC/ABAC] | [Permission model] |
| Audit logging | [HIPAA/GDPR compliant] | [Log retention] |
| Penetration testing | [Annual/Quarterly] | [Scope] |

**Security requirements by role:**
- Admin: Full access
- User: Own data + shared data
- Guest: Read-only public data
- [Custom roles as needed]

---

### Scalability

| Dimension | Target | Monitoring |
|-----------|--------|-----------|
| Concurrent users | [X,XXX]+ | [APM metric] |
| Daily active users | [X,XXX,XXX] | [Analytics] |
| Operations/second | [X,XXX] | [Metrics] |
| Data storage | [X]GB/[X]TB | [Disk usage] |
| [Custom metric] | [Target] | [How measured] |

**Scaling bottlenecks & solutions:**

| Bottleneck | Current Limit | Solution |
|-----------|---|---|
| [Database IOPS] | [Number] | [Replication/Sharding] |
| [API response time] | [X]ms | [Caching/CDN] |
| [Memory usage] | [X]GB | [Distributed cache] |
| [Disk space] | [X]GB | [Archival policy] |

---

## Technology Decisions

### Backend

**Language:** [e.g., Java 11, Python 3.9, Go 1.19]

**Framework:** [e.g., Spring Boot, Django, Gin]

**Database:**
- Primary: [e.g., PostgreSQL 14, MySQL 8, MongoDB 5]
- Cache: [e.g., Redis, Memcached, In-memory]
- Search: [e.g., Elasticsearch, Algolia]

**Other Services:**
- [Message queue: RabbitMQ, Kafka, etc.]
- [Task scheduler: Celery, Bull, etc.]
- [File storage: S3, GCS, etc.]

**Rationale:**
- ✅ [Reason 1: Mature ecosystem, type safety, etc.]
- ✅ [Reason 2: Team expertise, community support, etc.]
- ✅ [Reason 3: Performance, scalability, etc.]

---

### Frontend (Web)

**Framework:** [e.g., React 18, Angular 16, Vue 3, Svelte]

**Language:** [e.g., TypeScript, JavaScript]

**Styling:** [e.g., CSS-in-JS, CSS Modules, Tailwind, Sass]

**State Management:** [e.g., Redux, NgRx, Jotai, none]

**Build Tool:** [e.g., Webpack, Vite, Next.js, Angular CLI]

**Package Manager:** [npm, yarn, pnpm]

**Rationale:**
- ✅ [Ecosystem maturity and stability]
- ✅ [Team experience and learning curve]
- ✅ [Build performance and DX]

---

### Mobile (if applicable)

**Approach:** [Native/Cross-platform/Hybrid]

**Platforms:** [iOS, Android, or both]

**Framework:** [React Native, Flutter, NativeScript, Xamarin]

**Language:** [Swift/Kotlin, Dart, TypeScript, C#]

**Key Libraries:** [Navigation, state, etc.]

**Rationale:**
- ✅ [Code sharing or native performance]
- ✅ [Time to market considerations]
- ✅ [Device API requirements]

---

### DevOps & Infrastructure

**Container Runtime:** [Docker, Podman, etc.]

**Container Orchestration:** [Kubernetes, Docker Compose, ECS, App Engine]

**CI/CD Platform:** [GitHub Actions, GitLab CI, Jenkins, CircleCI]

**Monitoring Stack:**
- Metrics: [Prometheus, Datadog, CloudWatch, etc.]
- Logging: [ELK, Splunk, Datadog, Cloudwatch, etc.]
- Tracing: [Jaeger, Zipkin, Datadog, New Relic, etc.]

**Infrastructure as Code:** [Terraform, CloudFormation, Helm, etc.]

**Rationale:**
- ✅ [Why these tools fit your scale and team]
- ✅ [Integration with your platform]
- ✅ [Cost and complexity trade-offs]

---

## Architectural Patterns

### Backend Patterns

| Pattern | Purpose | Example in codebase |
|---------|---------|---|
| [Repository Pattern] | Data access abstraction | [Location] |
| [Service Layer] | Business logic encapsulation | [Location] |
| [Dependency Injection] | Loose coupling | [Location] |
| [Circuit Breaker] | Fault tolerance | [Location] |
| [Caching Layer] | Performance | [Location] |
| [Domain-Driven Design] | [Add if relevant] | [Location] |

**Antipatterns to avoid:**
- ❌ God objects (services doing too much)
- ❌ Database as message queue
- ❌ Tight coupling to external services
- ❌ Silent error swallowing

---

### Frontend Patterns

| Pattern | Purpose | Example |
|---------|---------|---------|
| [Component Pattern] | Reusable UI blocks | [Location] |
| [Smart/Dumb Components] | Separation of concerns | [Location] |
| [State Management] | Predictable data flow | [Location] |
| [Facade Pattern] | Simplified service access | [Location] |
| [Container Pattern] | Logic isolation | [Location] |

**Component hierarchy example:**
```
App (Smart)
├── Layout (Dumb)
├── PageContainer (Smart)
│   ├── PageComponent (Dumb)
│   └── FormContainer (Smart)
│       └── FormComponent (Dumb)
└── [...]
```

---

### Integration Patterns

| Pattern | Use Case | Technology |
|---------|----------|-----------|
| [REST API] | Synchronous, CRUD | HTTP/JSON |
| [WebSocket] | Real-time bidirectional | WebSocket/STOMP |
| [Message Queue] | Async processing | [Kafka/RabbitMQ] |
| [Event Stream] | Event-driven systems | [Kafka/Pub-Sub] |
| [GraphQL] | Flexible queries | GraphQL |
| [gRPC] | High-performance RPC | Protocol Buffers |

**Recommended combinations:**
- REST + WebSocket for real-time apps
- REST + Message Queue for async jobs
- Event streaming for microservices
- [Your architecture's approach]

---

## Code Quality Standards

### Testing Strategy

**Test Coverage Targets:**

| Test Type | Minimum Coverage | Focus Areas |
|-----------|---|---|
| Unit Tests | [X]% of services | Business logic, edge cases |
| Integration Tests | [X]% of critical paths | API contracts, DB interactions |
| E2E Tests | [X]% of happy paths | User workflows |
| Load Tests | Before each release | Performance SLAs |
| Security Tests | Critical operations | Auth, data isolation |

**Testing framework stack:**
- Backend: [JUnit/pytest/Jest, Mockito/Sinon]
- Frontend: [Jest/Jasmine/Vitest]
- E2E: [Cypress/Playwright/Selenium]
- Load: [JMeter/Locust/k6]

---

### Code Review Process

**Before merging:**
- [ ] [X] reviewers approve (minimum)
- [ ] All CI checks pass
- [ ] No self-approval
- [ ] Test coverage maintained
- [ ] Documentation updated

**Automated checks:**
- Linting: [ESLint, Prettier, Checkstyle]
- Type checking: [TypeScript, MyPy]
- Security scanning: [SonarQube, OWASP, Snyk]
- Dependency check: [Dependabot, WhiteSource]

---

### Documentation Requirements

**What to document:**
- ✅ Architecture decisions (see ADR format below)
- ✅ API contracts (OpenAPI/Swagger)
- ✅ Database schema & migrations
- ✅ Setup & deployment procedures
- ✅ Incident playbooks
- ✅ Complex algorithms or business logic

**ADR Template (for architecture decisions):**
```markdown
# ADR [#]: [Title]

## Context
[The issue motivating this decision]

## Decision
[The change that we're proposing]

## Consequences
- Positive: [Pro 1, Pro 2]
- Negative: [Con 1, Con 2]

## Alternatives Considered
- [Option 1] - why rejected
- [Option 2] - why rejected

## Status
[Proposed/Accepted/Deprecated/Superseded]
```

---

## Decision Framework

### When to Add Complexity

Add architectural complexity **only when ALL** of these are true:

1. ✅ Problem is well-understood (not speculative)
2. ✅ Solution is proven in production (not novel)
3. ✅ Team completely understands it (can explain to junior)
4. ✅ Benefit clearly outweighs maintenance cost (10x+)

**Example:** Adding Redis is justified when caching proves to be the bottleneck and in-memory cache saturates memory.

**Example:** Kafka is NOT justified for a startup's first system without known async processing needs.

---

### When to Refactor

Refactor code **when you have time and ALL** of these are true:

1. ✅ Code is used by multiple places (value in consolidation)
2. ✅ Same pattern appears 3+ times (reduce duplication)
3. ✅ Refactoring reduces bugs (clear improvement)
4. ✅ Team has capacity (not on critical deadline)

**Anti-pattern:** Refactoring during crunch to fix "code smell" instead of the actual bug.

---

### When to Optimize

Optimize **only when you can answer both**:

1. ✅ Measurement shows this IS the bottleneck
2. ✅ Optimization is 10%+ improvement (significant)
3. ✅ Acceptable trade-offs (memory/complexity/accuracy)
4. ✅ Not premature (optimize when problem is proven)

**Anti-pattern:** "This code might be slow" → rewrite without profiling first.

---

## Deployment Philosophy

### Zero-Downtime Deployments

**Strategy:** [Blue-green / Rolling / Canary / Feature flags]

**Process:**
1. [Deploy to staging, run smoke tests]
2. [Start traffic shift / canary / blue-green deployment]
3. [Monitor metrics for errors/latency increase]
4. [Rollback if problems detected]
5. [Complete deployment once stable]

**Rollback procedure:**
- Automatic rollback if error rate > [X]%
- Automatic rollback if latency > [X]ms
- Manual rollback with [X] minute SLA

---

### Database Migrations

**Principles:**
- All changes are backward compatible (supports two versions)
- Migrations are separate from code deployments
- Every migration has a tested rollback plan
- Data validation runs before and after

**Migration checklist:**
- [ ] Code runs with old schema (expansion phase)
- [ ] Migration is idempotent
- [ ] Data validated pre/post migration
- [ ] Rollback tested
- [ ] On-call notified
- [ ] Performance tested at scale

**Example timeline:**
```
Day 1: Add new column (nullable, default value)
Day 2: Code reads/writes new column
Day 3: Backfill existing data
Day 4: Add NOT NULL constraint
```

---

### Monitoring First Approach

**Before deploying:**
- ✅ Metrics & dashboards ready
- ✅ Alert thresholds tested
- ✅ On-call runbooks written
- ✅ Logging covers happy + error paths
- ✅ Trace correlation IDs configured

**Deployment gates:**
- [ ] Monitoring checks pass
- [ ] Logs appear in aggregation service
- [ ] Metrics flowing to dashboard
- [ ] Alert rules firing correctly

---

## Anti-Patterns (Avoid These)

| Anti-Pattern | Problem | Right Alternative |
|--------------|---------|-------------------|
| **God Objects** | One class/service doing everything | Single Responsibility Principle |
| **Tight Coupling** | Components hardcoded to internals | Dependency Injection + Interfaces |
| **Premature Optimization** | Optimizing before measuring | Profile first, optimize after |
| **Silent Failures** | Errors logged but not visible | Fail loud with alerts |
| **Magic Strings** | `if (user.type === "admin_internal_special_v2")` | Enum or configuration |
| **Overly Generic Code** | So abstract no one understands | Explicit over implicit |
| **Skipping Tests** | "I'll test in production" | Automated test suite |
| **Database as Queue** | Using RDBMS for async work | Purpose-built message queue |
| **Distributed Monolith** | Microservices without decoupling | Clear service boundaries |
| **Cargo Cult Architecture** | "Netflix does this" (copied without understanding) | Understand before adopting |

---

## Guiding Questions for Decisions

When proposing a new component, pattern, or technology, ask these questions:

1. **Does it violate these principles?**  
   If yes, reconsider unless you're intentionally changing a principle.

2. **Is it the simplest solution?**  
   If no, can we simplify further?

3. **Can we test it?**  
   If no, redesign so it becomes testable.

4. **Is it documented?**  
   If no, add architecture documentation + inline comments.

5. **Does the team understand it?**  
   If no, explain thoroughly or choose a simpler approach.

6. **Will it scale to our targets?**  
   If uncertain, do a prototype/PoC first.

7. **What's the failure mode?**  
   If unknown, think through failure scenarios now.

8. **Can we monitor it?**  
   If no, add observability before shipping.

---

## Evolution Path

**Timeframe:** [Adjust based on your sprint schedule]

### Phase 1: Foundation (Sprints 1-4)

**Goals:** Get core system working with quality foundations

- ✅ [Core feature 1]
- ✅ [Core feature 2]
- ✅ Security foundation (auth, encryption)
- ✅ Basic testing & CI/CD
- ✅ Logging and basic monitoring

**Validation:** Product works end-to-end, deployable to production

---

### Phase 2: Scale & Polish (Sprints 5-6)

**Goals:** Improve performance, reliability, and operations

- ✅ Performance optimization (based on profiling)
- ✅ Advanced features (based on feedback)
- ✅ User experience improvements
- ✅ Operations maturity (alerts, runbooks)
- ✅ Distributed tracing & logging
- ✅ Load testing & capacity planning

**Validation:** Can handle expected production load

---

### Phase 3: Expansion (Future)

**Goals:** Prepare for scale and global reach

- 🔄 [Advanced infrastructure: Kubernetes/Microservices]
- 🔄 [Advanced caching: Redis/CDN]
- 🔄 [Message queues: Kafka/RabbitMQ]
- 🔄 [Distributed systems: Service mesh, etc.]
- 🔄 [Multi-region deployment]

**Validation:** Can scale 10x without rearchitecting

---

## Accountability

**Architecture Governance:**

| Role | Responsibility |
|------|-----------------|
| **Architecture Owner** | [Title/Name] - Maintains principles, approves major changes |
| **Tech Lead** | [Per domain] - Advocates principles in code reviews |
| **Platform Team** | [If exists] - Enforces CI/CD and infrastructure policies |

**Review Schedule:**
- Architecture review: [Quarterly or when major changes needed]
- Principles update: [Annual or when strategic changes needed]
- Decision review: [When new ADRs proposed]

**Last Updated:** [DATE]  
**Next Review:** [DATE]  
**Changelog:**
- [Date] - [Change 1]
- [Date] - [Change 2]

---

## Related Documents

**You should create these documents:**

- `DESIGN_DECISIONS.md` — Archive of Architecture Decision Records (ADRs)
- `API_STANDARDS.md` — REST API design guidelines and response formats
- `DATABASE_STANDARDS.md` — Schema patterns, migrations, indexing strategy
- `SECURITY_POLICY.md` — Security requirements, threat model, compliance
- `DEPLOYMENT_CHECKLIST.md` — Pre-production verification procedures
- `INCIDENT_RUNBOOKS.md` — How to handle common failure scenarios
- `TEAM_ONBOARDING.md` — How new engineers get up to speed

**This document references:**
- [Link to your repo](URL)
- [Link to Confluence/Wiki](URL)
- [Link to API docs](URL)
- [Link to deployment runbooks](URL)

---

## Quick Reference

**When you need to...**

| Goal | See Section |
|------|------------|
| Make an architectural decision | Guiding Questions (line ~XX) |
| Add a new feature | Decision Framework (line ~XX) |
| Choose a technology | Technology Decisions (line ~XX) |
| Debug a production incident | Anti-Patterns (line ~XX) + Incident Runbooks |
| Optimize performance | When to Optimize (line ~XX) |
| Write a component | Architectural Patterns (line ~XX) |
| Deploy code | Deployment Philosophy (line ~XX) |
| Add observability | Observability Built-In (line ~XX) |

---

**End of Template**

---

## Customization Checklist

Before finalizing your ARCHITECTURE_PRINCIPLES.md:

- [ ] Replace all `[PLACEHOLDERS]` with your project details
- [ ] Remove sections that don't apply
- [ ] Add domain-specific constraints (lines 108-151)
- [ ] Customize technology choices to match your stack (lines 182-224)
- [ ] Adjust performance targets to your SLAs (lines 154-178)
- [ ] Update phase evolution path (lines 350-368)
- [ ] Set specific dates and owners (Accountability section)
- [ ] Create related documents (Security Policy, API Standards, etc.)
- [ ] Have team review and approve
- [ ] Set calendar reminder for next review date

