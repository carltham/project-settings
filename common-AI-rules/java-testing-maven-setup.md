# Java Testing with Maven - Setup Guide

Configuration for unit tests, integration tests, and code coverage reporting in Maven projects.

---

## Maven Plugins Required

This guide configures three Maven plugins to work together:
1. **Surefire** — Runs unit tests (`mvn test`)
2. **Failsafe** — Runs integration tests (in profile)
3. **JaCoCo** — Generates code coverage reports

---

## Test Naming Convention

**Integration Tests**
- Suffix: `IT` exactly (e.g., `CashierControllerIT`, `UserServiceIT`)
- Run via Failsafe plugin in dedicated profile
- Never use `ITTest` or `TestIT`

**Unit Tests**
- Suffix: `Test` or `Tests` (e.g., `CategoryValidatorTest`, `PasswordEncoderTest`)
- Must NOT use `IT` suffix
- Run via Surefire plugin in default build

---

## Unit Tests with Coverage (Default Build)

Add to `pom.xml` `<build><plugins>` section:

```xml
<!-- JaCoCo Maven Plugin - generates coverage reports -->
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.10</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>

<!-- Surefire Maven Plugin - runs unit tests (excludes *IT.java) -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.0.0-M9</version>
    <configuration>
        <excludes>
            <exclude>**/*IT.java</exclude>
        </excludes>
        <argLine>@{argLine}</argLine>
    </configuration>
</plugin>
```

**What This Does**:
- `prepare-agent` — Instruments bytecode for coverage measurement
- `report` — Generates coverage report in `target/site/jacoco/`
- `@{argLine}` — Late-binding placeholder (JaCoCo fills in the agent argument)
- `<exclude>**/*IT.java</exclude>` — Unit tests only, integration tests excluded

**Run Command**:
```bash
mvn test
```

**Output**: 
- Runs all `*Test.java` and `*Tests.java` files
- Produces coverage report at `target/site/jacoco/index.html`

---

## Integration Tests in Profile

Add to `pom.xml` `<profiles><profile>` section:

```xml
<profile>
    <id>integration-tests</id>
    <build>
        <plugins>
            <!-- JaCoCo for integration test coverage -->
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.8.10</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>prepare-agent-integration</goal>
                            <goal>report-integration</goal>
                        </goals>
                        <configuration>
                            <destFile>${project.build.directory}/jacoco-it.exec</destFile>
                            <dataFile>${project.build.directory}/jacoco-it.exec</dataFile>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            
            <!-- Failsafe Plugin - runs integration tests (*IT.java) -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <version>3.0.0-M9</version>
                <configuration>
                    <includes>
                        <include>**/*IT.java</include>
                    </includes>
                    <argLine>@{argLine}</argLine>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>integration-test</goal>
                            <goal>verify</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</profile>
```

**What This Does**:
- `prepare-agent-integration` — Instruments bytecode for integration test coverage
- `report-integration` — Generates separate coverage report from integration tests
- `destFile` — Stores results in separate file (`jacoco-it.exec`)
- Failsafe runs `*IT.java` files and verifies they pass

**Run Commands**:
```bash
# Run unit tests ONLY (default)
mvn test

# Run unit tests + integration tests
mvn test -P integration-tests

# Run integration tests only (verify = integration-test + verify)
mvn verify -P integration-tests

# Clean run with integration tests
mvn clean verify -P integration-tests
```

**Output**:
- Unit tests: `target/site/jacoco/index.html`
- Integration tests: `target/site/jacoco-it/index.html`

---

## Understanding @{argLine}

`@{argLine}` is a **late-binding placeholder** that JaCoCo fills in at runtime.

Without it:
```xml
<argLine>-Xmx1024m</argLine>  <!-- No JaCoCo agent added -->
```
Result: JaCoCo cannot measure coverage.

With it:
```xml
<argLine>@{argLine}</argLine>  <!-- JaCoCo inserts its agent -->
```
Result: JaCoCo instrument bytecode + `-Xmx1024m` still works.

JaCoCo expands it to something like:
```
-javaagent:/path/to/jacocoagent.jar=destfile=target/jacoco.exec
```

---

## Coverage Report Locations

After running tests, view coverage reports:

**Unit Test Coverage**:
```
target/site/jacoco/index.html
```

**Integration Test Coverage**:
```
target/site/jacoco-it/index.html
```

**Combined Coverage** (if needed in a separate build step):
```bash
# Merge both coverage files
mvn jacoco:merge -Djacoco.merge.destFile=target/jacoco-merged.exec
mvn jacoco:report -Djacoco.dataFile=target/jacoco-merged.exec
# Result: target/site/jacoco/index.html
```

---

## Troubleshooting

**Integration tests still run with `mvn test`**:
- Check `<exclude>**/*IT.java</exclude>` is in Surefire config
- Rebuild: `mvn clean test`

**JaCoCo agent not loading**:
- Check `@{argLine}` is in both Surefire and Failsafe `<argLine>`
- Verify jacoco-maven-plugin executions run before tests

**No coverage report generated**:
- Verify `<goal>report</goal>` is in execution
- Check `target/site/jacoco/` directory exists
- Rebuild: `mvn clean test`

**Different coverage between unit and integration tests**:
- This is normal — unit tests cover different code paths than integration tests
- Use separate files to compare: `jacoco.exec` vs `jacoco-it.exec`

---

## Best Practices

1. **Run unit tests frequently** — `mvn test` should be fast (< 30 seconds)
2. **Integration tests in CI/CD** — Include profile in build pipeline
3. **Monitor coverage trends** — Track `jacoco/index.html` over time
4. **Don't aim for 100%** — 80% unit + 60% integration = good baseline
5. **Separate coverage files** — Easier to debug coverage differences

---

**Last Updated:** 2026-07-12
