# Clients

## Purpose

This section explains how callers integrate with Veil through the local gateway.

Veil currently exposes a narrow client-facing surface:

- `GET /health`
- `GET /v1/models`
- `POST /v1/chat/completions`

## Reading Order

1. [openai-compatible.md](./openai-compatible.md)
2. [curl.md](./curl.md)
3. [javascript.md](./javascript.md)
4. [ide.md](./ide.md)
5. [agents.md](./agents.md)
6. [models.md](./models.md)

## Main Rule

Clients should treat Veil as a local OpenAI-compatible gateway, not as a direct Relay or Provider API.
This local gateway is the access surface of a Relay/Provider market network, not a standalone proxy product.

## Network Context

- The local gateway is an access entrypoint, while market behavior is defined by Relay admission, Provider execution, and witness flows.
- Quote budgets used by clients are routing-time controls and do not replace settlement interfaces.
- For role boundaries and request ownership, see [System Model](../technical-design/system-model/README.md).
- For economic and settlement boundaries, see [Governance and Economics](../product-design/governance-and-economics/README.md).
