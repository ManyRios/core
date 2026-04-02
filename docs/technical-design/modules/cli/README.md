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
