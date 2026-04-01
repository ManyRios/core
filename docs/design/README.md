# Veil Protocol — Architecture & Module Design

## Implementation Status

```
veil-core/
│
├── src/
│   ├── crypto/          ✅ Implemented   → 04-crypto-envelope.md
│   │   └── index.ts     69 lines: seal/open/sign/verify/sha256
│   │
│   ├── consumer/        ✅ Implemented   → 05-consumer-gateway.md
│   │   ├── index.ts     423 lines: OpenAI-compatible HTTP gateway
│   │   └── anthropic-stream.ts  37 lines: SSE stream adapter
│   │   └── selector.ts  ❌ NOT IMPLEMENTED [GAP] Provider selection
│   │
│   ├── provider/        ✅ Implemented   → 06-provider-engine.md
│   │   └── index.ts     397 lines: decrypt + Anthropic API call + retry
│   │   └── engine.ts    ❌ NOT IMPLEMENTED [GAP] multi-vendor adapter
│   │   └── accounts.ts  ❌ NOT IMPLEMENTED [GAP] multi-account pool
│   │
│   ├── relay/           ✅ Implemented   → 07-relay-routing.md
│   │   └── index.ts     375 lines: auth + blind forward + witness
│   │   └── auth.ts      ❌ NOT IMPLEMENTED [GAP] balance check
│   │
│   ├── network/         ✅ Implemented   → 08-network-transport.md
│   │   └── index.ts     169 lines: WebSocket + basic reconnect
│   │
│   ├── wallet/          ✅ Implemented   → 09-wallet-identity.md
│   │   └── index.ts     170 lines: scrypt + AES-256-GCM encrypted storage
│   │
│   ├── metering/        ❌ NOT IMPLEMENTED → 03-metering-billing.md
│   │   └── normalize.ts [GAP] usage normalization
│   │   └── pricing.ts   [GAP] cost calculation
│   │   └── witness.ts   [GAP] settlement witness
│   │
│   ├── cli.ts           ✅ Implemented   → 10-cli-ux.md
│   │                    289 lines: init/provide/relay/status
│   │
│   ├── types.ts         ✅ Implemented   → 11-wire-protocol.md
│   │                    138 lines: 14 message types
│   │
│   ├── db.ts            ✅ Implemented
│   │                    63 lines: SQLite schema
│   │
│   └── config/
│       └── bootstrap.ts ✅ Implemented
│                        27 lines: hardcoded relay pubkey + models
│
├── tests/               ✅ 36/36 passing
│   ├── consumer.test.ts
│   ├── crypto.test.ts
│   ├── e2e.test.ts
│   ├── network.test.ts
│   ├── provider.test.ts
│   ├── relay.test.ts
│   └── wallet.test.ts
│
├── desired/             ✅ 10 tasks for AI agents
│
└── docs/design/         ✅ 14 design documents (this directory)
```

## Design Documents

### Architecture (top-level)
| # | Document | Status | Lines |
|---|----------|--------|-------|
| 00 | [Architecture Review v0.2](00-architecture-review-v0.2.md) | ✅ Baseline | 153 |

### Core Modules
| # | Module | Document | Code | Gaps |
|---|--------|----------|------|------|
| 02 | Security | [02-security-threat-model.md](02-security-threat-model.md) | n/a | Needs expansion |
| 03 | Metering | [03-metering-billing.md](03-metering-billing.md) | ❌ No code yet | Full module missing |
| 04 | Crypto | [04-crypto-envelope.md](04-crypto-envelope.md) | ✅ 69 lines | Input validation, memory zeroize |
| 05 | Consumer | [05-consumer-gateway.md](05-consumer-gateway.md) | ✅ 460 lines | Provider selector, retry, rate limit |
| 06 | Provider | [06-provider-engine.md](06-provider-engine.md) | ✅ 397 lines | Multi-vendor, account pool, usage tracking |
| 07 | Relay | [07-relay-routing.md](07-relay-routing.md) | ✅ 375 lines | Balance check, load balancing |
| 08 | Network | [08-network-transport.md](08-network-transport.md) | ✅ 169 lines | Backoff jitter, state machine, max retries |
| 09 | Wallet | [09-wallet-identity.md](09-wallet-identity.md) | ✅ 170 lines | Backup/restore, key rotation |
| 10 | CLI | [10-cli-ux.md](10-cli-ux.md) | ✅ 289 lines | Stop commands, colors, balance, build |

### Cross-Cutting Concerns
| # | Topic | Document | Code | Gaps |
|---|-------|----------|------|------|
| 11 | Wire Protocol | [11-wire-protocol.md](11-wire-protocol.md) | ✅ types.ts | Version negotiation |
| 12 | Anti-Detection | [12-anti-detection.md](12-anti-detection.md) | ❌ No code yet | Full module missing |
| 13 | RBOB Scoring | [13-rbob-scoring.md](13-rbob-scoring.md) | ❌ No code yet | Full module missing |

### Legend
- ✅ Implemented — code exists and tests pass
- ❌ NOT IMPLEMENTED — design exists, code does not
- [GAP] — feature described in design doc but missing from current code

## How to Use These Docs

**As a human**: Read the module doc before touching that module's code.

**As an AI agent**: Parse the design doc for interfaces, then implement against the acceptance criteria.

```bash
# Find all gaps in the codebase
grep -r '\[GAP\]' docs/design/*.md
```
