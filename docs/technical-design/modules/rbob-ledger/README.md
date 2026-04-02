# RBOB Ledger

## Purpose

This module records open-source contribution grants and exposes auditable ledger queries.

## Responsibility Boundary

- store contribution points
- sign ledger entries with an admin key
- verify ledger entries
- export balances and leaderboard data

## Out Of Scope

- does not participate in inference routing
- does not replace production witness records
- does not manage contributor identity beyond ledger needs

## Interface

```ts
class RbobLedger {
  grant(
    params: GrantParams,
    adminSecretKey: Uint8Array,
    adminPublicKey: Uint8Array,
  ): LedgerEntry;
  verifyEntry(entry: LedgerEntry, adminPublicKey: Uint8Array): boolean;
  balance(contributor: string): BalanceResult;
  leaderboard(limit?: number): LeaderboardEntry[];
  exportJSON(): TokenClaimExport;
}
```

## Data Flow

Input: contributor identity, points, reason, PR reference.  
Process: validate grant, sign message, insert ledger entry, query totals.  
Output: signed contribution records and export payloads.

## State

- persistent: `rbob_ledger`
- memory: query results and export objects

## Errors

- non-positive points
- missing contributor
- missing reason
- invalid signature verification

## Security Constraints

- admin signatures are the trust anchor
- contribution accounting must stay separate from production witness accounting

## Test Requirements

- grant, verify, balance, leaderboard, export
- invalid grant rejection

## Dependencies

- calls: `crypto`
- called by: `cli`, maintainers
