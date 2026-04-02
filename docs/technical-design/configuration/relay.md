# Relay Configuration

## Main Runtime Inputs

Relay configuration is currently assembled from runtime options and environment variables.

Primary runtime options:

- `port`
- `wallet`
- `dbPath`
- `bootstrapUrl`
- `witnessDbPath`

## Why This Config Exists

- Relay runtime options keep Relay as an explicit market broker and witness role, not only a transport hop.
- `dbPath` and `witnessDbPath` preserve reproducible evidence required for settlement replay.
- rate-limit and region controls keep admission behavior bounded while supporting discoverable market operation.

## Environment Variables

- `VEIL_RELAY_PORT`: override the default Relay port
- `VEIL_RELAY_RATE_LIMIT`: request-rate control default
- `VEIL_RELAY_REGION`: region value published to Bootstrap

## Persistent State

Relay writes to:

- `relay.db`
- `witness.db`

## Bootstrap Registration

When `bootstrapUrl` is configured, Relay can publish:

- endpoint
- supported models
- fee
- region
- capacity
- active request state

## Operational Note

Relay configuration is partly code-driven today. This section should be kept aligned with `startRelay(...)` and the CLI flags that feed it.
