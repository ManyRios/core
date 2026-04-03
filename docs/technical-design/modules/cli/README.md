# CLI

## Purpose

This module provides the operator-facing command surface for local setup, runtime control, and RBOB queries.

Today the CLI is the primary runtime entrypoint. Over time it should also expose the supported Claw automation surface instead of forcing all operator work through manual subcommands.
The CLI is a transition surface for `Automation`: manual commands should progressively map to supported Claw workflows for join, publish, pause, and recovery.

## Responsibility Boundary

- initialize wallets
- configure and start Consumer, Provider, and Relay roles
- render runtime status
- expose RBOB ledger commands

## Out Of Scope

- does not own protocol logic
- does not store long-lived runtime state by itself
- does not replace runtime health or witness storage modules

## Interface

```bash
veil init
veil provide init
veil provide start
veil relay start
veil status
veil rbob balance
veil rbob leaderboard
```

Target extension:

```bash
veil autopilot init
veil autopilot join
veil autopilot show
```

## Data Flow

Input: CLI args, passwords, local config.  
Process: call runtime modules, validate prerequisites, format output.  
Output: human-readable status and actionable errors.

## State

- memory: current command state, spinner, formatted tables
- persistence: indirect via wallet files, Provider config, Relay DB, RBOB DB

## Errors

- missing wallet
- wrong password
- invalid Provider config
- upstream validation failure

## Security Constraints

- passwords must not be passed through argv
- output must not expose secrets

## Test Requirements

- init flow
- provide init flow
- status output
- RBOB queries

## Dependencies

- calls: `wallet`, `consumer`, `provider`, `relay`, `rbob`
- called by: operators and contributors

---

## Implementation Details

**Source:** `src/cli.ts`

### Command Structure

Single-file CLI (~600 lines) using `process.argv` parsing (no framework). Command dispatch via `switch` on `args[0]` + `args[1]`.

### Implemented Commands

| Command | Function | Description |
|---------|----------|-------------|
| `veil init` | `cmdInit()` | Create wallet + config |
| `veil provide init` | `cmdProvideInit()` | Validate API key, encrypt, save provider.json |
| `veil provide start` | `cmdProvideStart()` | Load wallet, decrypt keys, start provider |
| `veil relay start` | `cmdRelayStart()` | Load wallet, start relay server |
| `veil use start` | `cmdUseStart()` | Start consumer gateway |
| `veil status` | `cmdStatus()` | Check gateway, relay, provider status |
| `veil relays` | `cmdRelays()` | List relays from bootstrap |
| `veil rbob balance <id>` | `cmdRbobBalance()` | Show contributor balance + history |
| `veil rbob leaderboard` | `cmdRbobLeaderboard()` | Top 20 contributors |
| `veil rbob grant` | `cmdRbobGrant()` | Grant points (admin) |
| `veil rbob export` | `cmdRbobExport()` | Export ledger JSON |
| `veil witness <id>` | `cmdWitnessGet()` | Lookup witness record |
| `veil witness stats` | `cmdWitnessStats()` | Aggregate witness stats |
| `veil witness export` | `cmdWitnessExport()` | Export witness records |
| `veil wallet export` | `cmdWalletExport()` | Export encrypted wallet |
| `veil wallet change-password` | `cmdWalletChangePassword()` | Change wallet password |
| `veil wallet migrate` | `cmdWalletMigrate()` | Migrate legacy wallet |

### UX Utilities

- **Colors**: `NO_COLOR` / `FORCE_COLOR` env support, `styleText()` from `node:util`
- **Spinner**: Unicode braille animation (80ms frames), non-TTY fallback
- **Table formatter**: auto-column-width with ANSI code stripping
- **Password prompt**: `readline` interface, `VEIL_PASSWORD` env var for non-interactive use

### Config Validation

- `loadAndValidateConfig()`: validates `config.json` fields (relay_url, gateway_port, consumer_pubkey, encryption_pubkey)
- `loadAndValidateProviderConfig()`: validates `provider.json` fields (version, models, api_keys, max_concurrent)
- Both use `ConfigValidationResult` with detailed per-field error messages

### Discovery Mode Support

