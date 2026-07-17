# Angular Rules Template

Mandatory constraints for Angular projects. Customize for your project's needs.

**Template Usage**: Copy this file to your project's `/AI-rules/angular-rules.md` and customize for your Angular version and architecture.

---

## Component Lifecycle (MANDATORY)

### Rule 1: ALL Components Must Implement OnDestroy with Cleanup
- **MANDATORY**: Every component must implement `OnDestroy`
- **MANDATORY**: Every component must have a `destroy$` subject
- **MANDATORY**: Every subscription must use `takeUntil(this.destroy$)`
- **NEVER**: Leave subscriptions without cleanup
- **Action on Destroy**: Call `destroy$.next()` and `destroy$.complete()`
- **Why**: Prevents memory leaks, multiple subscriptions
- **Enforcement**: Code review checklist, memory profiler testing

### Rule 2: Observable Subscriptions Must Use takeUntil
- **MANDATORY**: All `.subscribe()` calls must be wrapped with `takeUntil(this.destroy$)`
- **NEVER**: Subscribe without `takeUntil`
- **Example**:
  ```typescript
  this.service.data$
    .pipe(takeUntil(this.destroy$))
    .subscribe(data => ...);
  ```
- **Why**: Automatic cleanup on component destruction
- **Enforcement**: Code review, linter rule

### Rule 3: Never Subscribe to Observables in Templates
- **MANDATORY**: Use `async` pipe in templates instead of manual subscriptions
- **NEVER**: `.subscribe()` in component, then assign to property
- **Example CORRECT**: `{{ data$ | async }}`
- **Example WRONG**: `this.data$.subscribe(d => this.data = d)`
- **Why**: Automatic subscription cleanup
- **Enforcement**: Code review, linter rule

---

## Dependency Injection (MANDATORY)

### Rule 4: Constructor-Based Injection Only
- **MANDATORY**: All dependencies injected via constructor
- **NEVER**: Global services, singletons in component code
- **MANDATORY**: Services marked with `@Injectable({ providedIn: 'root' })`
- **Why**: Enables testing via mock injection
- **Enforcement**: Code review

### Rule 5: Services Must Be @Injectable with providedIn: 'root'
- **MANDATORY**: All services use `@Injectable({ providedIn: 'root' })`
- **NEVER**: Manual provider registration in module declarations
- **Why**: Automatic singleton, no module coupling
- **Enforcement**: Code review, linter rule

---

## Type Safety (MANDATORY)

### Rule 6: Strict TypeScript Configuration
- **MANDATORY**: Same as TypeScript rules (strict, noImplicitAny, etc.)
- **MANDATORY**: All component properties and method parameters typed
- **NEVER**: Any use of `any`
- **Why**: Catch type errors at compile time
- **Enforcement**: TypeScript compiler, CI/CD

### Rule 7: Input/Output Properties Must Be Typed
- **MANDATORY**: All `@Input()` and `@Output()` must have types
- **MANDATORY**: Provide default values for inputs
- **MANDATORY**: Document input/output behavior
- **Example**:
  ```typescript
  @Input() data: Data = defaultData;
  @Output() updated = new EventEmitter<Data>();
  ```
- **Why**: Self-documenting, enables checking
- **Enforcement**: Code review, TypeScript strict mode

---

## State Management (MANDATORY)

### Rule 8: State Must Flow via Observables
- **MANDATORY**: Use RxJS Observables/Subjects for state management
- **NEVER**: Direct mutable state shared between components
- **MANDATORY**: Use `BehaviorSubject` or `Subject` for state
- **Why**: Reactive, testable
- **Enforcement**: Code review, state management review

### Rule 9: No Direct Component-to-Component Communication
- **MANDATORY**: Parent-child via `@Input()` and `@Output()` ONLY
- **NEVER**: Shared service references between siblings
- **NEVER**: Parent accessing child properties directly
- **Why**: Loose coupling, maintainable hierarchy
- **Enforcement**: Code review, architecture review

