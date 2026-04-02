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
