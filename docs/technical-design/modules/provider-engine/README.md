# Provider Engine

## Purpose

This module owns the execution boundary where sealed requests become upstream AI calls.
Provider execution is part of the `Market` path because published capacity is only real when it can execute under policy, and it must preserve evidence continuity needed by `Settlement`.

## Responsibility Boundary

- keep one or more Relay connections open
- decrypt sealed requests
- call upstream AI services
- return normal or streaming responses
- enforce local concurrency and credential boundaries

## Out Of Scope

- does not decide global routing policy
- does not persist Relay witness records
- does not expose a public client-facing API surface

## Interface

```ts
type RelayMode = 'bootstrap' | 'static' | 'manual';

interface ProviderOptions {
  wallet: Wallet;
  relayMode?: RelayMode;
  relayUrls?: string[];
  bootstrapUrl?: string;
  apiKeys: Array<{ provider: 'anthropic'; key: string }>;
  maxConcurrent: number;
}

function startProvider(options: ProviderOptions): Promise<{ close(): Promise<void> }>;
function handleRequest(...): Promise<HandleRequestResult>;
```

## Data Flow

Input: Relay-forwarded `request`.  
Process: maintain one or more Relay connections from discovery or configured endpoints, decrypt inner payload, build upstream request, call model API, stream or aggregate output.  
Output: `response`, `stream_start`, `stream_chunk`, `stream_end`, or `error`.

## State

- persistent: `provider.json`, encrypted API key material
- memory: active request count, Relay connections, metrics store

## Errors

- capacity reached
- upstream auth failure
- upstream 429 and 5xx
- malformed streaming body
- no reachable Relay for the selected mode

## Security Constraints

- upstream credentials must stay encrypted at rest
- Provider should not receive unnecessary Consumer identity
- plaintext should stay inside the execution boundary

## Test Requirements

- non-streaming request
- streaming request
- retry behavior
- max concurrency
- multi-Relay connectivity

## Dependencies

- calls: `network`, `crypto`, `metrics`
- called by: `relay`

---

## Implementation Details

**Source:** `src/provider/index.ts`, `src/provider/metrics.ts`

### Key Data Structures

```ts
// src/provider/index.ts
export interface ProviderOptions {
  wallet: Wallet;
  relayUrl: string;
  apiKeys: Array<{ provider: 'anthropic'; key: string }>;
  maxConcurrent: number;
  proxyUrl?: string;       // LiteLLM/custom proxy support
  proxySecret?: string;    // shared secret for proxy auth
  healthPort?: number;     // default 9962
  discoveryClient?: RelayDiscoveryClient;
}

export interface HandleRequestResult {
  content: string;
  usage: { input_tokens: number; output_tokens: number };
  finish_reason: string;
}

// src/provider/metrics.ts
class MetricsStore {
  totalRequests: number;
  totalErrors: number;
  modelRequests: Record<string, number>;
  recentLatencies: number[];  // rolling window of 1000
}
```

### Core Flow

1. **Relay connection**: connect to primary relay via WebSocket, send signed `provider_hello`
2. **Multi-relay**: if `discoveryClient` is available, connect to up to `MULTI_RELAY_COUNT - 1` (2) additional relays
3. **Request handling**: decrypt inner envelope (`nacl.box.open`), extract consumer encryption pubkey from first 32 bytes of sealed data
4. **Upstream call**: `handleRequest()` builds Anthropic API request, handles OAuth tokens specially (Claude Code headers + system prompt injection)
5. **Response encryption**: seal response with consumer's encryption pubkey before sending back
6. **Retry**: exponential backoff with jitter for 429/529/500 errors (max 3 retries)

### Upstream API Handling

- **Standard API key**: `x-api-key` header
- **OAuth token** (`sk-ant-oat`): Bearer auth + `anthropic-beta`, `anthropic-dangerous-direct-browser-access`, `user-agent: claude-cli/2.1.75` headers. Injects Claude Code system prompt.
- **Proxy mode**: `x-proxy-secret` header, uses `proxyUrl` as API base
- Model mapping: `MODEL_MAP` translates veil model IDs to Anthropic model IDs

### Streaming Protocol

1. Send `stream_start` with model
2. Send role chunk `{ role: 'assistant' }` (encrypted)
3. Forward each `content_block_delta` text as encrypted `stream_chunk` with index
4. Send finish_reason chunk (encrypted)
5. Send `stream_end` with usage totals

### Concurrency Control

- `activeRequests` counter incremented on entry, decremented in `finally`
- Requests exceeding `maxConcurrent` rejected with `rate_limit` error

### Health Server

- Separate Hono HTTP server on port 9962 (configurable via `VEIL_PROVIDER_HEALTH_PORT`)
- `GET /health`: status, uptime, models, capacity, version
- `GET /metrics`: total requests, error rate, model breakdown, latency percentiles (p50/p95/p99)

### Error Handling

- Decrypt failure → `decrypt_failed`
- Upstream 401 → `upstream_auth`
- Upstream 400 → forward Anthropic error message
- Upstream 429/529/500 → retry with backoff
- All errors recorded in MetricsStore

## API Specification

### `startProvider(options: ProviderOptions): Promise<{ close(): Promise<void> }>`

Connects to relay(s), registers as provider, starts health server.

### `handleRequest(inner, apiKey, onChunk?, apiBase?, proxySecret?): Promise<HandleRequestResult>`

Executes upstream Anthropic API call. Handles both streaming and non-streaming.

### MetricsStore

```ts
recordRequest(model: string, latencyMs: number, isError: boolean): void
getMetrics(): {
  total_requests: number;
  error_rate: number;
  models: Record<string, number>;
  latency: { p50: number; p95: number; p99: number };
}
```

## Integration Protocol

- **→ Relay (WebSocket)**: sends `provider_hello` (signed), `response`, `stream_start/chunk/end`, `error`; receives `provider_ack`, `request`
- **→ Discovery Client**: `fetchRelays()` for multi-relay connectivity
- **→ Anthropic API**: HTTP POST to `/v1/messages` (streaming SSE or JSON)
- **→ Crypto**: `open()` for decryption, `seal()` for response encryption, `sign()` for hello signature
- **Config**: `MODEL_MAP`, `RETRY_CONFIG` from `src/config/bootstrap.ts`

## Current Implementation Status

- ✅ Single + multi-relay connectivity [IMPLEMENTED]
- ✅ E2E encryption (decrypt request, encrypt response) [IMPLEMENTED]
- ✅ Streaming + non-streaming upstream calls [IMPLEMENTED]
- ✅ Retry with exponential backoff + jitter [IMPLEMENTED]
- ✅ Concurrency limiting [IMPLEMENTED]
- ✅ OAuth token support (Claude Code headers) [IMPLEMENTED]
- ✅ Proxy mode (LiteLLM compatible) [IMPLEMENTED]
- ✅ Health + metrics HTTP server [IMPLEMENTED]
- ✅ MetricsStore with p50/p95/p99 latency [IMPLEMENTED]
- ❌ Capacity publication to market [DESIGN ONLY]
- ❌ Credential rotation without restart [DESIGN ONLY]
- ❌ Multi-provider backend (only Anthropic) [DESIGN ONLY]
