---
id: platform.kubernetes
title: Kubernetes
category: platform
version: 1.0.0
status: active
owners:
  - platform-operations
applies_to:
  - kubernetes
depends_on:
  - platform.docker
  - universal.observability
---

# Kubernetes

## Purpose

Provide Kubernetes deployment guidance for reliable services.

## Mandatory requirements

- Workloads MUST define resource requests and appropriate limits.
- Readiness and liveness probes MUST match service behavior.
- Secrets MUST be provided through approved secret-management mechanisms.
- Deployments MUST define rollout behavior and failure expectations.
- Network exposure MUST be intentional and reviewed.

## Recommended practices

- Use namespaces, labels, and annotations consistently.
- Prefer declarative manifests managed through review.
- Validate changes against target cluster policy before release.

## Verification

Reviewers check resources, probes, secret references, rollout settings, service exposure, and policy validation output.
