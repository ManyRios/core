# Models

## Gateway-Visible Model IDs

Current model ids exposed by the gateway:

- `claude-sonnet-4-20250514`
- `claude-haiku-3-5-20241022`

## Source of Truth

The gateway-visible model list comes from the runtime configuration defaults in `src/config/bootstrap.ts`.

Clients should treat `GET /v1/models` as the runtime source of truth.
