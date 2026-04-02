# Governance and Economics

## Purpose

This section explains how Veil handles both sides of its economic design:

- the inference market loop between Consumers, Providers, and Relays
- the open-source build loop around maintainers, contributors, and RBOB

## Separation Rule

Veil keeps inference traffic and contribution accounting in separate domains.

## Current Runtime Boundary

- the current runtime already exposes quote-based budget guards, usage normalization, signed witness records, exportable audit data, and contribution accounting
- the current runtime does not yet expose end-user payment rails, Provider payout flows, or Relay revenue distribution APIs

## Quote Language vs Settlement Asset

- the runtime may use quote units for budgeting and deterministic price comparison; the current quote language is USD-estimated
- quote units are not the final settlement asset
- crypto-native payment and settlement remain on the main product path even while quote-driven budgeting exists first

## Inference Market Direction

- Veil is being built toward a market loop where Consumers pay for usable AI access through the Veil path
- Providers earn for executed inference capacity
- Relays earn for routing, admission, and witness services
- witness-backed records exist so usage and settlement can be reconciled later

## Settlement Direction

- settlement should be derived from request and witness evidence, not from opaque side channels
- pricing and settlement may evolve independently from routing transport
- quote amounts and settlement assets must remain explicitly distinguishable in interfaces and exports
- future payment rails should consume the same witness and accounting boundaries, not bypass them

## Governance Side

- maintainers review and merge code
- RBOB records contribution points
- protected modules require stricter review

## Community Rule

Veil is a public open-source project. Contributors, reviewers, and maintainers should remain visible parts of the project, and contribution records should stay attributable and auditable.

## RBOB Role

RBOB is a contribution ledger. It is not part of the inference hot path. Its job is to record signed contribution entries, balances, leaderboards, and exportable grant records.

## Accounting Rules

- production witness data must stay separate from contribution points
- inference revenue and contributor rewards must remain separable at the data model level
- contribution grants must be signed
- governance data must be auditable and exportable
