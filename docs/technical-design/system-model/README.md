# System Model

## Purpose

This section explains how requests move through Veil and what each runtime role can see or control.

## Read This Section If

- you need the end-to-end request path
- you are checking trust or visibility boundaries
- you want to understand which role owns which runtime responsibility

## Roles

The currently deployed runtime roles are Consumer, Relay, Provider, and Bootstrap. Claw is the target operator automation role on the main marketplace path, not an optional side tool.

### Consumer

- exposes a local HTTP gateway
- signs and encrypts requests
- enforces local API auth, budget, and timeout

### Relay

- accepts Consumer and Provider connections
- verifies request signatures
- applies rate limits
- routes requests to Providers
- records witness data

### Provider

- decrypts sealed request payloads
- calls upstream AI APIs
- returns single or streaming responses
- tracks local concurrency and credentials

### Bootstrap

- stores Relay registrations and heartbeats
- returns scored Relay lists to clients

### Claw

- captures operator intent such as join-network and sell-capacity
- automates node onboarding and runtime startup
- applies pricing and risk policy
- reconciles health, pause, resume, and recovery workflows

## Access Model

- Consumers access AI through the local gateway instead of integrating every tool directly with every upstream account
- Providers contribute routable execution capacity to the network path
- Relays broker supply and witness while staying outside the decryption boundary
- Claw reduces manual node operations so Provider capacity can be published and maintained with low-touch automation

## Request Path

1. Consumer receives an OpenAI-compatible request.
2. Consumer selects a Provider and seals the inner payload.
3. Relay verifies and forwards the request.
4. Provider decrypts and calls the upstream model.
5. Relay forwards the result and records witness data.
6. Consumer converts the result back into client-facing output.

## Operator Path

1. Operator expresses intent to join the network or sell capacity.
2. Claw validates local prerequisites, credentials, and policy guardrails.
3. Claw discovers or selects Relay infrastructure.
4. Claw starts or reconciles Provider runtime state.
5. Claw keeps pricing, capacity publication, and health behavior aligned with operator policy.
6. Witness and settlement systems consume the resulting usage evidence, pricing versions, and quote metadata.

## Visibility Boundary

| Role | Can See |
|------|---------|
| Consumer | local prompts, local responses, local wallet, local budget |
| Relay | request metadata, routing metadata, witness metadata |
| Provider | prompt plaintext, model parameters, upstream usage |
| Bootstrap | Relay metadata only |
| Claw | operator intent, local runtime state, policy state, and health outcomes |

Relay should not access prompt plaintext. Provider should not receive the original Consumer identity context beyond what execution requires.

## Privacy Position

Veil is designed for accountless access and privacy-preserving routing. That means visibility is split across roles. It does not mean Veil can guarantee perfect anonymity against traffic analysis, endpoint compromise, upstream operator visibility, or colluding nodes.
