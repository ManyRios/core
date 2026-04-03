# Claw Autopilot

## Purpose

This module defines the target automation boundary that turns operator intent into low-touch node operation.

## Responsibility Boundary

- capture join-network and sell-capacity intent
- validate local prerequisites before a node is exposed to the network
- orchestrate wallet, discovery, provider, and relay startup flows
- reconcile pricing policy, health status, pause and resume behavior
- persist operator intent and automation state

## Out Of Scope

- does not proxy inference traffic directly
- does not decrypt request payloads
- does not replace Relay admission logic
- does not invent settlement outcomes without witness input

## Interface

```ts
interface OperatorIntent {
  mode: 'join-network' | 'sell-capacity' | 'pause-market';
  providerMode: 'api-capacity' | 'self-hosted';
  relayMode: 'bootstrap' | 'static' | 'manual';
}

interface AutopilotPolicy {
  maxDailyLossQuote: number;
  minMarginPct: number;
  autoPauseOnHealthDegrade: boolean;
}

interface AutopilotRuntime {
  close(): Promise<void>;
  reconcile(): Promise<void>;
}

function startAutopilot(
  intent: OperatorIntent,
  policy: AutopilotPolicy,
): Promise<AutopilotRuntime>;
```

## Data Flow

Input: operator intent, local credentials, discovery results, Provider health, witness and pricing signals.  
Process: validate, plan desired runtime state, start or reconfigure nodes, monitor drift, reconcile changes.  
Output: running node state, policy actions, pause or resume decisions, automation logs.

## State

- persistent: operator intent, policy snapshots, last known relay choice, last known provider offer state
- memory: reconciliation loop state, health snapshots, pending actions, cooldown timers

## Errors

- missing credentials for selected provider mode
- no reachable Relay candidates
- policy conflict that would expose capacity unsafely
- failed start or reconcile action
- repeated health degradation causing forced pause

## Security Constraints

- operator credentials must never be exposed outside local secret boundaries
- autopilot must fail closed when policy or health signals are ambiguous
- automation must not bypass Relay, Provider, or settlement security checks

## Test Requirements

- join flow with valid prerequisites
- sell-capacity flow with policy application
- pause and resume on health degradation
- recovery after relay loss
- no-start behavior when secrets or policy requirements are missing

## Dependencies

- calls: `wallet-identity`, `bootstrap-discovery`, `provider-engine`, `relay`, `pricing-risk-policy`, `settlement-payout`
- called by: `cli`

---

## Implementation Details

**Source:** No implementation exists.

## API Specification

No code. See architecture section above for planned interface.

## Integration Protocol

No code. Planned to orchestrate wallet, discovery, provider, and relay modules. CLI would expose `veil autopilot init/join/show` commands.

## Current Implementation Status

- ❌ `startAutopilot()` [DESIGN ONLY]
- ❌ OperatorIntent / AutopilotPolicy types [DESIGN ONLY]
- ❌ Join-network flow [DESIGN ONLY]
- ❌ Sell-capacity with policy enforcement [DESIGN ONLY]
- ❌ Health-based pause/resume [DESIGN ONLY]
- ❌ Reconciliation loop [DESIGN ONLY]
- ❌ CLI commands (`veil autopilot init/join/show`) [DESIGN ONLY]

This module is planned for **Phase 3** (Guided Operator Automation) and **Phase 6** (Low-Touch Market Operation). No source code exists in `src/`.
