# Metering & Witness

## Purpose

This module turns upstream usage into normalized quote estimates and settlement-ready witness records.

## Responsibility Boundary

- normalize usage from upstream providers
- compute model-based cost estimates
- sign and store Relay witness records
- support witness query, stats, prune, and export

## Out Of Scope

- does not route requests
- does not choose Providers
- does not own contribution accounting

## Interface

```ts
type QuoteUnit = 'usd_estimate';

interface NormalizedUsage {
  input: number;
  output: number;
  cacheRead?: number;
  cacheWrite?: number;
}

interface PricingSnapshot {
  version: string;
  model: string;
  quoteUnit: QuoteUnit;
}

interface QuoteEstimate {
  total: number;
  quoteUnit: QuoteUnit;
}

interface WitnessRecord {
  requestId: string;
  providerId: string;
  relayId: string;
  model: string;
  usage: NormalizedUsage;
  pricingVersion: string;
  quoteUnit: QuoteUnit;
  completionStatus: 'success' | 'error' | 'aborted';
  evidenceHash: string;
  dedupeKey: string;
  relaySignature: string;
}

function normalizeUsage(usage: AnyUsage): NormalizedUsage;
function buildQuoteEstimate(
  usage: NormalizedUsage,
  pricing: PricingSnapshot,
): QuoteEstimate;

class WitnessStore {
  record(...): WitnessRecord;
  verify(...): boolean;
  export(...): WitnessRecord[];
}
```

## Data Flow

Input: upstream usage objects, Relay request metadata, deterministic pricing snapshot.  
Process: normalize usage, build quote estimate, sign witness payload with pricing version and evidence hash, persist record.  
Output: quote estimates and verifiable witness records.

## State

- persistent: witness database records, pricing snapshot references
- memory: per-request usage totals and quote estimates

## Errors

- unknown model pricing
- missing pricing snapshot version
- malformed usage payload
- invalid witness signature
- evidence hash mismatch
- duplicate request id

## Security Constraints

- witness signatures must cover deterministic payloads
- witness signatures must cover pricing version, quote unit, and evidence hash
- exported witness records must remain verifiable
- quote estimation should be conservative on unknown pricing

## Test Requirements

- usage normalization across providers
- quote estimate generation
- witness record, lookup, stats, prune, export
- signature verification
- pricing-version mismatch rejection

## Dependencies

- calls: `crypto`
- called by: `consumer`, `relay`, `operations`, `settlement-payout`
