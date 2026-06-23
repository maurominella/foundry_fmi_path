# 1) Challenge context

## Executive summary

When a client application invokes a Microsoft Foundry Agent with a delegated user token (`Authorization: Bearer ...`), the Foundry runtime does **not** expose the raw bearer token to agent code.  
Instead, the runtime provides user identity context through claims such as:

1. `oid` (user object id)
2. `tid` (tenant id)

This means the agent knows **who** the user is, but does not directly possess the user's bearer token for downstream delegated API calls.

## Problem statement

The engineering challenge is to enable secure delegated access from a Foundry Hosted Agent to downstream APIs (for example Microsoft Graph) without direct pass-through of the original user token, while preserving per-user authorization behavior and auditability.

## Why this happens

Foundry intentionally strips the raw user token before the request reaches agent code. This reduces token exposure risk and keeps agent runtime safer by default.

## Scope

### In scope

1. End-to-end identity and token flow analysis.
2. Clarification of runtime behavior (`oid`/`tid` available, raw bearer absent).
3. Architecture for delegated downstream access via Foundry token exchange.
4. Practical implementation guidance.

### Out of scope

1. Unrelated refactoring of non-authentication components.
2. Generic security hardening not connected to this flow.

## Technical goals

1. Allow the agent to call downstream APIs in user context.
2. Avoid insecure bearer pass-through patterns.
3. Keep strong traceability and tenant/user attribution.

## Observed symptoms

1. Agent receives user context (`oid`, `tid`) but no usable raw user bearer.
2. Delegated downstream calls fail if no replacement token is acquired.
3. Teams perceive a mismatch: user identity is known, authorization token is unavailable.

## Constraints and assumptions

1. No raw token persistence in logs, state, or storage.
2. Least-privilege scopes and short token lifetimes are required.
3. Solution must fit standard CI/CD and environment promotion pipelines.
