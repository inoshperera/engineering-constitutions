---
id: universal.api-design
title: API Design
category: universal
version: 1.0.0
status: active
owners:
  - platform-architecture
applies_to:
  - all
depends_on:
  - universal.engineering-principles
  - universal.security
---

# API Design

## Purpose

Define expectations for stable, understandable, and secure APIs.

## Mandatory requirements

- APIs MUST use consistent resource naming and documented versioning.
- Existing clients MUST remain compatible, including existing route paths, HTTP methods, request and response shapes, and status-code semantics, unless an approved migration exists.
- Error responses MUST be structured, stable, and actionable.
- Collection APIs MUST define pagination behavior.
- Mutating APIs SHOULD support idempotency where retries are expected.
- Protected APIs MUST enforce authentication and authorization.
- Public or cross-team APIs MUST maintain OpenAPI or equivalent documentation.
- Inputs MUST be validated and contracts MUST remain stable across releases.

## Recommended practices

- Use rate limiting for abuse-prone or resource-intensive APIs.
- Document deprecations with timelines and replacement paths.
- Prefer explicit request and response schemas.

## Verification

Reviewers inspect API documentation, compatibility notes, validation paths, error schemas, and authentication or authorization coverage.
