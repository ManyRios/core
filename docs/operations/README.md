# Operations

## Purpose

This section describes how Veil behaves as a running system: ports, state, persistence, observability, and release gates.

## Read This Section If

- you are operating Consumer, Provider, Relay, or Bootstrap services
- you need to know which state is persistent and which state is in memory
- you are deciding whether a runtime change is ready to ship

## Runtime Modes

- Consumer Gateway on `9960`
- Provider health endpoint on `9962`
- Relay on `8080`
- Bootstrap as an HTTP registry service

## Main Persistent Data

- Consumer: `wallet.json`, `config.json`
- Provider: `wallet.json`, `provider.json`
- Relay: `relay.db`, `witness.db`
- Bootstrap: Relay registry database
- Governance: RBOB ledger database

## Runtime State

- Consumer: pending requests, Provider list, Relay connection
- Provider: active request counter, metrics, Relay connections
- Relay: Provider map, request map, rate limiter
- Bootstrap: Relay cache and registry state

## Observability

- `health` endpoints
- structured logs
- witness stats and export
- Provider metrics

## Limits

- message size is bounded
- request age is bounded
- Provider concurrency is bounded
- Consumer budget and duration are bounded

## Release Gate

New runtime capability should only ship when it has:

- a clear module owner
- explicit failure semantics
- tests
- operational visibility

## Economic Readiness Checks

Before market-facing or settlement-facing changes are treated as ready:

- witness export output can be reproduced for sampled completed requests
- pricing snapshot references remain available for witness records that require deterministic settlement replay
- quote budgeting fields remain distinct from settlement asset fields in runtime outputs and exports
