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

---

## Implementation Details

**Source:** `src/relay/index.ts`, `src/relay/witness.ts`, `src/relay/rate_limiter.ts`

### Key Data Structures

```ts
// src/relay/index.ts
interface ConnectedProvider {
  conn: Connection;
  info: ProviderInfo;
}

// In-memory state
const providers = new Map<string, ConnectedProvider>();  // provider_pubkey -> conn+info
const consumers = new Map<string, Connection>();          // request_id -> consumer conn
const requestMeta = new Map<string, {
  consumerPubkey: string;
  providerId: string;
  model: string;
  startTime: number;
}>();

// src/relay/rate_limiter.ts
class RateLimiter {
  private counts = new Map<string, number[]>(); // pubkey -> timestamp array
  constructor(limit: number, windowMs: number = 60000);
  tryAcquire(pubkey: string): { success: boolean; retryAfter?: number };
}

// src/relay/witness.ts
interface WitnessRecord {
  request_id: string;
  consumer_pubkey: string;
  provider_pubkey: string;
  model: string;
  input_tokens: number;
  output_tokens: number;
  cache_read_tokens?: number;
  cache_write_tokens?: number;
  duration_ms: number;
  timestamp: number;
  relay_pubkey: string;
  relay_signature: string;
}
```

### Core Flow

1. **Provider registration**: `provider_hello` → verify Ed25519 signature → upsert `provider_state` in SQLite → send `provider_ack`
2. **Consumer request**: verify signature + timestamp age (MAX_REQUEST_AGE_MS = 5 min) → rate limit check → find provider → forward with `consumer_pubkey` redacted to `'redacted'`
3. **Response routing**: match `request_id` → forward to consumer → on `response`/`stream_end`: create witness record in both legacy DB and WitnessStore
4. **Bootstrap registration**: POST to `{bootstrapUrl}/v1/relays/register` with signed payload, heartbeat every 30s

### Witness System (Dual Store)

- **Legacy witness table** in main `usage.db` (via `db.ts` `initDatabase`)
- **Dedicated WitnessStore** in separate `*-witness.db` with richer schema (cache tokens, duration_ms)
- Both stores receive the same witness data on request completion
- `signablePayload()` produces deterministic JSON with all fields except `relay_signature`

### Rate Limiter

- Sliding window: tracks timestamps per consumer pubkey
- Default: 60 requests per 60s window (configurable via `VEIL_RELAY_RATE_LIMIT` env)
- Returns `retryAfter` in seconds on rejection

### State Management

- **Persistent (SQLite)**: `provider_state` (online/offline), `witness` (legacy), dedicated witness DB
- **In-memory**: provider connections Map, consumer routing Map, request metadata Map, rate limiter

### Error Handling

- Invalid provider signature → `provider_ack { status: 'rejected', reason: 'invalid_signature' }`
- Invalid consumer signature → `error { code: 'invalid_signature' }`
- Rate limited → `error { code: '429', message: 'Retry-After: N' }`
- Provider not found → `error { code: 'no_provider' }`
- Duplicate witness insert → silently ignored (UNIQUE constraint)

## API Specification

### `startRelay(options: RelayOptions): Promise<{ close(): Promise<void> }>`

### `verifyRequest(outer, requestId, timestamp, innerBase64): boolean`

Verifies consumer request: checks timestamp age (±5 min), recomputes `inner_hash` from base64 payload, verifies Ed25519 signature over deterministic JSON.

### `createWitness(requestId, consumerPubkey, providerId, model, inputTokens, outputTokens, relayWallet): WitnessRecord`

Creates a signed witness with daily-salted consumer hash (`sha256(pubkey + YYYY-MM-DD)`) for privacy.

### WitnessStore API

| Method | Signature | Description |
|--------|-----------|-------------|
| `record()` | `(params, relaySecretKey) => WitnessRecord` | Sign and persist |
| `get()` | `(requestId) => WitnessRecord \| null` | Lookup by request_id |
| `verify()` | `(witness, relayPublicKey) => boolean` | Verify signature |
| `list()` | `(opts?) => WitnessRecord[]` | Filter by consumer/provider/since |
| `stats()` | `(opts?) => AggregateStats` | Totals, unique counts |
| `prune()` | `(retentionMs?) => number` | Delete old records (default 30d) |
| `export()` | `(opts?) => WitnessRecord[]` | Export for on-chain migration |

## Integration Protocol

### WebSocket Message Types Handled

| Type | Direction | Handler |
|------|-----------|--------|
| `provider_hello` | Provider→Relay | Register provider, verify sig, send `provider_ack` |
| `request` | Consumer→Relay | Verify, rate limit, forward to provider |
| `response` | Provider→Relay | Forward to consumer, record witness |
| `stream_start/chunk` | Provider→Relay | Forward to consumer |
| `stream_end` | Provider→Relay | Forward to consumer, record witness with usage |
| `list_providers` | Consumer→Relay | Return online provider list |
| `ping` | Any→Relay | Reply `pong` |

### Bootstrap Integration

- Register: `POST /v1/relays/register` (signed)
- Heartbeat: every 30s (same endpoint)
- Deregister: `DELETE /v1/relays/{pubkey}` on shutdown

## Current Implementation Status

- ✅ Provider registration with Ed25519 verification [IMPLEMENTED]
- ✅ Consumer request verification (signature + timestamp age) [IMPLEMENTED]
- ✅ Request forwarding with consumer identity redaction [IMPLEMENTED]
- ✅ Sliding window rate limiter [IMPLEMENTED]
- ✅ Dual witness store (legacy + dedicated WitnessStore) [IMPLEMENTED]
- ✅ WitnessStore: record, get, verify, list, stats, prune, export [IMPLEMENTED]
- ✅ Bootstrap registration + heartbeat + deregister [IMPLEMENTED]
- ✅ Provider disconnect cleanup (mark offline in DB) [IMPLEMENTED]
- ⚠️ Consumer privacy: daily-salted hash only (no per-request rotation) [PARTIAL]
- ❌ Witness export to on-chain format [DESIGN ONLY]
- ❌ Multi-relay witness cross-verification [DESIGN ONLY]
