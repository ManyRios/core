# Contributing to Veil Protocol

You write code. If it passes tests and gets merged, you earn future revenue share. That's the deal.

## 5-Minute Setup

```bash
git clone https://github.com/runveil-io/core.git
cd core
npm install
npm test
```

You should see `36/36 tests passing`. If you don't, you found your first contribution.

**Requirements**: Node.js 22+, npm

## How You Get Paid (RBOB)

Veil uses the **Rule-Based Open Build** protocol. Four rules:

1. **R1** — Code must pass all tests in CI
2. **R2** — Merge requires K independent approvals (currently K=1, Kousan)
3. **R3** — Protected modules (`crypto/`, `wallet/`) require higher threshold
4. **R4** — Surviving code earns points. Points = future revenue share.

**What "surviving" means**: your code stays in the codebase and passes tests. Dead code gets pruned, and points go with it.

**Genesis Bonus**: Early contributors earn 5x points. This compensates for the risk of building before there's revenue. The bonus decreases as the protocol matures.

**The economic loop**:
```
Your code → earns points → TGE → points convert to TOKEN → protocol revenue flows to TOKEN holders
```

Points are tracked in `contributions.db` with full git audit trail. No USDC payouts pre-TGE — points are your equity in the protocol's future.

## Finding Work

No separate task board. **The repo is the task board.**

### 1. Failing tests
```bash
npm test -- --reporter=verbose 2>&1 | grep FAIL
```
A failing test is a bounty. Fix it, PR it, earn points.

### 2. Read the code
The codebase is ~1500 lines. You can read all of `src/` in 30 minutes. If you spot something missing or broken, that's your contribution. Check `docs/design/00-architecture-review-v0.2.md` for the planned module structure — gaps between the doc and the code are real tasks.

### 3. GitHub Issues
Look for labels:
- `good-first-issue` — scoped, tested, documented
- `bounty` — has explicit point value
- `help-wanted` — bigger items that need ownership

### 4. Desired states
Check `desired/` directory (when populated) for feature specs with acceptance criteria and bounty values.

### 5. Let your agent do it
```bash
clawd build
```
This scans the repo for failing tests, TODOs, and desired states, picks one, writes code, runs tests, and opens a PR. You review and submit. Or set it to `auto`.

## Submitting a PR

Keep it simple:

1. **Fork & branch**: `git checkout -b fix/relay-timeout`
2. **Write code + tests**: if you touch `src/`, touch `tests/`
3. **Run tests locally**: `npm test` — all green
4. **Open PR** with:
   - What you changed (1-2 sentences)
   - Which issue/TODO it addresses (if any)
   - Test output showing pass

That's it. No issue template. No commit message convention. No CLA.

### PR Review

- CI runs automatically. Red CI = not reviewed.
- Currently Kousan reviews all PRs (CRL-1 stage).
- Expect review within 24-48 hours.
- Nits are suggestions, not blockers. Ship > perfect.

## What NOT to Touch Without Discussion

These modules are under R3 protection (higher review threshold):

- `src/crypto/` — envelope encryption, key generation
- `src/wallet/` — encrypted storage, KDF
- `RULES.md` — the rules themselves
- `contributions.db` — the points ledger

Open an issue first if you want to modify these. They affect security and economics.

## Code Style

- TypeScript strict mode
- No `any` unless you explain why in a comment
- Tests use vitest
- We don't enforce a formatter yet — just be consistent with surrounding code

## Tech Stack Reference

| Component | Library | Why |
|-----------|---------|-----|
| Runtime | Node.js 22+ | LTS, AI tooling ecosystem |
| Language | TypeScript 5.x | Strict mode, AI agents write it well |
| HTTP | Hono | 3KB, fast |
| WebSocket | ws | Standard |
| Crypto | tweetnacl | Pure JS, zero native deps, audited |
| DB | better-sqlite3 | WAL mode, zero config |
| Tests | vitest | Fast, good DX |
| Build | tsup | Single-file output |

## Architecture (30-second version)

```
Consumer (localhost:9960)
    → encrypts prompt with Provider's public key
    → sends to Relay over WebSocket
    
Relay (wss://relay.runveil.io)
    → verifies auth, strips identity
    → forwards encrypted blob to Provider
    
Provider (your machine)
    → decrypts prompt
    → calls AI API (OpenAI, Anthropic, etc.)
    → encrypts response, sends back
```

Relay sees **who** but not **what**. Provider sees **what** but not **who**.

## Questions?

- [Telegram](https://t.me/+XJ-ogZ9hBy44ZmFl) — fastest response
- GitHub Issues — for anything technical
- `clawd build` — if you'd rather let your agent ask for you

---

*Your code survives → you earn. That's the only rule that matters.*
