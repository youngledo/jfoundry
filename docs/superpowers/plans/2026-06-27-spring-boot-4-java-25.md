# Spring Boot 4 Java 25 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a future 2.x line that supports Spring Boot 4.x, Spring Framework 7.x, and Java 25 without destabilizing the 1.x Spring Boot 3.5 production release.

**Architecture:** Keep `1.x` on Java 21/Spring Boot 3.5 for the first Maven Central release. Build the Boot 4/Java 25 work on a dedicated branch that updates dependency baselines, adapter APIs, CI matrices, and migration docs together. Maven 4 remains a compatibility gate until Apache Maven publishes a GA release and this project has clean full-reactor evidence with it.

**Tech Stack:** Java 25, Maven Wrapper 3.9.x, Maven 4 preview compatibility job, Spring Boot 4.x, Spring Framework 7.x, Jakarta EE 11, MyBatis-Plus Boot 4-compatible starter when available, JobRunr Boot 4-compatible starter when available, JUnit Jupiter, Failsafe, Testcontainers.

---

## Current External Baseline

- Spring Boot 4.1.0 is available in Maven Central, and Spring Boot 4.0.7 remains available as a stable 4.0 maintenance release.
- Spring Boot 4.1.0 requires at least Java 17 and documents compatibility up to Java 26.
- JDK 25 reached General Availability on 2025-09-16.
- Maven 4 remains preview at `4.0.0-rc-5`; use Maven 3.9.x as the release tool until Maven 4 GA.

## File Structure

- Modify `pom.xml`: raise `maven.compiler.release`, enforcer Java range, CI-facing plugin versions, and any Maven 4 model fixes required by the 2.x line.
- Modify `jfoundry-dependencies/pom.xml`: inherit the parent dependency management and publish the 2.x BOM.
- Modify Spring modules under `jfoundry-spring/`: move starters and auto-configuration to Spring Boot 4 package/module expectations.
- Modify infrastructure modules under `jfoundry-infrastructure/`: update MyBatis-Plus, JobRunr, Kafka, RabbitMQ, RocketMQ, and Jackson integrations against the Boot 4 ecosystem.
- Modify `.github/workflows/ci.yml`: make Java 25 the primary test target for 2.x and keep a Maven 4 compatibility job.
- Create or modify `docs/release/compatibility.md`: document the 2.x support matrix and migration status.
- Create `docs/release/spring-boot-4-migration.md`: user-facing migration notes from 1.x to 2.x.

## Task 1: Create The 2.x Compatibility Branch

**Files:**
- Modify: `pom.xml`
- Modify: `.github/workflows/ci.yml`
- Modify: `docs/release/compatibility.md`

- [ ] **Step 1: Create a branch from the latest 1.x release branch**

Run:

```bash
git checkout -b spring-boot-4-java-25
```

Expected: a clean branch based on the released 1.x baseline.

- [ ] **Step 2: Set the development version**

Run:

```bash
./mvnw -B org.codehaus.mojo:versions-maven-plugin:2.21.0:set \
  -DnewVersion=2.0.0-SNAPSHOT \
  -DprocessAllModules=true \
  -DgenerateBackupPoms=false
```

Expected: every reactor POM moves to `2.0.0-SNAPSHOT`.

- [ ] **Step 3: Raise the compiler release**

In `pom.xml`, change:

```xml
<maven.compiler.release>21</maven.compiler.release>
```

to:

```xml
<maven.compiler.release>25</maven.compiler.release>
```

Change the enforcer Java range from:

```xml
<version>[21,)</version>
```

to:

```xml
<version>[25,)</version>
```

- [ ] **Step 4: Update the CI Java matrix**

In `.github/workflows/ci.yml`, make Java 25 the primary required matrix entry. Keep Java 21 out of the 2.x required matrix unless a specific 2.x module intentionally remains Java 21-compatible.

- [ ] **Step 5: Verify Java 25 toolchain availability**

Run:

```bash
java -version
./mvnw -version
```

Expected: Java `25.x` and Maven wrapper `3.9.16` or newer.

- [ ] **Step 6: Commit the branch baseline**

```bash
git add pom.xml .github/workflows/ci.yml docs/release/compatibility.md '**/pom.xml'
git commit -m "build: start spring boot 4 java 25 line"
```

## Task 2: Upgrade Spring Boot And Spring Framework

**Files:**
- Modify: `pom.xml`
- Modify: `jfoundry-dependencies/pom.xml`
- Modify: Spring module POMs under `jfoundry-spring/`

- [ ] **Step 1: Select the Boot 4 baseline**

Use Spring Boot `4.1.x` unless a release blocker requires staying on `4.0.x`. Record the exact selected version in `docs/release/compatibility.md`.

- [ ] **Step 2: Update managed versions**

