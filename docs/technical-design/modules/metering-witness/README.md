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

---

## Implementation Details

**Source:** `src/metering/index.ts`, `src/metering/types.ts`, `src/metering/normalize.ts`, `src/metering/pricing.ts`, `src/metering/witness.ts`

### Target Interface (from Architecture)

```ts
// Architecture-defined types (authoritative)
type QuoteUnit = 'usd_estimate';

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

class WitnessStore {
  record(...): WitnessRecord;
  verify(...): boolean;
  export(...): WitnessRecord[];
}

function buildQuoteEstimate(usage: NormalizedUsage, pricing: PricingSnapshot): QuoteEstimate;
```

### Current Code Structures

```ts
// src/metering/types.ts — NormalizedUsage matches architecture ✅
interface NormalizedUsage {
  input: number;
  output: number;
  cacheRead?: number;
  cacheWrite?: number;
}

// src/metering/types.ts — implementation-specific (no architecture equivalent)
interface PriceConfig {
  model: string;
  inputPerM: number;      // USD per 1M input tokens
  outputPerM: number;
  cacheReadPerM?: number;
  cacheWritePerM?: number;
}

interface CostBreakdown {  // ⚠️ no quoteUnit; architecture uses QuoteEstimate instead
  inputCost: number;
  outputCost: number;
  cacheReadCost?: number;
  cacheWriteCost?: number;
  totalCost: number;
}

// src/relay/witness.ts — ⚠️ NEEDS ALIGNMENT: snake_case, flat tokens, missing architecture fields
interface WitnessRecord {      // relay-layer version
  request_id: string;          // ⚠️ should be requestId
  consumer_pubkey: string;
  provider_pubkey: string;     // ⚠️ should be providerId
  model: string;
  input_tokens: number;        // ⚠️ should be usage: NormalizedUsage
  output_tokens: number;
  cache_read_tokens?: number;
  cache_write_tokens?: number;
  duration_ms: number;
  timestamp: number;
  relay_pubkey: string;        // ⚠️ should be relayId
  relay_signature: string;     // ⚠️ should be relaySignature
  // MISSING: pricingVersion, quoteUnit, completionStatus, evidenceHash, dedupeKey
}

// Provider-specific usage formats (implementation detail, not in architecture)
interface AnthropicUsage { input_tokens; output_tokens; cache_read_input_tokens?; cache_creation_input_tokens? }
interface OpenAIUsage { prompt_tokens; completion_tokens; prompt_tokens_details?.cached_tokens? }
interface GoogleUsage { promptTokenCount; candidatesTokenCount; cachedContentTokenCount? }
type AnyUsage = AnthropicUsage | OpenAIUsage | GoogleUsage | Record<string, unknown>
```

### Usage Normalization (`src/metering/normalize.ts`)

- Type guards: `isAnthropic()`, `isOpenAI()`, `isGoogle()` detect format by field presence
- `normalizeUsage(usage: AnyUsage): NormalizedUsage` — maps provider-specific fields to unified format
- `normalizeUsageBatch()` — batch normalization
- `aggregateUsage()` — sum multiple NormalizedUsage records
- Fallback: tries generic field mapping (`input_tokens` / `prompt_tokens` / `promptTokenCount`)

### Pricing (`src/metering/pricing.ts`)

- `DEFAULT_PRICE_TABLE`: 8 models (claude-3-opus/sonnet/haiku, gpt-4/4-turbo/3.5-turbo, gemini-pro/ultra)
- `calculateCost(usage, priceConfig)`: `(tokens × pricePerM) / 1_000_000` per component
- `calculateCostByModel(usage, model, priceTable?)`: model lookup + cost calculation. Throws on unknown model.
- `formatCost()`: `$X.XXXXXX` format
- `formatCostBreakdown()`: human-readable `Input: $X | Output: $X | Total: $X`

### Witness Generation (`src/metering/witness.ts`)

- `generateWitness()`: creates witness record, signs with HMAC-SHA256 (Web Crypto API)
- `verifyWitness()`: verifies HMAC signature
- `generateWitnessBatch()`: parallel witness generation
- `serializeWitness()` / `deserializeWitness()`: JSON serialization
- **Note**: This is the metering-layer witness (HMAC-based). The relay-layer `WitnessStore` uses Ed25519 signatures. Architecture defines a unified `WitnessRecord` with camelCase fields, `pricingVersion`, `quoteUnit`, `completionStatus`, `evidenceHash`, `dedupeKey` — neither implementation matches this yet.

### Error Handling

- Unknown model in `calculateCostByModel()` → throws with available models list
- Budget tracker (consumer-side) falls back to expensive default ($30/$60 per 1M) on unknown model

## API Specification

### Normalization

```ts
normalizeUsage(usage: AnyUsage): NormalizedUsage
normalizeUsageBatch(usageRecords: AnyUsage[]): NormalizedUsage[]
aggregateUsage(usageRecords: NormalizedUsage[]): NormalizedUsage
```

### Pricing

```ts
calculateCost(usage: NormalizedUsage, priceConfig: PriceConfig): CostBreakdown
calculateCostByModel(usage: NormalizedUsage, model: string, priceTable?: Record<string, PriceConfig>): CostBreakdown
calculateCostBatch(usageRecords: NormalizedUsage[], priceConfig: PriceConfig): CostBreakdown[]
formatCost(cost: number): string
formatCostBreakdown(breakdown: CostBreakdown): string
```

### Witness

