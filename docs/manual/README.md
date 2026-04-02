# Manual

## Purpose

This section explains how to operate Veil after installation.

This page documents the current CLI-first operating path. The target operating model is still Claw-managed join, pricing, publishing, and recovery with much lower manual effort.

## Role-Based Usage

### Consumer

- initialize a wallet with `veil init`
- point an OpenAI-compatible client to `http://localhost:9960/v1`
- optionally configure local API auth

### Provider

- configure credentials with `veil provide init`
- start the Provider with `veil provide start`
- monitor health and upstream failures
- treat this as the current manual path, not the long-term operator experience

### Relay

- start the Relay with `veil relay start`
- watch Provider registration and request forwarding
- export and inspect witness data as needed

## Operational Checklist

### Before Starting

- wallet exists
- config files are present
- the selected ports are available
- upstream credentials are valid for Provider nodes

### While Running

- check `health` endpoints
- watch logs for Relay disconnects, Provider errors, or auth failures
- verify witness data is still being recorded

### When Debugging

- run `npm test`
- run targeted Vitest suites
- inspect local runtime files under `VEIL_HOME` or `~/.veil`

## Useful Commands

```bash
veil init
veil provide init
veil provide start
veil relay start
npm test
npx vitest run --reporter=verbose
```

## Target Operator Path

The target operator model reduces the manual steps above to:

- operator intent
- policy guardrails
- credential approval
- Claw-managed join, sell, pause, and recovery workflows

## Next Reading

- [Operations](../operations/README.md)
- [Protocol](../technical-design/protocol/README.md)
- [Modules](../technical-design/modules/README.md)
- [Claw Autopilot](../technical-design/modules/claw-autopilot/README.md)
