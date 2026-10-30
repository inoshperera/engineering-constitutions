---
id: universal.engineering-principles
title: Engineering Principles
category: universal
version: 1.0.0
status: active
owners:
  - engineering-leadership
applies_to:
  - all
depends_on: []
---

# Engineering Principles

## Purpose

Define the default engineering principles that apply to all Entgra projects.

## Mandatory requirements

- Teams MUST prefer correctness over cleverness.
- Code MUST be maintainable, readable, and owned by an accountable team.
- Public behavior MUST remain backward compatible unless an approved migration plan exists.
- Designs MUST be secure by default and testable before release.
- Services MUST expose enough observability to diagnose production behavior.
- Material decisions MUST be documented with evidence, trade-offs, and ownership.
- Changes SHOULD be small enough for meaningful review.
- Changes MUST trace to an accepted issue or specification that states the problem, affected users, expected behavior, acceptance criteria, important error cases, permissions, and platform differences, and records who accepted it and which version they accepted.
- Changes MUST receive human acceptance: an independent reviewer accepts the code and test evidence, the team lead accepts the delivered user behavior (with stakeholder acceptance where required), and the release owner approves the exact build being released, including deployment and recovery readiness.
- Each human decision MUST record the approver, decision, date, approved artifact or version, and supporting evidence. A material change made after approval MUST be re-approved for the affected decision.
- AI reviews and passing CI checks MUST NOT replace human approval; they are supporting evidence only.

## Recommended practices

- Optimize for clear intent, simple control flow, and predictable operational behavior.
- Split large changes into reviewable increments.
- Record assumptions when requirements, data, or external systems are uncertain.
- Agree expected behavior, including failure cases, before implementation starts.
- Small changes may record requirements, evidence, and decisions in the issue and PR; separate documents or meetings are unnecessary when the evidence and decisions are clear.

## Verification

Reviewers verify that changes have clear ownership, a link to accepted requirements, tests, documentation, recorded human approvals, and an explicit compatibility or migration story when behavior changes.
