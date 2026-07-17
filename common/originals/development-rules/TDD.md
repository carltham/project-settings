# Test-Driven Development (TDD) - MANDATORY

## Rule: All code changes MUST follow Test-Driven Development (RED-GREEN-REFACTOR cycle)

**Applies to:** Features, bug fixes, refactoring, infrastructure code

## The Process (MANDATORY)

1. **RED**: Write a failing test that defines expected behavior
2. **GREEN**: Write minimal code to make the test pass
3. **REFACTOR**: Improve code while keeping tests passing
4. **Repeat**: For each feature or fix

## Why This Rule Exists

- **Testability**: Code designed for tests is inherently testable
- **Correctness**: Failing test proves bug/feature is real before fix exists
- **Design**: Writing tests first forces better API design
- **Regression Prevention**: Tests catch future breaks immediately
- **Documentation**: Tests serve as executable specification of behavior
- **Prevents Over-Engineering**: Minimal code to pass test avoids gold-plating

## Violations (Never Do This)

**WRONG: Write code first, tests later**
- Code exists without test (defeats purpose of TDD)
- Tests added after implementation are verification, not design
- Never skip tests for "obvious" code

**WRONG: Weaken tests to make code pass**
- Overly vague assertions that prove nothing
- Mocks that bypass actual behavior
- Disabled tests (@Disabled)

## Correct TDD Workflow

**STEP 1: RED - Write failing test**
- Test expects specific behavior
- Test FAILS because implementation doesn't exist yet
- This is the requirement specification

**STEP 2: GREEN - Minimal code to pass**
- Write only what's needed to make test pass
- No extra features, no speculation
- Test PASSES

**STEP 3: REFACTOR - Improve while keeping tests green**
- Clean up variable names, extract methods, improve readability
- Tests still pass after refactoring
- Confidence comes from tests catching any breaks

## One Feature = Multiple Tests

Different test cases drive different code paths:
- Normal case behavior
- Edge cases and boundaries
- Error conditions
- Performance expectations

Each test drives one implementation detail; feature complete when all tests pass.

## Regression Fixes Use TDD

When fixing a bug:
1. Write test that reproduces the bug (fails with current code)
2. Fix the bug (test passes)
3. Commit both together (test + fix)

Never commit the fix without committing the failing test first.

## Impact on Development Workflow

- **More tests**: Prevents future bugs
- **Slower initial development**: Offset by fewer debugging cycles
- **Clearer code**: Driven by test requirements
- **Better APIs**: Designed for test-ability
- **Confidence in refactoring**: Tests catch breaks immediately

## Related Standards

- See `../testing-rules.md` for test naming, isolation, and security gates
- See `ARCHITECTURE_AND_LAYERING.md` for code organization and architecture
- Commits require explicit user authorization (do not commit automatically)

---

**Last Updated:** 2026-07-17  
**Status:** Mandatory organizational standard (applies to all developers)