---

## Testing Requirements (MANDATORY)

### Rule 10: Components Must Have Tests
- **MANDATORY**: Unit tests for all components
- **MANDATORY**: Test component initialization and cleanup
- **MANDATORY**: Mock services and observables
- **NEVER**: Skip lifecycle/subscription tests
- **Why**: Catch lifecycle bugs, memory leaks
- **Enforcement**: Code coverage gates, code review

### Rule 11: Never Use Real Services in Tests
- **MANDATORY**: Mock all services using jasmine.SpyObj
- **NEVER**: Inject real HTTP client, real services
- **Why**: Tests deterministic, fast, isolated
- **Enforcement**: Code review, test setup review

### Rule 12: No Hardcoded Timeouts in Tests
- **MANDATORY**: Use done callback or async/fakeAsync
- **NEVER**: `setTimeout(test, 1000)`
- **Why**: Tests deterministic, run quickly
- **Enforcement**: Code review, linter rule

---

## Observables & RxJS (MANDATORY)

### Rule 13: Combine Multiple Observables
- **MANDATORY**: Use `combineLatest`, `merge`, `concat` for multiple sources
- **NEVER**: Subscribe to one observable, then manually subscribe to another
- **Why**: Reactive, composable, correct ordering
- **Enforcement**: Code review

### Rule 14: Use Operators, Never Break the Pipe
- **MANDATORY**: Chain operators with `pipe()`
- **NEVER**: Break observable chain to reassign intermediate values
- **Why**: Cleaner, easier to follow
- **Enforcement**: Code review

---

## Performance (MANDATORY)

### Rule 15: Use OnPush Change Detection Where Possible
- **MANDATORY**: Set `changeDetection: ChangeDetectionStrategy.OnPush` on components
- **MANDATORY**: Ensure inputs are @Input properties (immutable references)
- **Why**: Faster rendering, fewer checks
- **Enforcement**: Performance review, code review

---

## Logging and Secrets (MANDATORY)

### Rule 16: NEVER Log Credentials or Sensitive Data
- **MANDATORY**: No tokens, passwords, or PII in console logs
- **MANDATORY**: Only log operation names, error messages
- **Why**: Security, no exposure via browser dev tools
- **Enforcement**: Code review, security review

---

## Code Review Checklist

Before merging any Angular code:

- ✅ All components implement `OnDestroy` with `destroy$` subject
- ✅ All subscriptions use `takeUntil(destroy$)`
- ✅ Templates use `async` pipe (no manual subscriptions)
- ✅ All dependencies injected via constructor
- ✅ Services use `@Injectable({ providedIn: 'root' })`
- ✅ Strict TypeScript enabled
- ✅ All types explicitly defined
- ✅ `@Input()` and `@Output()` typed with defaults
- ✅ State flows via Observables
- ✅ No direct component-to-component communication
- ✅ Unit tests present
- ✅ Services mocked in tests
- ✅ No hardcoded timeouts in tests
- ✅ Multiple observables combined with combineLatest/merge
- ✅ OnPush change detection where appropriate
- ✅ No credentials in logs
- ✅ No memory leaks (profiler tested)

---

## Customization Notes

**For your project**, consider:
- [ ] Angular version? (v15, v16, v17+)
- [ ] State management? (NgRx, Akita, simple services)
- [ ] Testing framework? (Jasmine/Karma, Jest, Vitest)
- [ ] UI library? (Material, PrimeNG, custom)
- [ ] Routing patterns? (lazy loading, guards)
- [ ] HTTP client patterns? (interceptors, error handling)
- [ ] Module structure? (feature modules, standalone)
- [ ] Performance requirements? (when to use OnPush)
- [ ] Project-specific change detection strategy

---

**Template Last Updated:** 2026-07-17  
**Status:** Copy and customize for your project
