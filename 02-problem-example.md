# 2) Problem example

## Minimal reproducible scenario

### Preconditions

1. A signed-in user calls a client app with a valid delegated token.
2. The client app invokes a Foundry Hosted Agent.
3. The agent needs to call a downstream API that requires delegated user access.

### Current sequence (as-is)

1. Client sends request with `Authorization: Bearer <user-token>`.
2. Foundry runtime strips the raw bearer before invoking agent code.
3. Agent receives identity context only (`oid`, `tid`).
4. Agent tries downstream delegated call.
5. Call fails unless a new delegated token is acquired through Foundry identity flow.

## Concrete evidence

### Incoming request (redacted)

```http
Authorization: Bearer <user-jwt-redacted>
```

### Context available to the agent

```json
{
  "identity": {
    "oid": "<user-object-id>",
    "tid": "<tenant-id>"
  },
  "raw_user_access_token": null
}
```

### Typical failure (without token exchange)

```text
401 Unauthorized / 403 Forbidden
Cause: delegated user token not available in agent runtime
```

## Where `FMI_PATH` fits

Foundry provides an internal path through `FMI_PATH` so the agent can request a new delegated token securely using:

1. User identity claims (`oid`, `tid`)
2. Agent identity (appId / managed identity context)
3. Blueprint credentials (secret, or managed identity + federated credential)

## Impact

1. Without this flow, user-context API access is blocked.
2. Features depending on delegated Graph/custom API calls cannot work.
3. Developers may incorrectly assume `oid`/`tid` is enough for authorization.

## Acceptance criteria

1. Downstream delegated calls succeed in user context.
2. No raw user bearer token is logged or persisted.
3. Expired/invalid token scenarios are handled explicitly and safely.
