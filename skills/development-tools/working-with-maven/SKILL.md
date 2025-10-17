---
name: Working with Maven
description: Build, test, format, and regenerate code in multi-module Maven projects
when_to_use: when building Java projects, running tests, formatting code, regenerating database models, or working with Maven multi-module projects
version: 1.0.0
languages: java
dependencies: Maven 3.6+
---

# Working with Maven

## Overview

**Maven is a build automation and dependency management tool for Java projects.** This skill covers essential Maven operations for multi-module projects including building, testing, formatting, and code generation.

## When to Use

Use this skill when:
- Building Java projects with Maven
- Running unit or integration tests
- Formatting code before commits
- Regenerating database models (JOOQ, JPA, etc.)
- Working with multi-module Maven projects
- Fixing compilation errors
- Managing dependencies

## Core Operations

### Building

```bash
# Full clean build with tests
mvn clean install

# Build without running tests (fast build)
mvn clean install -DskipTests

# Build specific module and its dependencies
mvn clean install -pl <module-name> -am

# Package without installing to local repository
mvn clean package

# Force update of SNAPSHOT dependencies
mvn clean install -U
```

**Key flags:**
- `-DskipTests` - Skip test execution (but compile tests)
- `-Dmaven.test.skip=true` - Skip test compilation and execution
- `-pl <module>` - Build specific module (project list)
- `-am` - Also make dependencies (build required modules)
- `-U` - Force update of SNAPSHOT dependencies

### Testing

```bash
# Run all tests in project
mvn test

# Run tests in specific module (with dependencies)
mvn test -pl <module-name> -am

# Run specific test class
mvn test -Dtest=ClassName

# Run specific test method
mvn test -Dtest=ClassName#methodName

# Run specific test in a module (with dependencies)
mvn test -Dtest=ClassName -pl <module> -am -Dsurefire.failIfNoSpecifiedTests=false

# Run multiple test classes
mvn test -Dtest=Class1,Class2

# Run integration tests (surefire + failsafe)
mvn verify

# Run tests with pattern matching
mvn test -Dtest=*IntegrationTest
```

**Test selection patterns:**
- `ClassName` - Single class
- `ClassName#methodName` - Single method
- `*IntegrationTest` - Pattern matching
- `Class1,Class2` - Multiple classes

**Important for multi-module projects:**
- When testing a specific module, always use `-am` to build dependencies first. Dependencies may not be in your local Maven repository, so Maven needs to build them before running tests.
- When running specific tests with `-Dtest` in a module with `-pl`, add `-Dsurefire.failIfNoSpecifiedTests=false` to prevent failures when the test doesn't exist in dependency modules.

### Code Formatting

```bash
# Format all code (fmt-maven-plugin or spotless)
mvn fmt:format

# Check formatting without changes
mvn fmt:check

# Format specific module
mvn fmt:format -pl <module-name>

# Format as part of clean build
mvn clean fmt:format install
```

**Common formatting plugins:**
- `fmt-maven-plugin` - Google Java Format
- `spotless-maven-plugin` - Multi-language formatting
- `maven-checkstyle-plugin` - Style checking

### Code Generation

```bash
# Regenerate JOOQ database models
mvn clean compile -Pgenerate-model -pl <repositorymodel-module>

# Alternative: verify phase for JOOQ generation
mvn clean verify -pl <repositorymodel-module> -P generate-model -am

# General code generation (from root)
mvn generate-sources

# Regenerate and build dependent modules
mvn clean install -pl <repositorymodel-module> -am
```

**When to regenerate:**
- After database schema changes (Flyway migrations)
- After updating OpenAPI specs
- After modifying code generation configuration
- When generated code is missing or outdated

## Multi-Module Project Patterns

### Module Selection

```bash
# Build single module only (no dependencies)
mvn clean install -pl <module>

# Build module and its dependencies
mvn clean install -pl <module> -am

# Build module and modules that depend on it
mvn clean install -pl <module> -amd

# Build multiple modules
mvn clean install -pl module1,module2 -am
```

### Working with Module Hierarchy

**Typical structure:**
```
parent-pom.xml
├── domain-model/
├── repository-model/
├── common/
├── api/
└── engine/
```

**Build order matters:**
1. Domain models (no dependencies)
2. Repository models (depends on domain)
3. Common services (depends on domain + repository)
4. API/Engine (depends on all)

**Maven reactor calculates build order automatically when using `-am`**

## Quick Reference

| Task | Command |
|------|---------|
| Full build | `mvn clean install` |
| Fast build | `mvn clean install -DskipTests` |
| Module build | `mvn clean install -pl <module> -am` |
| Run all tests | `mvn test` |
| Module tests | `mvn test -pl <module> -am` |
| Specific test | `mvn test -Dtest=ClassName#method` |
| Specific test in module | `mvn test -Dtest=ClassName -pl <module> -am -Dsurefire.failIfNoSpecifiedTests=false` |
| Format code | `mvn fmt:format` |
| Regenerate JOOQ | `mvn clean compile -Pgenerate-model -pl <repo-module>` |
| Update snapshots | `mvn clean install -U` |
| Integration tests | `mvn verify` |

## Common Workflows

### After Code Changes

