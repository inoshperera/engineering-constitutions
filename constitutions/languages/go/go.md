---
id: language.go
title: Go
category: language
version: 1.0.0
status: active
owners:
  - go-platform
applies_to:
  - go
depends_on:
  - universal.code-quality
  - universal.testing
  - universal.dependency-management
---

# Go

## Purpose

Provide Go-specific engineering guidance.

## Mandatory requirements

- Go modules MUST commit `go.mod` and `go.sum`.
- Errors MUST be handled explicitly.
- Context cancellation MUST be respected for request-scoped work.
- Public packages MUST document exported identifiers.
- Data races MUST be considered for concurrent code.

## Recommended practices

- Keep interfaces small and consumer-owned where practical.
- Prefer table-driven tests for behavior matrices.
- Run race detection for concurrency-sensitive changes.

## Verification

Reviewers check module files, error handling, context propagation, exported documentation, tests, and race-sensitive code paths.
