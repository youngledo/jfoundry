# Maven Central Release

This project has Maven Central publishing infrastructure for the public `youngledo/jfoundry` repository.

## Metadata

The root POM publishes URL and SCM metadata for `https://github.com/youngledo/jfoundry`. Verify that the published groupId is authorized in Sonatype Central Portal before the first release.

## Prerequisites

- Java 25 or newer for the 2.x line.
- Maven 3.9.0 or newer.
- A Sonatype Central Portal account with publishing rights for `org.jfoundry`.
- For local release dry-runs, a Maven server entry named `central` in a dedicated untracked
  settings file such as `~/.m2/settings-central-release.xml`.
- For GitHub Actions release publishing, repository environment `maven-central` with these secrets:
  - `CENTRAL_USERNAME`: Sonatype Central Portal username or publishing token username.
  - `CENTRAL_PASSWORD`: Sonatype Central Portal password or publishing token password.
  - `GPG_PRIVATE_KEY`: ASCII-armored private key used to sign artifacts.
  - `GPG_PASSPHRASE`: passphrase for the private key.
- A GPG key available to Maven for local artifact signing.
- Release versions in all reactor POMs. Central releases must not publish `*-SNAPSHOT` versions.
- Real project URL and SCM metadata in `pom.xml`.

Do not commit Maven Central usernames, tokens, passwords, or GPG private keys. Keep them in
local environment variables, a dedicated untracked release settings file, or GitHub Actions secrets.
Do not use any private Maven profile for public release verification or publishing.
This repository includes `.mvn/settings-central.xml` for Central-only dependency resolution
during local verification; it intentionally contains no credentials.

For local publishing, create a dedicated untracked settings file such as
`~/.m2/settings-central-release.xml` and pass it with `-s`. Do not reuse a settings file that
activates private mirrors or repositories.

Example server configuration:

```xml
<servers>
  <server>
    <id>central</id>
    <username>${env.CENTRAL_USERNAME}</username>
    <password>${env.CENTRAL_PASSWORD}</password>
  </server>
</servers>
```

## GitHub Secrets To Add Later

When the Central Portal namespace is ready, add these secrets to the repository environment
named `maven-central`:

| Secret | Value |
|--------|-------|
| `CENTRAL_USERNAME` | Sonatype Central Portal username or publishing token username |
| `CENTRAL_PASSWORD` | Sonatype Central Portal password or publishing token password |
| `GPG_PRIVATE_KEY` | ASCII-armored private key used to sign artifacts |
| `GPG_PASSPHRASE` | Passphrase for `GPG_PRIVATE_KEY` |

GitHub path: repository `Settings` -> `Environments` -> `maven-central` -> `Environment secrets`.
Add protection reviewers to the environment if release publication should require manual approval.

## Local Verification

Run the regular package build first:

```bash
./mvnw -s .mvn/settings-central.xml -DskipTests package
```

Then run the release profile through `verify` so sources, Javadocs, and local signatures are exercised:

```bash
./mvnw -s .mvn/settings-central.xml -Prelease -DskipTests verify
```

The `verify` phase checks local artifact generation and signatures up to the GPG signing step. It does not upload or stage a Central Portal deployment bundle; that behavior is triggered during `deploy`.

If GPG is not configured locally, the release-profile verification may fail at the signing step. Failures before signing, including compilation, source JARs, Javadocs, metadata, placeholder metadata guards, or Central publishing plugin setup, must be fixed before release.

## Publish

After replacing placeholder metadata, setting release versions, and configuring Central credentials and GPG, deploy locally with the release profile:

```bash
./mvnw -s ~/.m2/settings-central-release.xml -Prelease -DskipTests deploy
```

The Central publishing plugin is configured with `autoPublish=false`, so the deployment uploads a staged deployment to Central Portal for manual review and publishing.

## GitHub Release Publishing

The `Release` workflow publishes automatically when a GitHub Release is published.

Use a tag such as `v2.0.0`. The workflow strips the leading `v`, sets all reactor POM versions
to `2.0.0`, runs `./mvnw -B -Prelease -DskipTests deploy`, then updates the release branch to
the next development version in a separate job and Git worktree. For a GitHub Release, the release
branch defaults to the release target branch; for a manual `workflow_dispatch` run it defaults to
the selected branch and can be overridden with `release_branch`.

The publish and branch bump steps are intentionally separate jobs. If the Central staged
deployment succeeds but the branch push is rejected by branch protection or a non-fast-forward
race, rerun only the failed bump job; do not rerun the publish job for the same release version.

By default, the next development version is inferred as the next patch version, for example
`2.0.0` becomes `2.0.1-SNAPSHOT`. To override it, add a line to the GitHub Release notes:

```text
Next-Snapshot: 2.1.0-SNAPSHOT
```

Manual workflow runs require `release_version` and can optionally set `next_snapshot_version`
and `release_branch`.

The publish job uses the Central publishing plugin with `autoPublish=false`, so a successful
GitHub Release upload creates a staged Central Portal deployment for manual review and publishing.
