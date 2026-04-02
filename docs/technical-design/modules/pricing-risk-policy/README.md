# Pricing & Risk Policy

## Purpose

This module defines how Provider capacity is priced, exposed, paused, or limited in the market.

## Responsibility Boundary

- define provider-side pricing rules and offer floors
- convert operator policy into sell-side limits
- decide whether capacity should be published, repriced, throttled, or paused
- provide deterministic pricing snapshots for witness and settlement consumers

## Out Of Scope

- does not execute inference requests
- does not persist witness records
- does not transfer funds or custody assets
- does not replace Relay admission controls

## Interface

```ts
type QuoteUnit = 'usd_estimate';

interface CapacityOffer {
  model: string;
  capacityUnits: number;
  askQuoteUnitsPerUnit: number;
  quoteUnit: QuoteUnit;
  settlementAssetHint?: string;
}

interface RiskEnvelope {
  maxDailyLossQuote: number;
  maxConcurrent: number;
  pauseOn429Burst: boolean;
}

interface OfferDecision {
  publish: boolean;
  offers: CapacityOffer[];
  reason?: string;
}

function evaluateOffer(
  inputs: PricingInputs,
  risk: RiskEnvelope,
): OfferDecision;
```

## Data Flow

Input: provider capacity, upstream cost signals, health data, operator intent, witness history.  
Process: evaluate margins, apply risk guards, derive active offers, snapshot pricing terms.  
Output: market offers, pause decisions, deterministic pricing snapshots with quote-unit and settlement-asset metadata.

## State

- persistent: policy configuration, pricing snapshots, offer history
- memory: current health signals, recent rate-limit events, cooldown state

## Errors

- missing pricing input for an exposed model
- policy that would sell below configured floor
- stale health data preventing repricing
- inconsistent offer snapshot and witness version
- published offer missing quote-unit or settlement-asset hint

## Security Constraints

- pricing snapshots consumed by settlement must be deterministic and versioned
- risk policy must fail closed on unknown health or cost inputs
- offer publication must not bypass operator guardrails
- quote units must remain distinct from final settlement assets

## Test Requirements

- deterministic pricing for identical inputs
- pause behavior under degraded health
- repricing after cost change
- rejection when required pricing inputs are missing
- snapshot compatibility with witness consumers
- quote and settlement metadata consistency

## Dependencies

- calls: `metering-witness`, `provider-engine`, `claw-autopilot`
- called by: `provider-engine`, `claw-autopilot`, `settlement-payout`
