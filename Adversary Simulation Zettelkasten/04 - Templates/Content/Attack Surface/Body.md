## Overview

> [!todo] Todo: DELETE ME
> > [!tip]
> > Document the security-relevant architecture, trust boundaries, and common attack opportunities for the platform or service.

## Architecture Summary

> [!todo]- Diagram/Visual Aid
> Add an architecture diagram or other visual aid when it materially improves understanding of the attack surface, trust boundaries, or major data/identity flows.

| Component | Role | Exposure/Trust | Why It Matters |
| --------- | ---- | -------------- | -------------- |
| <!-- Core service, listener, or management plane --> | <!-- What this component does --> | <!-- Who can reach it or which trust zone it belongs to --> | <!-- Why assessors should care --> |
| <!-- Supporting service, identity, or integration point --> | <!-- What this component does --> | <!-- Who can reach it or which trust zone it belongs to --> | <!-- Why assessors should care --> |
| <!-- Data store, agent, broker, or remote dependency --> | <!-- What this component does --> | <!-- Who can reach it or which trust zone it belongs to --> | <!-- Why assessors should care --> |

## Trust Boundaries

| Boundary Type | What Crosses It | Why It Matters |
| ------------- | --------------- | -------------- |
| Identity Boundary | <!-- Where identities are established, federated, or delegated --> | <!-- Why it matters during assessment --> |
| Administrative Boundary | <!-- Where privileged actions are performed --> | <!-- Why it matters during assessment --> |
| Data Boundary | <!-- Where sensitive data is processed, stored, or transmitted --> | <!-- Why it matters during assessment --> |
| Network Boundary | <!-- Where traffic crosses zones, segments, or external interfaces --> | <!-- Why it matters during assessment --> |

## Key Components

### <!-- Component Name -->

> [!todo]- Diagram/Visual Aid
> Add an architecture diagram or other visual aid when it materially improves understanding of the attack surface component.

**Purpose**: <!-- What this component does -->

**Security Relevance**:
- <!-- Why it matters to a red team -->

**Notable Interfaces**:
- <!-- API, web UI, agent, protocol, service account, webhook, etc. -->

**Assumptions/Weak Points**:
- <!-- Common design assumptions, weak defaults, or risky dependencies -->

## Assessment Notes

### Common Attack Paths

| Attack Path | Preconditions | Impact |
| ----------- | ------------- | ------ |
| <!-- High-level path or abuse chain --> | <!-- Required access, trust, or exposed capability --> | <!-- What the attacker gains --> |

### Security Model

| Control Area | Notes |
| ------------ | ----- |
| Authentication | <!-- Authentication assumptions --> |
| Authorization | <!-- Authorization assumptions --> |
| Isolation/Tenancy | <!-- Isolation or trust assumptions --> |

### Common Weaknesses

| Weakness | Why It Recurs | Impact |
| -------- | ------------- | ------ |
| <!-- Misconfiguration, exposure pattern, or recurring failure mode --> | <!-- Why it commonly appears --> | <!-- What it enables --> |

## Defensive Notes

### High-Value Controls

| Control | Impact |
| ------- | ------ |
| <!-- Control that meaningfully reduces risk --> | <!-- Why it is especially valuable --> |

### Logging/Telemetry Considerations

| Source | What It Shows | Investigative Use |
| ------ | ------------- | ----------------- |
| <!-- Useful logs, telemetry, or investigative choke point --> | <!-- What the source captures --> | <!-- Why analysts care --> |
