# KubeOrch Release Process

This document describes how KubeOrch releases are planned, built, and published.

## Versioning

KubeOrch follows [Semantic Versioning 2.0.0](https://semver.org/):

- **MAJOR** (X.0.0): Breaking API changes or incompatible architectural changes
- **MINOR** (0.X.0): New features, non-breaking additions
- **PATCH** (0.0.X): Bug fixes, security patches, documentation corrections

All repositories in the KubeOrch organization share the same version number to ensure compatibility.

### Pre-1.0

While KubeOrch is pre-1.0, minor version bumps may include breaking changes. These will always be documented in the changelog.

## Release Cadence

| Type | Cadence | Description |
|------|---------|-------------|
| Minor release | Monthly | New features and improvements |
| Patch release | As needed | Bug fixes and security patches |
| Major release | As needed | Breaking changes (announced at least one minor release in advance) |

## Release Checklist

Before tagging a release, the release manager must verify:

### 1. Pre-Release

- [ ] All CI checks pass on the main branch
- [ ] No open critical or high-priority bugs
- [ ] CHANGELOG.md is updated with all changes since last release
- [ ] Documentation is updated to reflect new features or changes
- [ ] All dependency updates are reviewed (Dependabot PRs merged or dismissed)
- [ ] Security scan results reviewed (Trivy, gosec)

### 2. Release

- [ ] Create a release branch: `release/vX.Y.Z`
- [ ] Update version numbers in all repositories (go.mod, package.json, etc.)
- [ ] Tag the release: `git tag -s vX.Y.Z -m "Release vX.Y.Z"`
- [ ] Push the tag: `git push origin vX.Y.Z`
- [ ] CI automation builds and publishes:
  - **Core**: Docker image to container registry
  - **UI**: Docker image to container registry
  - **CLI**: Cross-platform binaries to GitHub Releases + npm package

### 3. Post-Release

- [ ] Verify all artifacts are published (Docker images, npm package, GitHub Release)
- [ ] Verify installation methods work (curl script, npm install, go install)
- [ ] Create GitHub Release with release notes
- [ ] Announce release in community channels (Slack, mailing list, discussions)
- [ ] Update docs site if needed

## Release Roles

| Role | Responsibility |
|------|---------------|
| Release Manager | A Maintainer who drives the release for a given version |
| Maintainers | Review and approve the release PR |
| CI/CD | Automated build, test, and publish pipeline |

The Release Manager role rotates among Maintainers.

## Support Policy

| Version | Support Level |
|---------|--------------|
| Current minor release | Full support (features, bugs, security) |
| Previous minor release | Security and critical bug fixes only |
| Older releases | No active support |

Users are encouraged to stay on the latest minor release.

## Hotfix Process

For critical security vulnerabilities or severe bugs in the current release:

1. Create a hotfix branch from the release tag: `hotfix/vX.Y.Z+1`
2. Apply the minimal fix
3. Follow the standard release checklist for a patch release
4. Cherry-pick the fix to the main branch

## Changelog

Each repository maintains a `CHANGELOG.md` following the [Keep a Changelog](https://keepachangelog.com/) format. Changes are categorized as:

- **Added**: New features
- **Changed**: Changes to existing functionality
- **Deprecated**: Features that will be removed in a future release
- **Removed**: Features that have been removed
- **Fixed**: Bug fixes
- **Security**: Security-related changes
