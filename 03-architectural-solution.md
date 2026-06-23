# 3) Architectural solution

## Recommended architecture

Use Foundry-mediated delegated token acquisition through `FMI_PATH` rather than raw bearer pass-through.

In practice:

1. The user bearer token authenticates the original client request.
2. Foundry runtime passes user identity context (`oid`, `tid`) to the agent.
3. The agent calls `POST {FMI_PATH}/token` with user context + requested scopes.
4. Foundry identity services issue a delegated token tied to agent identity and user context.
5. The agent uses that token for downstream API calls.

## What `FMI_PATH` represents

`FMI_PATH` is an environment-provided internal endpoint for Foundry Managed Identity/token exchange operations.  
The agent does not build tenant/agent URLs manually; Foundry injects the correct path.

## Why this design is the best fit

1. Keeps raw user tokens out of agent code.
2. Preserves user-context authorization through delegated scopes.
3. Provides clearer per-agent audit trails (agent identity + user context).
4. Centralizes token issuance controls and policy enforcement.

## Target flow (to-be)

```text
User -> Client App -> Foundry Hosted Agent (oid, tid only)
     -> FMI_PATH token exchange
     -> Delegated access token (scoped, short-lived)
     -> Downstream API (Graph/custom)
```

## Alternatives considered

### A) Direct user-token pass-through to agent

- Benefit: lowest short-term implementation effort.
- Drawback: significantly higher token leakage risk and weaker governance.

### B) Agent-only application permissions

- Benefit: simpler operationally.
- Drawback: loses user-context authorization semantics.

### C) External token broker outside Foundry

- Benefit: broad enterprise reuse.
- Drawback: extra complexity versus native `FMI_PATH` flow.

## Security design rules

1. Never log raw JWTs or token payloads.
2. Use least-privilege scopes only.
3. Keep delegated tokens short-lived and non-persistent.
4. Handle token exchange failures explicitly; no silent fallback.
5. Maintain key/credential rotation processes.
