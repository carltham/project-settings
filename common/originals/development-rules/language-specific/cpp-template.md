# C++ Rules Template

Mandatory constraints for C++17+ projects. Customize for your project's needs.

**Template Usage**: Copy this file to your project's `/AI-rules/cpp-rules.md` and customize the examples and tech stack details.

---

## Memory Safety (MANDATORY)

### Rule 1: NO Raw Pointers in Application Code
- **MANDATORY**: Use `std::unique_ptr<T>` for exclusive ownership, `std::shared_ptr<T>` for shared ownership
- **NEVER**: `new`, `delete`, or raw pointers in non-system code
- **Exception**: Only in @Configuration-equivalent startup code
- **Why**: Prevents memory leaks, use-after-free, double-delete
- **Enforcement**: Code review checklist item

### Rule 2: RAII Pattern - All Resources Must Acquire in Constructor, Release in Destructor
- **MANDATORY**: Every resource (memory, file, connection, socket) must follow RAII
- **NEVER**: Manual cleanup via cleanup() methods
- **NEVER**: Resource leaks on exception
- **Why**: Guarantees deterministic cleanup even on error
- **Enforcement**: Design review before implementation

### Rule 3: Destructors Must Never Throw
- **MANDATORY**: All destructors must be `noexcept`
- **NEVER**: throw from destructor
- **Why**: Prevents stack unwinding chaos and program termination
- **Enforcement**: Compiler warning (treat as error)

---

## Naming Conventions (MANDATORY)

### Rule 4: Classes and Types - PascalCase Only
- **MANDATORY**: All class names `PascalCase` (e.g., `SessionManager`, `FrameBuffer`)
- **MANDATORY**: Exception classes end with `Exception` (e.g., `NotFoundException`)
- **MANDATORY**: Abstract classes/interfaces prefix with `I` (e.g., `IHandler`)
- **NEVER**: `UPPER_CASE`, `snake_case`, or `camelCase` for class names
- **Why**: Consistency, readability, IDE support
- **Enforcement**: Code review, naming checklist

### Rule 5: Functions and Methods - camelCase Only
- **MANDATORY**: All function names `camelCase` (e.g., `process()`, `validate()`)
- **MANDATORY**: Getters use `get`/property style (e.g., `getId()` or `id()`)
- **MANDATORY**: Predicates start with `is`/`has` (e.g., `isValid()`, `hasData()`)
- **NEVER**: `UPPER_CASE`, `snake_case`, or `PascalCase` for functions
- **Why**: Consistency, standard C++ convention
- **Enforcement**: Code review checklist

### Rule 6: Variables and Members - snake_case Only
- **MANDATORY**: All variable names `snake_case` (e.g., `session_id`, `data_buffer`)
- **MANDATORY**: Constants `UPPER_SNAKE_CASE` (e.g., `MAX_SIZE`, `DEFAULT_PORT`)
- **MANDATORY**: Member variables suffix with `_` (e.g., `session_id_`, `connections_`)
- **NEVER**: `camelCase` or `PascalCase` for variables
- **Why**: Consistency, member visibility at a glance
- **Enforcement**: Code review checklist

### Rule 7: File Names Must Match Public Class Name
- **MANDATORY**: Class `SessionManager` → file `session_manager.hpp`/`session_manager.cpp`
- **MANDATORY**: File names `lowercase_with_underscores`
- **Why**: IDE support, discoverability
- **Enforcement**: Build system check or code review

---

## Async I/O (MANDATORY)

**Note**: Customize this section based on your async I/O library (Boost.ASIO, event loops, etc.)

### Rule 8: Error Codes MUST Be Checked on All Async Operations
- **MANDATORY**: Check error codes after every async operation
- **NEVER**: Ignore error codes
- **NEVER**: Assume success
- **Action on Error**: Log context, gracefully close connections/handlers
- **Why**: Early detection of failures
- **Enforcement**: Code review, integration testing

### Rule 9: Synchronization for Concurrent Access
- **MANDATORY**: Prevent data races in multi-threaded contexts
- **MANDATORY**: Use appropriate synchronization (mutexes, atomics, thread pools)
- **NEVER**: Unprotected shared state
- **Why**: Prevents race conditions and undefined behavior
- **Enforcement**: Thread safety review, testing

