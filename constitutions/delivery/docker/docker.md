---
id: platform.docker
title: Docker
category: platform
version: 1.0.0
status: active
owners:
  - platform-operations
applies_to:
  - docker
depends_on:
  - universal.security
  - universal.dependency-management
---

# Docker

## Purpose

Provide container image guidance for reproducible and secure deployments.

## Mandatory requirements

- Images MUST use maintained base images.
- Dockerfiles MUST avoid embedding secrets.
- Runtime containers SHOULD run as non-root where feasible.
- Images MUST minimize unnecessary packages and build artifacts.
- Exposed ports and entrypoints MUST be intentional and documented.

## Recommended practices

- Use multi-stage builds for compiled artifacts.
- Pin base images by version and review digest pinning for critical services.
- Scan images for vulnerabilities in CI.

## Verification

Reviewers check base images, secret handling, runtime user, image size, exposed ports, and vulnerability scan results.
