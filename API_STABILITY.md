# KubeOrch API Stability Policy

This document defines how KubeOrch manages API versioning, stability guarantees, and deprecation of features.

## API Versioning

KubeOrch's REST API uses URL-based versioning with a `/vN/` prefix:

- `/v1/api/...` — Current stable API
- `/v2/api/...` — Next major version (when applicable)

Only one major API version is actively developed at a time. The previous major version enters maintenance mode when a new one is released.

## Stability Levels

Every API endpoint and feature in KubeOrch has a stability level:

### Alpha

- **Prefix**: None (experimental endpoints may be under `/v1/api/experimental/`)
- **Guarantees**: No stability guarantees. May change or be removed without notice.
- **Use case**: Early prototyping and feedback gathering.
- **Enabled by default**: No (may require feature flag or configuration).

### Beta

- **Guarantees**: The feature is considered functional and the API shape is unlikely to change significantly. Breaking changes will be announced at least one minor release in advance.
- **Use case**: Safe to use in non-critical environments. Feedback encouraged.
- **Enabled by default**: Yes.

### Stable

- **Guarantees**: The API will not have breaking changes within the same major version. Any breaking changes require a new major API version.
- **Use case**: Safe for production use.
- **Enabled by default**: Yes.

## What Constitutes a Breaking Change

The following are considered breaking changes:

- Removing an API endpoint
- Removing or renaming a request or response field
- Changing the type of an existing field
- Changing the HTTP method for an endpoint
- Changing authentication or authorization requirements for an existing endpoint
- Changing the default behavior of an endpoint in a way that breaks existing clients

The following are **not** breaking changes:

- Adding a new API endpoint
- Adding a new optional request field
- Adding a new field to a response
- Adding a new optional query parameter
- Fixing a bug where behavior did not match documentation
- Performance improvements

## Deprecation Policy

When a Stable or Beta feature is scheduled for removal or replacement:

1. **Announcement**: The deprecation is announced in the changelog and release notes.
2. **Minimum notice**: Deprecated features remain functional for at least **2 minor releases** after the deprecation announcement.
3. **Documentation**: Deprecated endpoints are marked in the API reference with the recommended replacement.
4. **Response headers**: Deprecated endpoints include a `Deprecation` header in their response.
5. **Removal**: After the notice period, the feature may be removed in the next minor or major release.

### Timeline Example

| Version | Action |
|---------|--------|
| v0.5.0 | Feature X deprecated; replacement Feature Y introduced |
| v0.6.0 | Feature X still functional, deprecation warnings active |
| v0.7.0 | Feature X may be removed |

## Configuration and CLI Flags

Configuration options and CLI flags follow the same stability levels and deprecation policy as API endpoints. Deprecated configuration options will log a warning at startup with guidance on the replacement.

## Client SDK Compatibility

When official client SDKs are available, they will follow the same versioning as the API:

- SDK major version matches API major version
- SDK minor versions may introduce new features but not break existing usage
- Deprecated SDK methods will emit warnings before removal

## Exceptions

Security vulnerabilities may require immediate breaking changes without the standard deprecation period. In such cases:

- A security advisory will be published
- The changelog will document the breaking change and the reason
- Migration guidance will be provided

## Current API Status

| API Version | Stability | Status |
|-------------|-----------|--------|
| `/v1/api/` | Beta | Active development (pre-1.0) |

The API will be promoted to Stable when KubeOrch reaches version 1.0.
