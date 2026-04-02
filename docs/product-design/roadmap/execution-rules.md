# Execution Rules

## Purpose

This document defines how Veil should sequence delivery so the roadmap can be executed strictly instead of fragmenting into unsynchronized parallel workstreams.

## Planning Unit

Every roadmap item should be mapped to one of these units:

- `Foundation`: runtime behavior that must work before automation can depend on it
- `Control`: discovery, policy, and health systems that coordinate the market
- `Market`: capacity publication, pricing, and sell-side automation
- `Settlement`: witness-backed accounting and payout-ready outputs
- `Privacy`: baseline privacy-preserving routing is required; stronger privacy profiles can be staged without blocking the standard market loop

## Hard Dependency Order

1. runtime baseline before autopilot
2. discovery and health before automated selling
3. pricing and risk policy before autonomous market publication
4. witness integrity and pricing snapshots before payout logic
5. standard marketplace operation before any anonymity-heavy profile

## Phase Gate Rules

### Runtime Baseline Gate

Must exist before later phases depend on it:

- stable Consumer to Relay to Provider request path
- reproducible witness generation
- bounded retry, timeout, and rate-limit behavior
- operator-visible health and logs

### Discovery Gate

Must exist before low-touch automation starts:

- Bootstrap returns usable Relay sets
- Provider can reconnect across more than one Relay candidate
- local runtime can survive relay loss without manual rebuild

### Autopilot Gate

Must exist before sell-side automation is exposed:

- operator intent can be persisted
- join flow can validate prerequisites before exposing capacity
- health degradation can pause or resume capacity automatically
- automation cannot bypass existing security checks

### Market Control Gate

Must exist before autonomous selling is called complete:

- pricing policy is deterministic
- risk envelopes can block unsafe selling
- offer publication and withdrawal are traceable
- repricing does not invalidate witness accounting

### Settlement Gate

Must exist before payout surfaces are exposed:

- settlement entries are reproducible from witness plus pricing
- duplicate settlement protection exists
- payout outputs remain separate from contributor accounting
- export flows are auditable

## Definition Of Done For A Roadmap Capability

Every roadmap capability should ship with:

- an explicit owner module
- interface and state definition
- failure handling rules
- security constraints
- test cases
- operator-visible status or logs
- documentation updates

## Red-Line Rules

- do not ship sell-side automation before policy and health gates exist
- do not ship payout workflows before deterministic settlement inputs exist
- do not let privacy claims outrun deployed guarantees
- do not collapse RBOB and market settlement into one ledger

## Recommended Breakdown Order

1. align docs and modules
2. build missing foundation code
3. add Claw-managed join flow
4. add pricing and risk policy
5. add settlement and payout ledgering
6. add near-autonomous market operation
7. only then evaluate a stronger privacy profile
