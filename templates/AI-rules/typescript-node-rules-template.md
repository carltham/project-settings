# TypeScript/Node.js Rules Template

Mandatory constraints for TypeScript/Node.js projects. Customize for your project's needs.

**Template Usage**: Copy this file to your project's `/AI-rules/typescript-node-rules.md` and customize for your framework (Express, Fastify, NestJS, etc.).

---

## Type Safety (MANDATORY)

### Rule 1: Strict TypeScript Configuration Required
- **MANDATORY**: `tsconfig.json` must have:
  - `"strict": true`
  - `"noImplicitAny": true`
  - `"noUnusedLocals": true`
  - `"noUnusedParameters": true`
- **NEVER**: Any use of `any` type without explicit `// @ts-ignore` comment
- **Why**: Catch type errors at compile time
- **Enforcement**: CI/CD pipeline, TypeScript compiler

### Rule 2: All Types Must Be Explicitly Defined
- **MANDATORY**: Export types explicitly, no inferred types for public APIs
- **MANDATORY**: Function return types must be declared
- **NEVER**: Omit return types from public functions
- **Why**: Self-documenting code, enables API contracts
- **Enforcement**: Code review, TypeScript strict mode

---

## Architecture & Layering (MANDATORY)

### Rule 3: Strict Layer Separation
- **MANDATORY**: Maintain clear separation of concerns (customize for your architecture)
- **NEVER**: Database code in business logic layer
- **NEVER**: Business logic in HTTP/API layer
- **Example Layers**: Controllers → Services → Repositories → Database
- **Why**: Enables testing, maintains modularity
- **Enforcement**: Code review, architecture review

### Rule 4: No Direct Service/Repository Instantiation
- **MANDATORY**: Constructor-based dependency injection ONLY
- **NEVER**: `new ServiceClass()` in application code
- **Exception**: `new DatabaseConnection()` ONLY in @Configuration classes
- **Why**: Enables testing, reduces coupling
- **Enforcement**: Code review

### Rule 5: Database Access Centralized in Repository Layer
- **MANDATORY**: All database queries in @Repository or data access classes
- **NEVER**: Direct database calls in business logic
- **NEVER**: Raw SQL queries outside data layer
- **Why**: Centralized audit trail, consistency
- **Enforcement**: Code review, grep for db imports

---

## Statefulness (MANDATORY)

### Rule 6: Stateless Where Appropriate
- **MANDATORY**: NO in-memory session/user state in [stateless servers/API gateways]
- **MANDATORY**: All state goes in [database/external store]
- **Why**: Enables horizontal scaling
- **Enforcement**: Architecture review, distributed testing

---

## Dependency Injection (MANDATORY)

### Rule 7: Constructor-Based Injection Only
- **MANDATORY**: All dependencies injected via constructor
- **NEVER**: `new ClassName()` in service code
- **NEVER**: Global singletons except in @Configuration/@Module
- **Example**:
  ```typescript
  export class UserService {
    constructor(private db: Database, private logger: Logger) {}
  }
  ```
- **Why**: Enables mocking for tests
- **Enforcement**: Code review

---

## Error Handling (MANDATORY)

### Rule 8: Custom Error Classes for All Domain Errors
- **MANDATORY**: Create typed error classes (not generic Error)
- **MANDATORY**: Include error code, status code, message
- **NEVER**: Throw raw strings or generic errors
- **Why**: Enables proper HTTP/API responses
- **Enforcement**: Code review, integration testing

### Rule 9: Never Swallow Errors
- **MANDATORY**: Catch and log errors, then re-throw or throw derived error
- **NEVER**: Empty catch blocks
- **Action on Error**: Log with context, then throw
- **Why**: Prevents silent failures
- **Enforcement**: Code review checklist

---

## Logging and Secrets (MANDATORY)

### Rule 10: NEVER Log Credentials, Tokens, or Secrets
- **MANDATORY**: No passwords, tokens, or API keys in logs
- **MANDATORY**: No personal data in logs (unless specifically audited)
- **Action**: Log operation name, user_id, context (not tokens)
- **Why**: Security breach prevention
- **Enforcement**: Code review (grep for secrets), security review

### Rule 11: Structured Logging Format
- **MANDATORY**: All logs in structured format (JSON or key=value)
- **MANDATORY**: Include timestamp, level, message, context
- **NEVER**: Dump entire objects to logs
- **Why**: Machine-readable, queryable logs
- **Enforcement**: Logging utility enforces format

### Rule 12: Auth Events and Permission Denials Must Be Logged
- **MANDATORY**: Log every authentication attempt (success/failure)
- **MANDATORY**: Log permission denials
- **Why**: Audit trail, security accountability
- **Enforcement**: Integration testing, security review

---

## Testing Requirements (MANDATORY)

### Rule 13: Mocking of External Dependencies
- **MANDATORY**: Tests must mock external services (database, HTTP, APIs)
- **NEVER**: Tests hit real databases or external APIs
- **Why**: Tests are deterministic, fast, isolated
- **Enforcement**: Code review, test setup review

### Rule 14: No Production Data in Tests
- **MANDATORY**: No real credentials, personal data, live accounts
- **MANDATORY**: Use test fixtures with synthetic data only
- **Why**: Security, compliance, no accidental production impact
- **Enforcement**: Code review, secret scanning

---

## Code Review Checklist

Before merging any TypeScript/Node.js code:

- ✅ Strict TypeScript configuration enforced
- ✅ All types explicitly defined (no `any`)
- ✅ Function return types declared
- ✅ Layer separation maintained
- ✅ No business logic in HTTP layer
- ✅ No database code in business logic
- ✅ Dependency injection via constructors only
- ✅ Custom error classes used
- ✅ No empty catch blocks
- ✅ No credentials in logs
- ✅ Structured logging format
- ✅ Auth/permission events logged
- ✅ Tests mock external dependencies
- ✅ No production data in tests
- ✅ No TypeScript errors or warnings

---

## Customization Notes

**For your project**, consider:
- [ ] What framework? (Express, Fastify, NestJS, etc.)
- [ ] What ORM/database? (Prisma, TypeORM, Raw queries)
- [ ] What testing framework? (Jest, Mocha, Vitest)
- [ ] Logging framework? (pino, winston, bunyan)
- [ ] DI container? (Manual, InversifyJS, etc.)
- [ ] API style? (REST, GraphQL, gRPC)
- [ ] Auth pattern? (JWT, OAuth, custom)
- [ ] What "stateless" means for your project
- [ ] Project-specific layer requirements

---

**Template Last Updated:** 2026-07-17  
**Status:** Copy and customize for your project