### Rule 10: Keep Objects Alive During Async Operations
- **MANDATORY**: Use `enable_shared_from_this<>` for classes with async operations
- **MANDATORY**: Capture `shared_from_this()` in async callbacks
- **NEVER**: Let object be destroyed while async operations pending
- **Why**: Prevents use-after-free, segfaults in callbacks
- **Enforcement**: Code review, design review

---

## Header/Source Organization (MANDATORY)

### Rule 11: Headers Contain Interface Only
- **MANDATORY**: `.hpp` files: class declarations, function signatures, inline functions
- **NEVER**: Implementation code in headers (except template definitions)
- **Why**: Separation of concerns, faster compilation
- **Enforcement**: Build-time policy

### Rule 12: Implementations in Source Files Only
- **MANDATORY**: `.cpp` files: all function bodies, static members, constants
- **NEVER**: Implementation leakage into headers
- **Why**: Encapsulation, single point of change
- **Enforcement**: Code review checklist

---

## Build System & Package Management (MANDATORY)

### Rule 13: Conan as Package Manager
- **MANDATORY**: Use Conan for all C++ dependency management
- **MANDATORY**: All external dependencies declared in `conanfile.txt` or `conanfile.py`
- **NEVER**: Manually download or commit dependencies
- **MANDATORY**: CI/CD runs `conan install` before build
- **Why**: Reproducible builds, version control, cross-platform compatibility
- **Enforcement**: Build system validation, CI/CD gates

### Rule 14: CMake as Build System
- **MANDATORY**: CMakeLists.txt required at project root
- **MANDATORY**: All targets defined in CMake
- **NEVER**: Manual compilation commands in documentation
- **Why**: Standardized build, IDE integration, reproducibility
- **Enforcement**: Build system requirement

### Installation & Dependencies (Project Setup)

**Manual Installation (One-time):**
```bash
# Install Conan package manager
pip install conan

# Install system build tools
sudo apt-get install build-essential cmake g++ libssl-dev libpq-dev libgtest-dev
```

**Project Setup (Per clone):**
```bash
# Install project dependencies via Conan
conan install . --output-folder=build --build=missing

# Build project
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make
```

**Why these dependencies:**
- `build-essential`, `cmake`, `g++` — C++ compilation toolchain
- `libssl-dev` — TLS/SSL support
- `libpq-dev` — PostgreSQL database client
- `libgtest-dev` — Google Test framework for unit tests

---

## Logging and Secrets (MANDATORY)

### Rule 15: NEVER Log Credentials or Tokens
- **MANDATORY**: No passwords, tokens, or API keys in logs
- **MANDATORY**: No sensitive data in error messages
- **Action**: Log operation name, context (IDs only)
- **Why**: Security breach prevention
- **Enforcement**: Code review (grep for secrets)

### Rule 16: Error Logging with Context
- **MANDATORY**: Log errors with sufficient context for debugging
- **MANDATORY**: Include error code, operation name, relevant IDs
- **NEVER**: Swallow errors silently
- **Why**: Enables incident investigation
- **Enforcement**: Integration testing, security review

---

## Code Review Checklist

Before merging any C++ code:

- ✅ All pointers are smart pointers (no raw `new`/`delete`)
- ✅ RAII applied to all resources
- ✅ Destructors `noexcept`
- ✅ Class names `PascalCase`, functions `camelCase`, variables `snake_case`
- ✅ Member variables end with `_`
- ✅ All error codes checked on async operations
- ✅ Proper synchronization for concurrent access
- ✅ Objects kept alive during async ops
- ✅ No implementation in headers
- ✅ All dependencies in conanfile.txt (no manual deps)
- ✅ CMakeLists.txt updated with new targets
- ✅ No credentials in logs
- ✅ Tests present and deterministic
- ✅ No compiler warnings
- ✅ Project builds via `conan install && cmake && make`

---

## Customization Notes

**For your project**, consider:
- [ ] Which async I/O library? (Boost.ASIO, event loops, thread pools)
- [ ] What logging framework? (spdlog, Boost.Log, custom)
- [ ] What testing framework? (Google Test, Catch2, etc.)
- [ ] Add project-specific patterns (design patterns used)
- [ ] Add module structure requirements
- [ ] Define build system conventions (CMake, Meson)
- [ ] Add dependencies policy (Conan, vcpkg, etc.)

---

**Template Last Updated:** 2026-07-17  
**Status:** Copy and customize for your project
