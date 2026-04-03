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

---

## Implementation Details

**Source:** `src/metering/pricing.ts` (partial overlap with metering module)

### Current Implementation

Pricing logic currently lives in the metering module (`src/metering/pricing.ts`). There is no standalone pricing-risk-policy module in `src/`. The design doc describes a future module boundary.

### What Exists in Code

```ts
// src/metering/pricing.ts
const DEFAULT_PRICE_TABLE: Record<string, PriceConfig> = {
  'claude-3-opus':    { inputPerM: 15,   outputPerM: 75   },
  'claude-3-sonnet':  { inputPerM: 3,    outputPerM: 15   },
  'claude-3-haiku':   { inputPerM: 0.25, outputPerM: 1.25 },
  'gpt-4-turbo':      { inputPerM: 10,   outputPerM: 30   },
  'gpt-4':            { inputPerM: 30,   outputPerM: 60   },
  'gpt-3.5-turbo':    { inputPerM: 0.5,  outputPerM: 1.5  },
  'gemini-pro':       { inputPerM: 0.5,  outputPerM: 1.5  },
  'gemini-ultra':     { inputPerM: 7,    outputPerM: 21   },
};

calculateCost(usage: NormalizedUsage, priceConfig: PriceConfig): CostBreakdown
calculateCostByModel(usage: NormalizedUsage, model: string): CostBreakdown
```

### What Does NOT Exist in Code

- `CapacityOffer`, `RiskEnvelope`, `OfferDecision` types
- `evaluateOffer()` function
- Pricing snapshots with versioning
- Sell-side policy enforcement
- Margin calculation
- Pause/resume on health degradation
- Quote-unit / settlement-asset distinction

## API Specification

See metering module for the implemented pricing functions. The dedicated pricing-risk-policy API surface defined in the architecture section is not yet implemented.

## Integration Protocol

- Currently: `calculateCostByModel()` is called by consumer budget tracker
- Future: pricing snapshots would feed into witness records and settlement

## Current Implementation Status

- ✅ Static price table with per-model pricing [IMPLEMENTED] (in metering module)
- ✅ Cost calculation with cache token support [IMPLEMENTED] (in metering module)
- ❌ CapacityOffer / RiskEnvelope types [DESIGN ONLY]
- ❌ evaluateOffer() with margin and risk guards [DESIGN ONLY]
- ❌ Deterministic pricing snapshots [DESIGN ONLY]
- ❌ Sell-side pause/resume policy [DESIGN ONLY]
- ❌ Quote-unit to settlement-asset mapping [DESIGN ONLY]
