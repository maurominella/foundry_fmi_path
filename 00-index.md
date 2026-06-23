# Foundry Hosted Agent - User Token Propagation Challenge

This folder contains a complete analysis package for the challenge of propagating user-delegated access to a Foundry Hosted Agent when the original user bearer token is not exposed to agent code.

## Document structure

1. [01-challenge-context.md](./01-challenge-context.md) - context, problem statement, goals, and scope.
2. [02-problem-example.md](./02-problem-example.md) - reproducible example of the current behavior.
3. [03-architectural-solution.md](./03-architectural-solution.md) - target architecture and design rationale.
4. [04-implementation-guide.md](./04-implementation-guide.md) - implementation approach, pseudocode, rollout, and validation.
5. [05-risks-security-observability.md](./05-risks-security-observability.md) - risks, security controls, observability, and runbook.

## Intended outcome

After reading these files, an engineer should be able to:

1. Explain why only `oid` and `tid` are available inside the agent.
2. Demonstrate the failure mode caused by the missing raw user bearer token.
3. Describe and justify the recommended architecture based on `FMI_PATH` token exchange.
4. Implement and operate the solution safely in real environments.
