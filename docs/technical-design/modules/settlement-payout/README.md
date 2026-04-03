# Settlement & Payout

## Purpose

This module defines how witness-backed usage evidence and deterministic quote snapshots become settlement-ready accounting records.

## Responsibility Boundary

- consume witness records and pricing snapshots
- derive deterministic settlement entries
- generate payable balances for Provider and Relay roles
- expose exportable payout instructions without mixing them into contributor accounting

## Out Of Scope

- does not route traffic
- does not invent usage without witness evidence
- does not own contributor grants
- does not force a specific payment rail implementation

## Interface

```ts
type QuoteUnit = 'usd_estimate';

interface SettlementEntry {
  requestId: string;
  pricingVersion: string;
  quoteUnit: QuoteUnit;
  providerQuoteAmount: number;
  relayQuoteAmount: number;
  feesQuoteAmount: number;
  settlementAsset?: string;
  evidenceHash: string;
  dedupeKey: string;
  status: 'quoted' | 'ready' | 'exported';
}

interface PayoutInstruction {
  recipient: string;
  quoteAmount: number;
  settlementAsset: string;
  reason: 'provider' | 'relay';
}

function settleWitness(
  witness: WitnessRecord,
  pricing: PricingSnapshot,
): SettlementEntry;

function buildPayouts(
  entries: SettlementEntry[],
  settlementAsset: string,
): PayoutInstruction[];
```

## Data Flow

Input: witness records, pricing snapshots, role identities, payout windows, settlement-asset mapping.  
Process: validate evidence, compute settlement entries, aggregate payable balances, resolve settlement asset, export payout instructions.  
Output: settlement ledger entries, balances, payout-ready exports.

## State

- persistent: settlement ledger, payout batches, export history
- memory: current aggregation windows, validation caches

## Errors

- missing witness for a requested settlement
- missing pricing snapshot version
- invalid witness signature
- duplicate settlement entry
- missing settlement-asset mapping for export
- payout export mismatch

## Security Constraints

- settlement must never trust opaque side-channel usage
- payout instructions must be reproducible from witness and pricing inputs
- contributor accounting and market settlement must stay on separate ledgers
- quote amounts and settlement assets must remain explicitly distinguishable

## Test Requirements

- deterministic settlement entry generation
- duplicate protection
- invalid witness rejection
- payout aggregation by role
- export reproducibility
- quote-to-settlement export consistency

## Dependencies

- calls: `metering-witness`, `pricing-risk-policy`
- called by: `cli`, `claw-autopilot`

---

## Implementation Details

**Source:** No implementation exists.

## API Specification

No code. See architecture section above for planned interface.

## Integration Protocol

No code. Planned to consume witness records from `relay/witness.ts` WitnessStore and pricing snapshots from a future pricing-risk-policy module.

## Current Implementation Status

- ❌ `settleWitness()` [DESIGN ONLY]
- ❌ `buildPayouts()` [DESIGN ONLY]
- ❌ Settlement ledger [DESIGN ONLY]
- ❌ Payout instruction export [DESIGN ONLY]
- ❌ Deduplication / evidence hash verification [DESIGN ONLY]
- ❌ Quote-to-settlement asset mapping [DESIGN ONLY]

This module is planned for **Phase 5** (Witness-Backed Settlement). No source code exists in `src/`.
