# Agents and Automation

## Purpose

Automated tools should call the local Veil gateway the same way a normal OpenAI-compatible client does.
The gateway is only the automation access surface; the product behavior still depends on Relay/Provider market roles behind it.
Budget fields in automation requests are quote-time controls and should stay separate from settlement asset interfaces.

## Base Rule

Treat Veil as a local HTTP endpoint:

```text
http://localhost:9960/v1
```

Do not connect automation directly to Relay or Provider processes.

## Typical Setup

1. start the local gateway
2. point the agent or script base URL at `http://localhost:9960/v1`
3. provide the local bearer token if API auth is enabled
4. choose one of the model ids returned by `GET /v1/models`

## Good Integration Pattern

- keep all application logic at the client layer
- let Veil handle routing, budgets, and execution
- read health and model availability from the local gateway

## Avoid

- hard-coding Relay endpoints into automation
- assuming every OpenAI API surface is implemented
- bypassing local auth checks in shared environments

For market and settlement boundaries that automation should respect, see [Governance and Economics](../product-design/governance-and-economics/README.md).
