# Changelog — `armature-opentelemetry`

All notable changes to this crate will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this crate adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Earlier changes are recorded in the workspace [`CHANGELOG.md`](../CHANGELOG.md).

## [Unreleased]

### Changed

- Bumped `tokio` to `1.53` and dev-dependency `serial_test` to `4`; no API changes required in this crate (the `#[serial_test::serial]` attribute path is unaffected). `opentelemetry`, `opentelemetry_sdk`, `opentelemetry-otlp`, `opentelemetry-zipkin`, and `opentelemetry-semantic-conventions` remain aligned on the `0.32` line — verified with `cargo tree -d` that no duplicate `opentelemetry*` versions are pulled in.

### Fixed

- **Breaking:** span and metric attributes use stable semantic conventions (`http.request.method`, `url.path`, `http.response.status_code`); dashboards keyed on the retired pre-1.0 names need updating.
- `http.route` no longer carries the raw target. Every distinct URL — query string included — minted a new metric time series, which is unbounded cardinality on a low-cardinality attribute.

### Fixed

- `http.route` is the query-less path rather than the raw request target. OTel defines `http.route` as a low-cardinality route template, so every distinct URL was minting its own time series.

### Changed

- **Breaking (telemetry):** span and metric attributes use the stable OTel HTTP semantic conventions instead of the retired pre-1.0 names: `http.method` → `http.request.method`, `http.target` → `url.path`, `http.status_code` → `http.response.status_code`, `http.scheme` → `url.scheme`, `http.host` → `server.address`, `http.user_agent` → `user_agent.original`, `http.response_content_length` → `http.response.body.size`. Dashboards and alerts keyed on the old names must be updated.
- Span names use the query-less path, for the same cardinality reason.

### Changed — `0.2.0` → `0.2.1`

- Migrated onto `armature-core` `0.8`'s `Bytes`-backed request and response types. No behavior change beyond what that migration implies; see [`armature-core/CHANGELOG.md`](../armature-core/CHANGELOG.md).
- `http.method` and `http.target` are read through the request's new accessors; `http.target` now carries the query string, which it previously dropped.

## [0.4.0] - 2026-08-05

### Changed

- **Requires `armature-core` 0.9 (breaking).** The requirement moved `0.8` →
  `0.9`. `armature-core 0.9.0` itself moves `armature-h1` across a breaking
  0.x boundary; because `armature-core` types appear in this crate's own
  public API, the requirement change is breaking here too and the minor moves
  with it. Under Cargo's 0.x caret rules the 0.8 and 0.9 types are distinct
  and do not unify, so a consumer holding an `armature-core 0.8` type cannot
  pass it to this crate. Part of the `armature-core 0.9.0` release train; see
  `armature-core`'s CHANGELOG for the publish order.

## [0.3.1] - 2026-08-04

### Fixed

- Requirements on sibling armature crates name a minor instead of `0`. Under
  Cargo's 0.x rules `version = "0"` matches any release ever made, and edition
  2024 selects the MSRV-aware resolver, so a consumer declaring an older
  `rust-version` was handed the oldest version satisfying it — resolving
  `armature-core = "0"` on Rust 1.89 produced `armature-core 0.2.3` while an
  explicit `armature-core = "0.8"` elsewhere in the same graph pulled 0.8.2.
  Two copies of core, and a build failing on symbols the older one lacks. Each
  0.x minor in this family is a breaking change, so the requirement now names
  one. No API change.
