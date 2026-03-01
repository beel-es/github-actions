# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-03-01

### Added

#### Composite Actions
- `openapi-sync` - Download and validate OpenAPI specs from remote sources
- `openapi-bundle` - Bundle multiple OpenAPI YAML files using Redocly CLI

#### Reusable Workflows
- `sync-from-openapi.yml` - Complete workflow to sync OpenAPI spec from URL
- `bundle-openapi.yml` - Complete workflow to bundle OpenAPI files
- `publish-npm-sdk.yml` - Complete workflow to regenerate, bump, and publish npm packages

#### Features
- SHA256 hash verification for downloaded specs
- Change detection (skip commits if no changes)
- Automatic version bumping (patch/minor/major)
- GitHub Release creation on npm publish
- Generic and reusable across any OpenAPI project

### Documentation
- Comprehensive README with examples
- Real-world usage patterns
- Security best practices
- MIT License

[1.0.0]: https://github.com/beel-es/github-actions/releases/tag/v1.0.0
