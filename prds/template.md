---
# ============================================================================
# Managed section — preserve the document metadata when updating this file
# ============================================================================
id: PRD-0000
type: prd
title: "Descriptive title of the product initiative"
status: draft
created_at: YYYY-MM-DD
updated_at: YYYY-MM-DD
# ============================================================================
owners:
  - "@owner"
stakeholders: []
tags: []
---

# PRD-0000: Descriptive title of the product initiative

## Overview

Provide a concise (1–2 paragraphs) summary of the product capability, why it matters, and the expected outcome.

## Problem Statement

- What user or business problem exists today, and what is the impact of not solving it?
- Who are the target users or personas affected?
- What evidence, research, or metrics support this need?

## Goals and Success Metrics

- **Primary Metrics**:
  - `[Metric 1]`: Baseline -> Target (how and when it will be measured)
  - `[Metric 2]`: Baseline -> Target
- **Guardrail Metrics**: Metrics that must not degrade as a result of this initiative (e.g., error rates, latency, retention).

## User Scenarios

1. **Scenario 1 — [Primary Flow]**: End-to-end user journey from trigger to expected outcome.
2. **Scenario 2 — [Alternative / Error Flow]**: How the product behaves when preconditions fail or limits are reached.

## Scope

### In-Scope
- **REQ-01**: Core user-facing capability or requirement 1
- **REQ-02**: Core user-facing capability or requirement 2

### Out-of-Scope (Non-Goals)
- Capabilities or workflows explicitly **excluded** from this initiative

## Constraints and Risks

- **Privacy, Compliance & Security**: Data sensitivity, regulatory obligations, or access rules.
- **Dependencies & Assumptions**: Upstream teams, third-party services, or operational prerequisites.
- **Risks**: Potential adoption or delivery risks and how they will be mitigated.

## Open Questions

- [ ] Open question 1 to be resolved during review
- [ ] Open question 2

## Adoption Plan & Follow-up Artifacts

Once this PRD is approved (`status: approved`), create the following downstream artifacts:

- **RFCs to Create**:
  - [ ] `RFC-NNNN`: [Title of technical proposal when architectural discussion is needed]
- **Specs to Create**:
  - [ ] `SPEC-NNNN`: [Title of executable specification for implementation]
