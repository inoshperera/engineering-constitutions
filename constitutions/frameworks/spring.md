---
id: framework.spring
title: Spring
category: framework
version: 1.0.0
status: active
owners:
  - java-platform
applies_to:
  - spring
depends_on:
  - language.java
  - universal.api-design
  - universal.observability
---

# Spring

## Purpose

Provide Spring-specific guidance for Java services.

## Mandatory requirements

- Configuration properties MUST be typed and documented.
- Controllers MUST keep authorization and validation behavior explicit.
- Controllers MUST preserve existing route paths, HTTP methods, request and response shapes, and status-code semantics unless an approved API migration exists.
- Dependency injection MUST avoid hidden global state.
- Transaction boundaries MUST be clear for data-changing operations.
- Service-to-service clients MUST use framework-managed clients with configurable endpoints, timeouts, authentication behavior, and observability hooks.
- Actuator or equivalent health endpoints MUST be configured safely.
- Scheduled jobs MUST make cadence configurable and MUST make timezone behavior explicit when business dates or local operating windows are involved.

## Recommended practices

- Keep business logic outside controller classes.
- Prefer constructor injection.
- Keep request validation close to the API boundary with Bean Validation, explicit controller checks, or a documented project-local convention.
- Keep outbound client configuration centralized so authentication headers, correlation identifiers, codecs, retries, and logging are consistent.
- Document profile-specific configuration.

## Examples

Good:

```java
@PostMapping("/billing/kpis")
ResponseEntity<BillingKpiResponse> getBillingKpis(@Valid @RequestBody BillingKpiRequest request) {
    return ResponseEntity.ok(dashboardService.getBillingKpis(request));
}
```

Bad:

```java
@PostMapping("/billing/kpis")
Object getBillingKpis(@RequestBody Map<String, Object> request) {
    return new RestTemplate().postForObject(serviceUrl + "/kpis", request, Object.class);
}
```

## Verification

Reviewers check property definitions, validation, authorization, API compatibility, transaction boundaries, health endpoints, scheduled-job configuration, outbound-client configuration, and dependency injection style.
