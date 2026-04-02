# Bootstrap & Discovery

## Purpose

This module defines how Relay nodes are registered, refreshed, listed, and selected.
It keeps Relay as an explicit market role by making broker availability and selection a visible operational dependency.

## Responsibility Boundary

- register Relay nodes
- accept Relay heartbeats
- publish Relay lists
- score and select Relays on the client side

## Out Of Scope

- does not forward inference traffic
- does not decrypt business payloads
- does not store Consumer prompt content

## Interface

```ts
function createBootstrapApp(db: Database): Hono;

class RelayDiscoveryClient {
  fetchRelays(region?: string): Promise<RelayInfo[]>;
  selectRelay(exclude?: string[]): Promise<RelayScore | null>;
  refreshCache(): Promise<RelayInfo[]>;
}
```

## Data Flow

Input: Relay register and heartbeat requests, Consumer and Provider list fetches.  
Process: verify signatures, update registry state, cache Relay lists, compute scores.  
Output: active Relay inventory and best-Relay selections.

## State

- persistent: `relay_registry`
- memory: cached Relay list, latency measurements

## Errors

- invalid signature
- duplicate endpoint
- stale Relay state
- bootstrap fetch failure

## Security Constraints

- Bootstrap handles Relay metadata only
- Relay registration must be signed
- selection inputs must be bounded and cacheable

## Test Requirements

- register, heartbeat, offline prune
- cached list behavior
- score calculation and selection

## Dependencies

- calls: `crypto`, `logger`
- called by: `consumer`, `provider`, Relay operators
