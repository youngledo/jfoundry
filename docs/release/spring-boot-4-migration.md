# Spring Boot 4 Migration

This document records the 1.x to 2.x baseline changes.

## Baseline

| Area | 1.x | 2.x |
|------|-----|-----|
| Java compile target | 21 | 25 |
| Runtime Java | 21 | 25 |
| Spring Boot | 3.5.x | 4.1.x |
| Spring Framework | 6.2.x | 7.0.x |
| MyBatis-Plus starter | `mybatis-plus-spring-boot3-starter` | `mybatis-plus-spring-boot4-starter` |
| JobRunr starter | `jobrunr-spring-boot-3-starter` | `jobrunr-spring-boot-4-starter` |
| Maven release tool | Maven wrapper 3.9.x | Maven wrapper 3.9.16 |
| Maven 4 | compatibility check | compatibility check |

## Application Impact

- Move applications to Java 25 before adopting jfoundry 2.x.
- Use Spring Boot 4.1.x dependency management in applications.
- Replace direct MyBatis-Plus Boot 3 starter usage with the Boot 4 starter.
- Replace direct JobRunr Boot 3 starter usage with the Boot 4 starter.
- Keep using jfoundry starter artifact names; the public jfoundry starter names did not change for the Boot 4 line.

## Jackson Compatibility

jfoundry 2.x still provides the existing Jackson 2 payload serializer module
(`jfoundry-messaging-jackson`) because `jmolecules-jackson:0.33.0` is a Jackson 2 integration.
Spring Boot 4.1's own Jackson stack defaults to Jackson 3 (`tools.jackson` packages), so
applications that use jfoundry's default payload serializer must provide a Jackson 2
`com.fasterxml.jackson.databind.ObjectMapper` bean or keep the jfoundry messaging starter's
Jackson 2 dependencies on the classpath.

This is an intentional compatibility fallback for 2.0.0-SNAPSHOT. A future 2.x milestone can add
a Jackson 3 serializer once upstream jMolecules integration and payload compatibility are verified.

## Release Notes

- The 2.x release workflow publishes to Maven Central when a GitHub Release is published.
- After successful Central deployment, the workflow bumps the release branch to the next
  `*-SNAPSHOT` version.
- Maven Central credentials and signing keys must be configured in the GitHub environment
  `maven-central`; they must not be committed to the repository.
- Do not use private Maven profiles for public release verification.
  Local Central-only verification can use `.mvn/settings-central.xml`.
