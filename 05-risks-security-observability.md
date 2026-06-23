# 5) Risks, security, and observability

## Primary risks

1. Token leakage through logs or telemetry.
2. Over-scoped delegated tokens.
3. Misdiagnosed auth failures (identity vs authorization vs token exchange).
4. Environment drift causing inconsistent behavior.

## Mitigations

1. Strict log sanitization and redaction.
2. Least-privilege scope strategy by endpoint/use case.
3. Clear error taxonomy: `identity_context`, `token_exchange`, `downstream_auth`.
4. Configuration-as-code with deployment validation checks.

## Observability model

### Metrics

1. `fmi_token_exchange_success_rate`
2. `fmi_token_exchange_latency_ms`
3. `downstream_auth_401_403_rate`
4. `agent_request_end_to_end_latency_ms`

### Logging

1. Correlation ID across client, agent, FMI, and downstream API.
2. Safe identity diagnostics (`oid`, `tid`) only when needed.
3. No raw JWT, no token body, no authorization headers.

### Alerting

1. Sustained drop in token exchange success rate.
2. Sudden increase in 401/403 from downstream APIs.
3. Latency degradation beyond defined SLO.

## Operational runbook

1. Confirm `FMI_PATH` is present and reachable.
2. Validate agent identity and credential health.
3. Validate required scopes and consent state.
4. Inspect downstream API audience/scope expectations.
5. If needed, rollback to last stable release and disable affected feature paths.
