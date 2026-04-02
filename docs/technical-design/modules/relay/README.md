# Relay

## Purpose

This module is the control-plane broker between Consumers and Providers.

## Responsibility Boundary

- accept Provider registration
- verify Consumer requests
- apply rate limits
- route requests to online Providers
- map request ids back to Consumers
- record witness data on completion

Relay is both a routing control node and a market witness role, and its records must remain compatible with quote-to-settlement separation.

## Out Of Scope

- does not decrypt sealed business payloads
- does not act as a public client API gateway
- does not own contributor accounting

## Interface

```ts
interface RelayOptions {
  port: number;
  wallet: Wallet;
  dbPath: string;
  bootstrapUrl?: string;
  witnessDbPath?: string;
}

function startRelay(options: RelayOptions): Promise<{ close(): Promise<void> }>;
function verifyRequest(...): boolean;
function createWitness(...): RelayWitness;
```

## Data Flow

Input: Provider `provider_hello`, Consumer `request`, Provider `response`, and `stream_end`.  
Process: verify, limit, look up Provider, forward, persist witness.  
Output: forwarded protocol messages and witness records.

## State

- persistent: `provider_state`, `usage_log`, `witness`, dedicated `witness.db`
- memory: online Provider map, request metadata map, Consumer connection map, rate limiter

## Errors

- invalid Provider signature
- invalid Consumer signature
- rate limit rejection
- missing Provider
- duplicate witness insert

## Security Constraints

- Relay must not decrypt business payloads
- request age must be enforced
- witness records must be signed and exportable

## Test Requirements

- Provider registration
- Consumer verification
- rate limiting
- witness record and verification

## Dependencies

- calls: `network`, `crypto`, `db`, `witness`, `rate_limiter`
- called by: `consumer`, `provider`