In `pom.xml`, update the managed properties. Prefer Spring Boot dependency management for
Spring Framework artifacts; keep an explicit `<spring.version>` only if this project still
has a local reason to override Boot's managed Spring Framework version.

```xml
<spring-boot.version>4.1.0</spring-boot.version>
```

If newer compatible patch versions are available when executing the plan, use the latest patch
release and record the exact values. If `<spring.version>` remains explicit, set it to the
Spring Framework 7.x version aligned with the selected Boot BOM.

- [ ] **Step 3: Run dependency resolution**

Run:

```bash
./mvnw -U -DskipTests validate
```

Expected: unresolved artifacts expose every module that still depends on Boot 3-only coordinates.

- [ ] **Step 4: Fix Boot 4 coordinate changes**

Update Boot starter, auto-configuration, and metadata dependencies only where Maven resolution or compilation proves a change is required. Do not rename public jfoundry artifacts unless the artifact is explicitly Boot-major-specific.

- [ ] **Step 5: Verify package compilation**

Run:

```bash
./mvnw -DskipTests package
```

Expected: compilation succeeds or produces a focused list of Boot 4 API changes for Task 3.

## Task 3: Update Optional Infrastructure Adapters

**Files:**
- Modify: `pom.xml`
- Modify: `jfoundry-infrastructure/**/pom.xml`
- Modify: `jfoundry-spring/**/pom.xml`

- [ ] **Step 1: Check MyBatis-Plus Boot 4 support**

Search Maven Central for a Spring Boot 4-compatible MyBatis-Plus starter. If no Boot 4 starter is available, keep MyBatis-Plus modules marked as not yet supported on the 2.x line and document the blocker.

- [ ] **Step 2: Check JobRunr Boot 4 support**

Search Maven Central for a JobRunr Spring Boot 4 starter. If unavailable, keep `jfoundry-outbox-jobrunr` optional and document the blocker.

- [ ] **Step 3: Update messaging adapters**

Validate `spring-kafka`, `spring-amqp`, RocketMQ, and Jackson versions against Boot 4 dependency management. Prefer Boot-managed versions unless a module needs an explicit override.

- [ ] **Step 4: Run focused adapter tests**

Run:

```bash
./mvnw -pl jfoundry-infrastructure -am test
./mvnw -pl jfoundry-spring -am test
```

Expected: supported adapters compile and test; unsupported adapters have explicit documentation and are excluded or marked optional.

## Task 4: Run Full Java 25 And Maven 4 Verification

**Files:**
- Modify: `.github/workflows/ci.yml`
- Modify: `docs/release/compatibility.md`

- [ ] **Step 1: Run the full unit suite on Java 25**

Run:

```bash
./mvnw test
```

Expected: PASS on Java 25.

- [ ] **Step 2: Run the full package gate**

Run:

```bash
./mvnw -DskipTests package
```

Expected: PASS on Java 25.

- [ ] **Step 3: Run integration tests**

Run:

```bash
./mvnw -pl jfoundry-integration-tests -am -Pit verify
```

Expected: PASS for every adapter advertised as supported in 2.x.

- [ ] **Step 4: Run Maven 4 compatibility**

Run with the latest Maven 4 preview or GA available at execution time:

```bash
mvn -B -DskipTests package
```

Expected: PASS with no model warnings from jfoundry POMs.

- [ ] **Step 5: Commit verification docs**

```bash
git add .github/workflows/ci.yml docs/release/compatibility.md
git commit -m "docs: record spring boot 4 java 25 verification"
```

## Task 5: Publish Migration Notes

**Files:**
- Create: `docs/release/spring-boot-4-migration.md`
- Modify: `README.md`

- [ ] **Step 1: Document the migration boundary**

Create `docs/release/spring-boot-4-migration.md` with:

```markdown
# Spring Boot 4 Migration

The Spring Boot 4 / Java 25 baseline is the jfoundry 2.x line.

## Requirements

- Java 25 or newer.
- Spring Boot 4.x.
- Spring Framework 7.x.
- Maven 3.9.x for releases. Maven 4 is supported as a compatibility target after clean CI evidence.

## Compatibility

The 1.x line remains on Java 21 and Spring Boot 3.5.x.

## Known Adapter Status

| Adapter | 2.x Status |
|---------|------------|
| Spring auto-configuration | Supported after full CI verification |
| MyBatis-Plus | Supported only after a Boot 4-compatible starter is verified |
| JobRunr | Supported only after a Boot 4-compatible starter is verified |
| Kafka | Supported after full CI verification |
| RabbitMQ | Supported after full CI verification |
| RocketMQ | Supported after full CI verification |
```

- [ ] **Step 2: Update README support table**

Add separate rows for `1.x` and `2.x` so users do not assume Boot 4 support exists in the first Central release.

- [ ] **Step 3: Commit migration docs**

```bash
git add README.md docs/release/spring-boot-4-migration.md
git commit -m "docs: add spring boot 4 migration guide"
```
