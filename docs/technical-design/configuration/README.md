# Configuration

## Purpose

This section describes the runtime configuration surfaces of Veil.

Veil configuration currently comes from four places:

- local files such as `config.json` and `provider.json`
- wallet-generated defaults
- environment variables
- CLI flags for selected runtime options

## Reading Order

1. [defaults.md](./defaults.md)
2. [consumer.md](./consumer.md)
3. [provider.md](./provider.md)
4. [proxy.md](./proxy.md)
5. [relay.md](./relay.md)
6. [bootstrap.md](./bootstrap.md)
7. [environment.md](./environment.md)

## Main Config Objects

- Consumer config: local `config.json`
- Provider config: local `provider.json`
- Proxy config: local loopback port, upstream URL, and shared secret
- Relay config: port, DB path, rate-limit and discovery-related environment variables
- Bootstrap config: bootstrap URL, cache TTL, Relay selection limits

## Scope

This section documents what the current codebase actually reads and validates.

## Vision Guardrails

Configuration evolution should remain aligned with product vision constraints:

- keep Relay as an explicit market role, not only a transport hop
- preserve the automation path for Claw or equivalent agent operation
- keep quote budgeting semantics separate from final settlement asset semantics
