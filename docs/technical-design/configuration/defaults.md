# Defaults

## Core Defaults

These defaults come from `src/config/bootstrap.ts` and the runtime modules:

| Key | Default |
|-----|---------|
| Current official Relay default URL | `wss://relay-jp.runveil.io` |
| Bootstrap URL | `https://bootstrap.runveil.io` |
| Gateway port | `9960` |
| Relay port | `8080` |
| Provider health port | `9962` |
| Request max age | `300000 ms` |
| Ping interval | `30000 ms` |
| Pong timeout | `10000 ms` |
| Max WebSocket payload | `10 MB` |
| Discovery cache TTL | `60000 ms` |
| Discovery max relays | `20` |
| Default request quote budget | `1.0 USDC` |
| Default request duration | `120000 ms` |
| Relay rate limit | `60` requests per identity window |

## Interpretation Notes

- the current official Relay URL is a runtime default and onboarding convenience, not a permanent single-operator architecture assumption
- the default request quote budget is a budgeting and comparison value expressed in the current quote language; it is not the final settlement asset

## Retry Defaults

Provider retry settings:

| Key | Default |
|-----|---------|
| maxRetries | `3` |
| baseDelayMs | `1000` |
| maxDelayMs | `30000` |
| jitterFactor | `0.5` |

## Built-In Models

Current gateway-visible model ids:

- `claude-sonnet-4-20250514`
- `claude-haiku-3-5-20241022`