- Reads `relay_discovery` from config: `'bootstrap'` | `'static'` | `'manual'`
- Creates `RelayDiscoveryClient` when mode is `'bootstrap'`
- Passes discovery client to both consumer gateway and provider

### Budget Configuration

- `--budget` CLI flag > `config.json default_budget_usdc` > default $1.00

### Error Handling

- Missing wallet → "Run 'veil init' first"
- Wrong password → spinner stops with ✗, exit 1
- Invalid config → formatted validation errors
- API key validation → test call to Anthropic `/v1/models`
- SIGINT/SIGTERM handlers for graceful shutdown on provide/relay/use commands

## API Specification

CLI module exposes no programmatic API. All interaction is via `process.argv`.

### Environment Variables

| Variable | Description |
|----------|-------------|
| `VEIL_HOME` | Wallet directory (default `~/.veil`) |
| `VEIL_PASSWORD` | Non-interactive password (CI/systemd) |
| `VEIL_RELAY_PORT` | Relay port override |
| `ANTHROPIC_API_KEY` | Fallback API key |
| `PROXY_URL` | LiteLLM proxy URL |
| `PROXY_SECRET` | Proxy shared secret |
| `NO_COLOR` / `FORCE_COLOR` | Color output control |

## Integration Protocol

- **→ Wallet**: `createWallet`, `loadWallet`, `encryptApiKey`, `decryptApiKey`, `exportWallet`, `changeWalletPassword`, `migrateWallet`
- **→ Consumer**: `startGateway()` with wallet, relay URL, discovery client, budget
- **→ Provider**: `startProvider()` with wallet, relay URL, API keys, proxy config, discovery client
- **→ Relay**: `startRelay()` with wallet, DB path, port
- **→ RBOB**: `RbobLedger` for grant, balance, leaderboard, export
- **→ WitnessStore**: for witness lookup, stats, export
- **→ Discovery**: `RelayDiscoveryClient` for relay listing
- **→ Config**: `loadAndValidateConfig`, `loadAndValidateProviderConfig`

## Current Implementation Status

- ✅ Full wallet lifecycle (init, export, change-password, migrate) [IMPLEMENTED]
- ✅ Provider init + start with config validation [IMPLEMENTED]
- ✅ Relay start [IMPLEMENTED]
- ✅ Consumer gateway start with budget config [IMPLEMENTED]
- ✅ Status check (gateway, relay, provider) [IMPLEMENTED]
- ✅ RBOB commands (balance, leaderboard, grant, export) [IMPLEMENTED]
- ✅ Witness commands (get, stats, export) [IMPLEMENTED]
- ✅ Relay discovery listing [IMPLEMENTED]
- ✅ UX: spinner, colors, table formatting, NO_COLOR support [IMPLEMENTED]
- ✅ Non-interactive mode via VEIL_PASSWORD env [IMPLEMENTED]
- ⚠️ Consumer gateway startup (`cmdUseStart`) [IMPLEMENTED · NEEDS ALIGNMENT] — creates `RelayDiscoveryClient` and passes as `discoveryClient` to gateway; should pass `relayMode`+`bootstrapUrl` instead per architecture
- ⚠️ Provider startup (`cmdProvideStart`) [IMPLEMENTED · NEEDS ALIGNMENT] — same `discoveryClient` injection pattern; should use `relayMode`+`bootstrapUrl`
- ⚠️ Budget config uses `defaultBudgetUsdc` / `default_budget_usdc` [IMPLEMENTED · NEEDS ALIGNMENT] — should be `defaultQuoteBudget` per architecture
- ❌ `veil autopilot init/join/show` [NOT IMPLEMENTED]
- ❌ Interactive TUI dashboard [NOT IMPLEMENTED]

## Code Alignment Tasks

- [ ] Replace `discoveryClient` injection with `relayMode`+`bootstrapUrl` in `cmdUseStart()` and `cmdProvideStart()`
- [ ] Rename `default_budget_usdc` config key → `default_quote_budget`
- [ ] Rename `--budget` flag semantics to align with `defaultQuoteBudget` + `quoteUnit`
- [ ] Update config validation to accept `relayMode` instead of `relay_discovery`