```bash
# 1. Format code
mvn fmt:format

# 2. Build and test
mvn clean install

# 3. If tests fail, run specific test
mvn test -Dtest=FailingTest -pl <module> -am -Dsurefire.failIfNoSpecifiedTests=false
```

### After Database Schema Changes

```bash
# 1. Ensure migration files are in place
# (e.g., src/main/resources/db/migration/V*__.sql)

# 2. Regenerate JOOQ models
mvn clean compile -Pgenerate-model -pl <repository-module>

# 3. Build dependent modules
mvn clean install -pl <repository-module> -amd

# 4. Run tests to verify
mvn test
```

### Before Committing

```bash
# 1. Format all code
mvn clean fmt:format

# 2. Build and test everything
mvn clean install

# 3. If working with generated code, ensure it's up to date
mvn generate-sources
```

## Common Mistakes

### ❌ Running tests without building first
**Problem:** Stale compiled classes cause false failures
```bash
mvn test  # May use old classes
```
**Fix:** Clean before testing
```bash
mvn clean test
```

### ❌ Building module without dependencies
**Problem:** Module fails because dependencies not built
```bash
mvn clean install -pl api  # Fails if dependencies changed
```
**Fix:** Include dependencies with `-am`
```bash
mvn clean install -pl api -am
```

### ❌ Forgetting to regenerate after schema changes
**Problem:** JOOQ classes don't match database schema
```bash
# Add migration V002__add_column.sql
mvn clean install  # JOOQ still has old schema
```
**Fix:** Regenerate with profile first
```bash
mvn clean compile -Pgenerate-model -pl repository-module
mvn clean install
```

### ❌ Not using -U for SNAPSHOT dependencies
**Problem:** Local cache has stale SNAPSHOT versions
```bash
mvn clean install  # Uses cached SNAPSHOT
```
**Fix:** Force update with -U
```bash
mvn clean install -U
```

### ❌ Running tests from wrong directory
**Problem:** Maven can't find pom.xml or wrong module executes
```bash
cd src/
mvn test  # Error: Cannot find pom.xml
```
**Fix:** Run from project or module root
```bash
cd /path/to/project
mvn test -pl api -am
```

### ❌ Testing module without building dependencies
**Problem:** Module tests fail because dependencies not in local repo
```bash
mvn test -pl api  # Fails if dependencies not installed
```
**Fix:** Include dependencies with `-am`
```bash
mvn test -pl api -am
```

### ❌ Running specific test in module without failIfNoSpecifiedTests flag
**Problem:** Maven fails when test doesn't exist in dependency modules
```bash
mvn test -Dtest=ApiTest -pl api -am
# [ERROR] No tests were executed! (Expected: 1)
```
**Fix:** Add `-Dsurefire.failIfNoSpecifiedTests=false`
```bash
mvn test -Dtest=ApiTest -pl api -am -Dsurefire.failIfNoSpecifiedTests=false
```

## Profile Usage

Profiles activate specific build configurations:

```bash
# Activate single profile
mvn clean install -P<profile-name>

# Activate multiple profiles
mvn clean install -P<profile1>,<profile2>

# Common profile examples
mvn clean install -Pproduction        # Production build
mvn clean compile -Pgenerate-model    # Code generation
mvn test -Pintegration-tests          # Integration tests
```

**Check available profiles:**
```bash
mvn help:all-profiles
```

## Debugging Maven

```bash
# Show debug output
mvn clean install -X

# Show dependency tree
mvn dependency:tree

# Show effective POM (with inheritance)
mvn help:effective-pom

# Show active profiles
mvn help:active-profiles

# Analyze dependency conflicts
mvn dependency:analyze
```

## Performance Tips

**Parallel builds:**
```bash
# Use multiple threads
mvn clean install -T 4

# One thread per CPU core
mvn clean install -T 1C
```

**Skip unnecessary steps:**
```bash
# Skip tests + javadoc + source jars
mvn clean install -DskipTests -Dmaven.javadoc.skip=true -Dmaven.source.skip=true

# Skip only during local development
```

**Offline mode (use local cache):**
```bash
mvn clean install -o
```

## Red Flags

Watch for these signs you're using Maven incorrectly:

- **Building same module repeatedly** → Use `-pl` to target specific module
- **Tests failing randomly** → Run `mvn clean test` to ensure clean state
- **"Class not found" after changes** → Run `mvn clean install` not just `mvn install`
- **JOOQ errors after schema change** → Regenerate models with profile
- **SNAPSHOT dependency issues** → Use `-U` to force updates
- **Building from src/ or target/ directory** → Navigate to project root

## Real-World Impact

**Without this skill:**
- 5-10 minutes searching for correct Maven command
- Multiple failed builds trying wrong flags
- Stale generated code causing mysterious errors
- Unnecessary full builds (2-5 minutes) when module build (30s) would work

**With this skill:**
- Immediate correct command for any Maven task
- Efficient module-targeted builds
- Proper code generation workflow
- Understanding of when to use which approach

## Related Skills

- `skills/testing/test-driven-development` - Writing tests before implementation
- `skills/debugging/systematic-debugging` - Debugging test failures
- `skills/collaboration/finishing-a-development-branch` - Pre-commit workflows