```ts
generateWitness(requestId: string, usage: NormalizedUsage, cost: CostBreakdown, relayPrivateKey: string): Promise<Witness>
verifyWitness(witness: Witness, relayPublicKey: string): Promise<boolean>
generateWitnessBatch(records: Array<{requestId, usage, cost}>, relayPrivateKey: string): Promise<Witness[]>
serializeWitness(witness: Witness): string
deserializeWitness(data: string): Witness
```

## Integration Protocol

- **Called by Consumer (`budget.ts`)**: `calculateCostByModel()` for per-request budget estimation
- **Called by Relay (`witness.ts`)**: separate Ed25519-based witness system (not using metering witness)
- **Exports**: all types, normalization, pricing, and witness functions via `src/metering/index.ts` barrel
- **No external dependencies**: pure computation module, no DB or network calls

## Current Implementation Status

- ✅ `NormalizedUsage` interface [IMPLEMENTED] — matches architecture
- ✅ `normalizeUsage()` function [IMPLEMENTED]
- ✅ Multi-provider usage normalization (Anthropic, OpenAI, Google + fallback) [IMPLEMENTED]
- ✅ Batch normalization and aggregation [IMPLEMENTED]
- ✅ Per-model pricing with cache token support [IMPLEMENTED]
- ✅ Cost formatting utilities [IMPLEMENTED]
- ⚠️ Metering witness (HMAC-based) [IMPLEMENTED · NEEDS ALIGNMENT] — uses HMAC-SHA256 instead of Ed25519; does not produce architecture-defined `WitnessRecord` fields (`pricingVersion`, `quoteUnit`, `completionStatus`, `evidenceHash`, `dedupeKey`)
- ⚠️ Relay witness (`src/relay/witness.ts`) [IMPLEMENTED · NEEDS ALIGNMENT] — uses snake_case field names, flat token counts instead of `NormalizedUsage`, missing `pricingVersion`/`quoteUnit`/`completionStatus`/`evidenceHash`/`dedupeKey`
- ⚠️ Price table is static (8 models), not versioned [IMPLEMENTED · NEEDS ALIGNMENT]
- ❌ `PricingSnapshot` type with `version` + `quoteUnit` [NOT IMPLEMENTED]
- ❌ `QuoteEstimate` type [NOT IMPLEMENTED]
- ❌ `buildQuoteEstimate()` function [NOT IMPLEMENTED]
- ❌ `WitnessStore` class (architecture-defined: `record()`, `verify()`, `export()`) [NOT IMPLEMENTED] — relay has a `WitnessStore` class but with different interface
- ❌ Deterministic pricing snapshots with versioning [NOT IMPLEMENTED]
- ❌ Quote-unit / settlement-asset separation [NOT IMPLEMENTED]

## Code Alignment Tasks

- [ ] Create `PricingSnapshot` type with `version`, `model`, `quoteUnit` fields
- [ ] Create `QuoteEstimate` type with `total`, `quoteUnit` fields
- [ ] Implement `buildQuoteEstimate(usage, pricing)` function
- [ ] Unify witness record format: rename relay `WitnessRecord` fields to camelCase (`request_id` → `requestId`, `provider_pubkey` → `providerId`, `relay_pubkey` → `relayId`, `relay_signature` → `relaySignature`)
- [ ] Replace flat token fields (`input_tokens`, `output_tokens`, etc.) with `usage: NormalizedUsage` in `WitnessRecord`
- [ ] Add `pricingVersion`, `quoteUnit`, `completionStatus`, `evidenceHash`, `dedupeKey` to `WitnessRecord`
- [ ] Align `WitnessStore` class interface to architecture: `record()`, `verify()`, `export()`
- [ ] Add version string to price table for `PricingSnapshot` support
- [ ] Decide whether to unify metering-layer HMAC witness with relay-layer Ed25519 witness into single architecture-defined `WitnessRecord`

---

## Design Specifications for Unimplemented Items

### Deterministic Pricing Snapshots with Versioning [DESIGN SPEC · Phase 4/5]

```ts
// Extends existing PricingSnapshot in metering module
interface VersionedPricingSnapshot extends PricingSnapshot {
  version: string;               // sha256(model + inputPerM + outputPerM + bucketId)
  bucketId: string;              // floor(timestamp / 300000) — 5-min bucket
  cacheReadPerM?: number;
  createdAt: number;
}

// Integration with existing calculateCost():
// - calculateCostByModel() currently looks up DEFAULT_PRICE_TABLE
// - Future: also writes a VersionedPricingSnapshot on each price table change
// - WitnessStore.record() captures snapshot.version in witness record
// - Settlement resolves pricing by version, not by current table
//
// Migration: existing witnesses without pricingVersion get version='legacy-v0'
```

### Quote-Unit / Settlement-Asset Separation [DESIGN SPEC · Phase 5]

```ts
// Current state: all amounts are in 'usd_estimate' quote units
// Future: amounts stay in quote units through metering/witness pipeline
// Conversion to settlement asset happens ONLY in settlement-payout module

// Changes to WitnessRecord:
interface WitnessRecordV2 extends WitnessRecord {
  quoteUnit: QuoteUnit;            // explicit on every record (not assumed)
  pricingVersion: string;          // links to VersionedPricingSnapshot
  // settlementAsset is NOT stored here — settlement module resolves it
}

// Rules:
// - Metering pipeline only speaks quote units
// - No settlement-asset references in metering or witness code
// - Budget tracker compares in quote units
// - Cost breakdown reports quote unit explicitly
// - Settlement-payout module owns the quote→asset conversion
```
