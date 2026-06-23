# 4) Implementation guide

## Implementation plan

1. Identify where delegated downstream calls currently fail due to missing user bearer token.
2. Integrate `FMI_PATH` token acquisition in the agent request flow.
3. Request only required scopes for each downstream API call.
4. Ensure credentials and identity settings are configured in the Blueprint/runtime.
5. Add functional and security validation.

## Reference API interaction

Agent calls:

```http
POST {FMI_PATH}/token
Content-Type: application/json
```

Payload pattern:

```json
{
  "oid": "<user-object-id>",
  "tid": "<tenant-id>",
  "scopes": [
    "https://graph.microsoft.com/.default"
  ]
}
```

## Python example

```python
import os
import requests

def get_delegated_token(oid: str, tid: str, scopes: list[str]) -> dict:
    fmi_path = os.environ["FMI_PATH"]
    payload = {"oid": oid, "tid": tid, "scopes": scopes}
    response = requests.post(f"{fmi_path}/token", json=payload, timeout=30)
    response.raise_for_status()
    return response.json()
```

## End-to-end flow in code

1. Receive request context from Foundry runtime.
2. Read `oid` and `tid` claims.
3. Acquire delegated token through `FMI_PATH`.
4. Call downstream API with returned token.
5. Return business response; never return token content.

## Configuration checklist

1. Correct delegated scopes for Graph/custom APIs.
2. Required user/admin consent granted.
3. Agent identity configuration valid (managed identity/app identity path).
4. Blueprint credentials configured securely.
5. Environment variable `FMI_PATH` available at runtime.

## Test plan

### Functional tests

1. Happy path: downstream delegated API call succeeds.
2. Invalid/expired delegated token path returns explicit error.
3. Missing scope returns authorization failure with clear diagnostics.

### Security tests

1. Verify no token data appears in logs.
2. Verify no token persistence in files, memory cache, or telemetry payloads.
3. Verify failure handling does not fallback to unsafe auth behavior.

## Rollout strategy

1. Enable in dev with masked tracing.
2. Promote to test/staging with real tenant validation.
3. Canary rollout in production.
4. Keep rollback path to previous auth behavior (without delegated downstream calls).
