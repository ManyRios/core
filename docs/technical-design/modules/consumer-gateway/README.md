# Consumer Gateway

## Purpose

This module exposes Veil as a local OpenAI-compatible gateway for end users, tools, and agents.

## Responsibility Boundary

- expose an OpenAI-compatible HTTP API
- authenticate local callers
- select a Provider from Relay-published capacity
- sign and seal outbound requests
- convert Relay responses into client-facing JSON or SSE
- enforce per-request budget and timeout

## Out Of Scope

- does not execute upstream inference directly
- does not persist witness or contribution ledger data
- does not define transport protocol semantics on its own

## Interface

```ts
type RelayMode = 'bootstrap' | 'static' | 'manual';
type QuoteUnit = 'usd_estimate';

interface GatewayOptions {
  port: number;
  wallet: Wallet;
  relayMode?: RelayMode;
  relayUrl?: string;
  bootstrapUrl?: string;
  apiKey?: string;
  defaultQuoteBudget?: number;
  quoteUnit?: QuoteUnit;
}

function startGateway(
  options: GatewayOptions,
): Promise<{ close(): Promise<void>; port: number }>;
```

## Data Flow

Input: HTTP `chat/completions` request.  
Process: auth, model validation, Relay discovery or selection, Provider selection, request signing, payload sealing, Relay round-trip, response decode.  
Output: OpenAI-compatible response.

## State

- memory: Relay connection, Provider list, pending requests, budget trackers
- persistence: none beyond local wallet and config files owned elsewhere

## Errors

- invalid request body
- unknown model
- Relay unavailable
- Provider unavailable
- streaming interruption
- no usable Relay discovered for the selected mode

## Security Constraints

- local API auth must be supported
- request signatures must bind request metadata and ciphertext hash
- budget checks must fail closed
- quote budgeting must stay separate from final settlement semantics

## Test Requirements

- OpenAI-compatible request and response
- streaming
- budget overrun and timeout
- no-Provider path

## Dependencies

- calls: `network`, `crypto`, `wallet`, `budget`
- called by: local AI clients, IDEs, agents
